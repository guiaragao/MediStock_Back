# Modelo Oracle do MediStock / Smart HAS

Base do domínio: repositório `luanaestanislau/MediStock_Back`, commit `083ba6a56216cfa030a6e229df9e307c76a53888`, analisado na integração inicial. A consolidação acrescenta a auditoria definida nos scripts Oracle desta entrega.

**Estrutura consolidada: 7 tabelas, 63 colunas, 8 chaves estrangeiras e 9 índices de apoio explicitamente criados, além dos índices de PK e unicidade.**

As evidências da implantação inicial e da persistência estão no [README](../README.md#evidencias). O modelo abaixo incorpora também a estrutura de auditoria da consolidação; os scripts SQL são a referência física e a [documentação PL/SQL](PLSQL_OBJETOS.md) descreve os objetos que a utilizam.

![DER completo do MediStock](DER_MediStock.png)

## DER — chaves e cardinalidades

Cada vínculo é 1:N: um registro do pai pode ter zero ou muitos filhos; cada filho aponta para exatamente um pai em cada FK. A tabela USUARIOS é independente porque o código atual não a relaciona a um hospital. A tabela AUDITORIA_CONSUMO também não possui FKs: seus IDs são referências históricas por valor. A linha tracejada do desenho representa a gravação pela trigger, não um relacionamento de integridade referencial. O desenho mostra as colunas; o dicionário abaixo registra os tipos completos e a nulabilidade.

```mermaid
erDiagram
    direction TB
    HOSPITAIS ||--o{ ITENS_ESTOQUE : armazena
    HOSPITAIS ||--o{ HISTORICO_CONSUMO : registra
    ITENS_ESTOQUE ||--o{ HISTORICO_CONSUMO : referencia
    HOSPITAIS ||--o{ ENTREGAS : recebe
    ITENS_ESTOQUE ||--o{ ENTREGAS : referencia
    HOSPITAIS ||--o{ TRANSFERENCIAS : origem
    HOSPITAIS ||--o{ TRANSFERENCIAS : destino
    ITENS_ESTOQUE ||--o{ TRANSFERENCIAS : referencia
    USUARIOS {
        number ID PK
        string EMAIL_INSTITUCIONAL UK
    }
    HOSPITAIS {
        number ID PK
        string NOME
    }
    ITENS_ESTOQUE {
        number ID PK
        string NOME
        number HOSPITAL_ID FK
    }
    HISTORICO_CONSUMO {
        number ID PK
        number ITEM_ESTOQUE_ID FK
        number HOSPITAL_ID FK
        date MES_REFERENCIA
        number QUANTIDADE_CONSUMIDA
    }
    ENTREGAS {
        number ID PK
        number ITEM_ESTOQUE_ID FK
        number HOSPITAL_DESTINO_ID FK
        string STATUS
    }
    TRANSFERENCIAS {
        number ID PK
        number ITEM_ESTOQUE_ID FK
        number HOSPITAL_ORIGEM_ID FK
        number HOSPITAL_DESTINO_ID FK
        string STATUS
    }
    %% Auditoria sem FKs: preenchida pela trigger do historico.
    AUDITORIA_CONSUMO {
        number ID PK
        number HISTORICO_ID
        string OPERACAO
        number ITEM_ID_ANTES
        number ITEM_ID_DEPOIS
        number HOSPITAL_ID_ANTES
        number HOSPITAL_ID_DEPOIS
        date REFERENCIA_ANTES
        date REFERENCIA_DEPOIS
        number QUANTIDADE_ANTES
        number QUANTIDADE_DEPOIS
        string USUARIO_BANCO
        string CLIENT_IDENTIFIER
        string MODULO
        timestamp REGISTRADO_EM
    }
```

## Decisões de modelagem

- As seis entidades JPA correspondem às tabelas de domínio. A sétima tabela, AUDITORIA_CONSUMO, é mantida pela trigger Oracle e não exige uma entidade JPA. Os nomes do domínio seguem os mapeamentos existentes.
- Todas as tabelas utilizam identidade Oracle. No domínio, isso corresponde a `GenerationType.IDENTITY`. A carga omite IDs e recupera os gerados com `RETURNING`, evitando conflito com cadastros futuros.
- `Long` vira `NUMBER(19,0)`, `Integer` vira `NUMBER(10,0)`, `Double` vira `BINARY_DOUBLE`, `BigDecimal(12,2)` vira `NUMBER(12,2)`.
- `boolean` usa `BOOLEAN`, disponível no Oracle Free 26ai. O script não foi feito para Oracle 19c ou 21c.
- Strings usam `VARCHAR2(... CHAR)`. `LocalDate` usa `DATE` e `LocalDateTime` usa `TIMESTAMP(6)`. O horário da auditoria usa `TIMESTAMP(6) WITH TIME ZONE`.
- As FKs preservam os relacionamentos do código. Não há exclusão em cascata: referências precisam ser tratadas antes de excluir os pais.
- Quantidades, custos e estados logísticos têm constraints. Há oito índices de suporte às FKs e consultas de histórico, mais `IX_AUD_CONSUMO_HISTORICO` para a auditoria, além dos índices das PKs e da unicidade de e-mail.
- O histórico preserva a possibilidade de registrar a demanda de um item em outro hospital, usada pela análise de redistribuição. A FK do histórico não é obrigada a repetir o hospital atual do estoque.
- Não foi adicionada unicidade por item/hospital/mês: o serviço original permite vários registros. A procedure também preserva a data recebida; a carga usa o primeiro dia do mês por convenção.
- Os níveis de estoque e os alertas são calculados a partir dos dados; não criamos tabelas extras para resultados derivados.
- O requisito de dados simulados é atendido com históricos e dados do domínio de logística hospitalar: hospitais, itens, entregas e transferências.
- Entregas e transferências são registros logísticos. No código atual, alterar o status não movimenta automaticamente o saldo; a carga e a procedure não acrescentam esse comportamento.

## Dicionário físico completo

### USUARIOS

| Coluna | Tipo Oracle | Permite NULL | Chave / descrição |
|---|---|---|---|
| `ID` | `NUMBER(19,0)` | Não | PK. Chave primária; gerada pelo Oracle. |
| `PRIMEIRO_NOME` | `VARCHAR2(80 CHAR)` | Não | Primeiro nome do usuário. |
| `ULTIMO_NOME` | `VARCHAR2(80 CHAR)` | Não | Sobrenome do usuário. |
| `EMAIL_INSTITUCIONAL` | `VARCHAR2(150 CHAR)` | Não | UNIQUE. E-mail institucional único; a API normaliza para minúsculas. |
| `SENHA_HASH` | `VARCHAR2(255 CHAR)` | Não | Hash BCrypt produzido pelo cadastro da API. |
| `CRIADO_EM` | `TIMESTAMP(6)` | Não | Data e hora de criação, sem fuso no modelo atual. |

### HOSPITAIS

| Coluna | Tipo Oracle | Permite NULL | Chave / descrição |
|---|---|---|---|
| `ID` | `NUMBER(19,0)` | Não | PK. Chave primária; gerada pelo Oracle. |
| `NOME` | `VARCHAR2(150 CHAR)` | Não | Nome do hospital. |
| `ENDERECO` | `VARCHAR2(200 CHAR)` | Sim | Endereço do hospital. |
| `CIDADE` | `VARCHAR2(80 CHAR)` | Sim | Cidade. |
| `ESTADO` | `VARCHAR2(2 CHAR)` | Sim | Sigla da unidade federativa. |
| `LATITUDE` | `BINARY_DOUBLE` | Não | Coordenada para cálculos geográficos. |
| `LONGITUDE` | `BINARY_DOUBLE` | Não | Coordenada para cálculos geográficos. |

### ITENS_ESTOQUE

| Coluna | Tipo Oracle | Permite NULL | Chave / descrição |
|---|---|---|---|
| `ID` | `NUMBER(19,0)` | Não | PK. Chave primária; gerada pelo Oracle. |
| `NOME` | `VARCHAR2(150 CHAR)` | Não | Nome do insumo. |
| `QUANTIDADE_ATUAL` | `NUMBER(10,0)` | Não | Saldo atual; não negativo. |
| `QUANTIDADE_MINIMA` | `NUMBER(10,0)` | Não | Limite mínimo do estoque; não negativo. |
| `UNIDADE_MEDIDA` | `VARCHAR2(20 CHAR)` | Sim | Unidade usada para as quantidades. |
| `LOCAL_ARMAZENAMENTO` | `VARCHAR2(150 CHAR)` | Sim | Local do item dentro do hospital. |
| `HOSPITAL_ID` | `NUMBER(19,0)` | Não | FK → HOSPITAIS.ID; local atual do item. |
| `VALIDADE` | `DATE` | Sim | Data de validade. |
| `CUSTO_UNITARIO` | `NUMBER(12,2)` | Sim | Custo unitário; não negativo. |
| `ALTO_CUSTO_BAIXA_DEMANDA` | `BOOLEAN` | Não | Indica elegibilidade para análise de redistribuição. |
| `ATUALIZADO_EM` | `TIMESTAMP(6)` | Sim | Última atualização; a API atualiza via @PreUpdate. |

### HISTORICO_CONSUMO

| Coluna | Tipo Oracle | Permite NULL | Chave / descrição |
|---|---|---|---|
| `ID` | `NUMBER(19,0)` | Não | PK. Chave primária; gerada pelo Oracle. |
| `ITEM_ESTOQUE_ID` | `NUMBER(19,0)` | Não | FK → ITENS_ESTOQUE.ID. |
| `HOSPITAL_ID` | `NUMBER(19,0)` | Não | FK → HOSPITAIS.ID; hospital que registrou a demanda. |
| `MES_REFERENCIA` | `DATE` | Não | Referência do consumo; a carga usa o primeiro dia do mês. |
| `QUANTIDADE_CONSUMIDA` | `NUMBER(10,0)` | Não | Quantidade consumida registrada; não negativa. |

### ENTREGAS

| Coluna | Tipo Oracle | Permite NULL | Chave / descrição |
|---|---|---|---|
| `ID` | `NUMBER(19,0)` | Não | PK. Chave primária; gerada pelo Oracle. |
| `ITEM_ESTOQUE_ID` | `NUMBER(19,0)` | Não | FK → ITENS_ESTOQUE.ID. |
| `HOSPITAL_DESTINO_ID` | `NUMBER(19,0)` | Não | FK → HOSPITAIS.ID. |
| `QUANTIDADE` | `NUMBER(10,0)` | Não | Quantidade da entrega; maior que zero. |
| `STATUS` | `VARCHAR2(20 CHAR)` | Não | PENDENTE, EM_ROTA, CONCLUIDA ou CANCELADA. |
| `DATA_PREVISTA` | `DATE` | Sim | Data prevista para a entrega. |
| `TRANSPORTADORA` | `VARCHAR2(150 CHAR)` | Sim | Nome da transportadora. |
| `CRIADO_EM` | `TIMESTAMP(6)` | Sim | Data e hora de criação. |

### TRANSFERENCIAS

| Coluna | Tipo Oracle | Permite NULL | Chave / descrição |
|---|---|---|---|
| `ID` | `NUMBER(19,0)` | Não | PK. Chave primária; gerada pelo Oracle. |
| `ITEM_ESTOQUE_ID` | `NUMBER(19,0)` | Não | FK → ITENS_ESTOQUE.ID. |
| `HOSPITAL_ORIGEM_ID` | `NUMBER(19,0)` | Não | FK → HOSPITAIS.ID; origem. |
| `HOSPITAL_DESTINO_ID` | `NUMBER(19,0)` | Não | FK → HOSPITAIS.ID; destino. |
| `QUANTIDADE` | `NUMBER(10,0)` | Não | Quantidade a transferir; maior que zero. |
| `STATUS` | `VARCHAR2(20 CHAR)` | Não | PENDENTE, EM_ROTA, CONCLUIDA ou CANCELADA. |
| `DISTANCIA_KM` | `BINARY_DOUBLE` | Sim | Distância estimada em quilômetros. |
| `TEMPO_ESTIMADO_MIN` | `BINARY_DOUBLE` | Sim | Tempo estimado em minutos. |
| `MOTIVO` | `VARCHAR2(300 CHAR)` | Sim | Justificativa da transferência. |
| `GERADO_POR_IA` | `BOOLEAN` | Não | Indica se a origem do registro foi uma sugestão de IA. |
| `CRIADO_EM` | `TIMESTAMP(6)` | Sim | Data e hora de criação. |

### AUDITORIA_CONSUMO

| Coluna | Tipo | Nulo? | Finalidade |
|---|---|---|---|
| `ID` | `NUMBER(19,0)` identity | Não | Chave primária do evento. |
| `HISTORICO_ID` | `NUMBER(19,0)` | Não | ID afetado: novo em INSERT/UPDATE, antigo em DELETE. |
| `OPERACAO` | `VARCHAR2(6 CHAR)` | Não | `INSERT`, `UPDATE` ou `DELETE`. |
| `ITEM_ID_ANTES` | `NUMBER(19,0)` | Sim | Item anterior. |
| `ITEM_ID_DEPOIS` | `NUMBER(19,0)` | Sim | Item posterior. |
| `HOSPITAL_ID_ANTES` | `NUMBER(19,0)` | Sim | Hospital anterior. |
| `HOSPITAL_ID_DEPOIS` | `NUMBER(19,0)` | Sim | Hospital posterior. |
| `REFERENCIA_ANTES` | `DATE` | Sim | Referência anterior. |
| `REFERENCIA_DEPOIS` | `DATE` | Sim | Referência posterior. |
| `QUANTIDADE_ANTES` | `NUMBER(10,0)` | Sim | Quantidade anterior. |
| `QUANTIDADE_DEPOIS` | `NUMBER(10,0)` | Sim | Quantidade posterior. |
| `USUARIO_BANCO` | `VARCHAR2(128 CHAR)` | Não | `SYS_CONTEXT('USERENV','SESSION_USER')`. |
| `CLIENT_IDENTIFIER` | `VARCHAR2(64 CHAR)` | Sim | Identificador da sessão, caso tenha sido definido pela aplicação. |
| `MODULO` | `VARCHAR2(64 CHAR)` | Sim | Módulo informado pela ferramenta ou aplicação. |
| `REGISTRADO_EM` | `TIMESTAMP(6) WITH TIME ZONE` | Não | Data/hora e fuso, com default `SYSTIMESTAMP`. |

Restrições explícitas: `PK_AUDITORIA_CONSUMO`, para a chave primária; `CK_AUD_CONSUMO_OPERACAO`, para limitar as operações; e os `NOT NULL` indicados acima. A identidade e o índice de suporte à chave primária são administrados pelo Oracle.

Os IDs armazenados são referências históricas por valor. Não há FKs para histórico, item ou hospital, permitindo preservar os registros depois de exclusões confirmadas. A tabela é um registro transacional da aplicação, não um mecanismo independente de auditoria administrativa ou de proteção contra alterações pelo dono do schema.

## Objetos PL/SQL

`PR_REGISTRAR_CONSUMO(item_id, hospital_id, data_referencia, quantidade)` mantém a interface usada pelo Java. Na versão consolidada, delega para `PKG_MEDISTOCK.PR_REGISTRAR_CONSUMO`, que valida as entradas, insere em `HISTORICO_CONSUMO` e devolve o ID gerado.

O mesmo package contém `FN_TOTAL_CONSUMO`, `PR_CONSULTAR_ESTOQUE` e duas validações privadas. A trigger `TRG_AUD_HISTORICO_CONSUMO` grava os eventos em `AUDITORIA_CONSUMO`. Assinaturas, finalidade, regras e códigos completos estão em [PLSQL_OBJETOS.md](PLSQL_OBJETOS.md).

A procedure insere o histórico e preserva o saldo de estoque. O chamador controla a transação: a rotina não contém `COMMIT` ou `ROLLBACK`. No fluxo de integração Oracle documentado, o serviço Java usa `@Transactional` e o gravador Oracle chama a procedure uma única vez, com os quatro parâmetros. Os testes de banco exercitam inserção, rejeição de valores inválidos e reversão pelo chamador.

| Código | Significado |
|---|---|
| -20001 | IDs inválidos. |
| -20002 | Quantidade nula, negativa, fracionária ou fora de Java Integer. |
| -20003 | Data de referência ausente. |
| -20004 | Item inexistente. |
| -20005 | Hospital inexistente. |

## Referências técnicas

- [Código do MediStock](https://github.com/luanaestanislau/MediStock_Back/tree/083ba6a56216cfa030a6e229df9e307c76a53888)
- [Oracle CREATE USER](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/CREATE-USER.html)
- [Oracle CREATE TABLE e IDENTITY](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/CREATE-TABLE.html)
- [Oracle tipos de dados e BOOLEAN](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/Data-Types.html)
- [Hibernate OracleDialect](https://github.com/hibernate/hibernate-orm/blob/main/hibernate-core/src/main/java/org/hibernate/dialect/OracleDialect.java)
- [Hibernate: opção de tipos binários, padrão desde 7.0](https://github.com/hibernate/hibernate-orm/blob/main/hibernate-core/src/main/java/org/hibernate/cfg/DialectSpecificSettings.java)
- [Oracle JDBC Quick Start](https://www.oracle.com/database/technologies/getting-started-using-jdbc.html)
