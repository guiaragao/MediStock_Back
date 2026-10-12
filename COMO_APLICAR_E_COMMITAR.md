# Como aplicar a documentação e realizar o commit

## 1. Copiar os arquivos

Extraia `MediStock_Documentacao_GitHub.zip`. Dentro da pasta `MediStock_Documentacao_GitHub`, copie o conteúdo para a raiz do projeto, normalmente `C:\Projetos\MediStock_Back`.

O arquivo principal deve ficar em:

```text
C:\Projetos\MediStock_Back\README.md
```

As imagens devem ficar em:

```text
C:\Projetos\MediStock_Back\docs\evidencias\01-cadastro-usuario.png
C:\Projetos\MediStock_Back\docs\evidencias-plsql\13-lista-scripts-plsql.png
C:\Projetos\MediStock_Back\docs\evidencias-plsql\14-validacao-conexao-system.png
```

Substitua o README anterior e mescle as pastas `docs` e `database`. Os arquivos SQL incluídos são as mesmas versões dos roteiros `00` a `14` já fornecidos. Se houver ajustes locais nesses scripts, compare as versões antes de substituí-los.

Este pacote contém documentação, imagens, diagramas, scripts Oracle e um modelo `.env.example`. O código Java da aplicação continua sendo o código do seu projeto local.

O README utiliza caminhos relativos. Para as imagens aparecerem no GitHub, mantenha os nomes e as pastas e inclua os arquivos no mesmo commit.

## 2. Conferir o estado do repositório

Abra o PowerShell na pasta do projeto:

```powershell
Set-Location C:\Projetos\MediStock_Back
git status
git branch --show-current
git diff --stat
```

Esses comandos mostram a branch atual e as alterações existentes. O commit seguinte será realizado nessa branch.

## 3. Adicionar a documentação e os scripts desta entrega

```powershell
git add README.md COMO_APLICAR_E_COMMITAR.md .env.example
git add database/oracle
git add docs/MODELO_E_DICIONARIO.md docs/PLSQL_OBJETOS.md docs/PLSQL_EVIDENCIAS.md
git add docs/DER_MediStock.png docs/DER_MediStock.svg docs/DER_MediStock.mmd
git add docs/evidencias docs/evidencias-plsql
```

O arquivo versionado de configuração é `.env.example`, preenchido com marcadores. As credenciais usadas na execução continuam na configuração local.

Se o código Java e o `pom.xml` da integração também estiverem pendentes de commit, revise essas diferenças e adicione os arquivos correspondentes:

```powershell
git diff -- src pom.xml
git add src pom.xml
```

Os dois prints de 11/10/2026 já estão incluídos em `docs/evidencias-plsql`. Acrescente nessa mesma pasta as próximas capturas dos resultados da consolidação.

## 4. Revisar o que será incluído

```powershell
git diff --cached --stat
git diff --cached -- README.md
git status
```

Confira se o README, os diagramas e os 14 prints aparecem na lista: 12 em `docs/evidencias` e dois em `docs/evidencias-plsql`. A árvore de arquivos do repositório deve manter a mesma organização descrita no README.

## 5. Criar o commit

Para um commit que reúne a consolidação e a documentação:

```powershell
git commit -m "feat: consolida Oracle e PL/SQL com auditoria e documentacao"
```

Se o código e os scripts já estiverem versionados e o commit tiver apenas documentação, use uma mensagem correspondente, por exemplo:

```powershell
git commit -m "docs: atualiza README, DER e evidencias da integracao Oracle"
```

Escolha apenas uma das mensagens, de acordo com os arquivos incluídos no commit.

## 6. Enviar ao GitHub

Se a branch já acompanha uma branch remota:

```powershell
git push
```

Se o Git informar que não há upstream para a branch atual:

```powershell
git push -u origin HEAD
```

Depois, abra o repositório no GitHub na branch enviada. Confira o README, expanda os anexos e verifique se as imagens aparecem. O envio dos arquivos ao GitHub é separado do envio da atividade na FIAP ON.

## 7. Complementar a comprovação da etapa nova

Os prints incluídos registram a integração inicial e os dois registros de preparação da consolidação enviados em 11/10/2026. O README documenta também o package, a trigger e o SQL dinâmico, com seus testes e resultados esperados. O arquivo [docs/PLSQL_EVIDENCIAS.md](docs/PLSQL_EVIDENCIAS.md) indica quais capturas de resultados ainda acrescentar para documentar a consolidação.
