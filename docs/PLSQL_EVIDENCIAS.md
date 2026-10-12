# Evidências e validação — MediStock / Oracle PL/SQL

## Escopo das capturas disponíveis

O pacote contém **14 prints**, apresentados e comentados no [README](../README.md#evidencias). Os 12 arquivos em [evidencias](evidencias) registram a primeira integração com Oracle, em 26/09/2026. Os dois arquivos em [evidencias-plsql](evidencias-plsql) registram a preparação da consolidação, em 11/10/2026.

| Resultado observado | Arquivos |
|---|---|
| Cadastro com HTTP 201 e token | `01-cadastro-usuario.png`, `02-cadastro-201-token.png`. |
| Usuário persistido no Oracle | `03-usuario-no-oracle.png`. |
| Autorização configurada no Swagger | `04-autorizar-token.png`, `05-token-autorizado.png`. |
| GET autenticado de hospitais com HTTP 200 | `06-hospitais-autenticado.png`. |
| Procedure independente com status VALID | `07-procedure-valid.png`. |
| Inserção válida, rejeições e rollback do script 05 | `08-testes-plsql.png`. |
| Corpo da requisição de histórico de consumo | `09-post-historico-consumo.png`. |
| Contagem de 72 para 73 históricos | `10-historico-antes-72.png`, `11-historico-depois-73.png`. |
| Consumo válido persistido: ID 221 e quantidade 7 | `12-consumo-persistido.png`. |

O anexo 12 é uma consulta do consumo persistido. A validação de quantidade negativa está na saída do script 05, no anexo 8. A imagem 9 mostra a preparação do POST, sem o status final de sua resposta.

### Prints de 11/10/2026 incluídos no pacote

| Anexo | Arquivo no pacote | Registro visual |
|---|---|---|
| 13 | [13-lista-scripts-plsql.png](evidencias-plsql/13-lista-scripts-plsql.png) | Lista dos arquivos SQL de 06 a 14, originalmente enviada como `image(20261011-230910).png`. |
| 14 | [14-validacao-conexao-system.png](evidencias-plsql/14-validacao-conexao-system.png) | Validação do script 06 com `ORA-20090`, originalmente enviada como `image(20261011-231449).png`. |

O anexo 14 registra a exigência de executar o script de permissões como `SYSTEM / FREEPDB1`. A mensagem indica que o usuário ou o container da sessão estava diferente do esperado e interrompe o script antes do `GRANT`. Não demonstra a criação da trigger nem a concessão posterior do privilégio. As legendas completas estão na [seção da consolidação](../README.md#evidencias-consolidacao).

## Situação da consolidação PL/SQL

A execução dos scripts `06` a `13` foi informada no acompanhamento da atividade em 11/10/2026. Os dois prints dessa data estão incluídos, mas não mostram as saídas bem-sucedidas dos testes da consolidação. Os critérios abaixo descrevem como complementar a comprovação da etapa.

| Etapa | Implementação / roteiro | Evidência visual neste conjunto |
|---|---|---|
| Organização dos scripts e proteção da conexão | Arquivos 06 a 14 e validação inicial do script 06. | Anexos 13 e 14 incluídos. |
| Package e procedure adaptadora | Scripts 08, 09 e 11. | Não anexada. |
| Trigger e tabela de auditoria | Scripts 07, 10 e 11. | Não anexada. |
| Testes de auditoria, consulta e rollback | Script 12, com 14 verificações esperadas. | Não anexada. |
| Demonstração de Dynamic SQL e function | Script 13. | Não anexada. |
| POST com auditoria persistida | API e script 14, antes/depois. | Não anexada. |

## Capturas para complementar a documentação

Salve as capturas adicionais em `docs/evidencias-plsql`, que já contém os anexos 13 e 14. As sugestões de nomes abaixo são um roteiro para os resultados de sucesso; esses arquivos adicionais ainda não integram o pacote.

| Nome sugerido | Conteúdo a registrar | Critério de sucesso |
|---|---|---|
| `01-objetos-validos.png` | Saída do script 11. | Seis objetos VALID; trigger ENABLED; ausência de erros; adaptadora depende do package. |
| `02-auditoria-insert-update-delete.png` | Parte do script 12 referente aos eventos. | INSERT com 7, UPDATE de 7 para 9 e DELETE com valor anterior 9. |
| `03-testes-e-rollback.png` | Final do script 12. | Mensagens OK e `SUCESSO: 14 verificacoes concluidas; dados de teste desfeitos.` |
| `04-consultas-dinamicas.png` | Consultas do script 13. | Resultados coerentes com filtros e ordenação. Use mais de um print se necessário. |
| `05-function-total.png` | Total no final do script 13. | Soma para o item e hospital indicados. |
| `06-api-consumo-201.png` | POST de consumo no Swagger. | Corpo válido, token autorizado e HTTP 201. |
| `07-auditoria-api.png` | Script 14 depois do POST. | Histórico e evento INSERT correspondentes; quantidade posterior 7. |

Sem gravações concorrentes, um POST válido aumenta `TOTAL_HISTORICOS` e `TOTAL_EVENTOS` em uma unidade. Anote também as contagens antes da requisição para demonstrar a comparação.

Os dados do script 12 são revertidos ao final. A ausência dos respectivos logs depois do teste faz parte do comportamento esperado. O teste pela API, confirmado pela transação do serviço, produz uma evidência persistida separada.

## Como acrescentar um novo print

Depois de salvar a imagem, inclua no README da raiz:

```markdown
![Consolidação — objetos Oracle validados](docs/evidencias-plsql/01-objetos-validos.png)
```

Neste documento, que fica dentro de `docs`, o caminho correspondente é `evidencias-plsql/01-objetos-validos.png`.

Atualize a situação de cada evidência conforme os resultados reais. Use IDs, datas e contagens observados no ambiente da execução, pois eles podem variar entre instalações.

## Material para a atividade

A documentação reúne os scripts de implantação, o modelo, o catálogo dos objetos e os resultados disponíveis. Para a entrega da consolidação, acrescente as novas evidências e a identificação acadêmica solicitada pelo enunciado. O envio deve ser feito na seção da atividade na FIAP ON, no formato aceito pela plataforma.
