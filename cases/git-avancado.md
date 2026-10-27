# Git avançado na prática

> Case do portfólio · Giovanni Melo · Engenharia de Dados

## 1. Contexto

Trabalho com dados em um ambiente onde quase tudo vira código: scripts SQL e PL/SQL, pipelines ETL, notebooks Python e projetos Power BI no formato `.pbip`. Quando esses arquivos passaram a ser versionados, percebi que usar apenas `add`, `commit` e `push` não bastava: o histórico crescia, ficava difícil achar a origem de um erro e o trabalho em equipe pedia um fluxo organizado.

Estudei Git a fundo (análise de histórico, administração de repositórios, reescrita de histórico e automação com GitHub Actions) para tratar o versionamento como parte da engenharia e não como detalhe. Este documento resume o que aprendi e o que reutilizo no dia a dia.

## 2. O que aprendi, por tema

### Análise de repositórios

Responder "o que mudou, quando e por quê" sem abrir arquivo por arquivo.

- **`git log`**: histórico filtrável por autor, data, arquivo ou mensagem. A forma mais útil é a versão em grafo (`--oneline --graph --decorate --all`).
- **`git diff`**: compara working directory, área de stage, commits e branches.
- **`git blame`**: mostra quem alterou cada linha e em qual commit, ótimo para entender a motivação de uma regra de negócio.
- **`git bisect`**: busca binária no histórico para achar o commit que introduziu um bug. Com `git bisect run <script>`, a busca roda sozinha.

### Administração de repositórios

- **Tags**: marcam versões estáveis (`v1.0.0`). Tags anotadas (`-a`) guardam autor, data e mensagem.
- **Hooks**: scripts que o Git executa em eventos como `pre-commit` e `pre-push`. Servem para rodar lint ou testes antes de o código sair da máquina. Os hooks em `.git/hooks` não são versionados; para compartilhar com o time, use `core.hooksPath` ou uma ferramenta como o `pre-commit`.
- **Submodules**: incluem outro repositório dentro do seu, fixando uma versão específica. Úteis para reaproveitar código compartilhado, mas exigem cuidado ao clonar (`--recurse-submodules`).

### Manipulação de histórico

- **Rebase interativo (`git rebase -i`)**: reordena, junta (`squash`/`fixup`) e reescreve mensagens de commits para deixar o histórico limpo antes de abrir um Pull Request. Regra de ouro: nunca reescrever histórico de branch que outras pessoas já usam.
- **`git cherry-pick`**: leva um commit específico de uma branch para outra, comum em correções urgentes.
- **`git stash`**: guarda alterações incompletas para trocar de branch sem criar commit de rascunho.

### GitHub como plataforma de colaboração

- **Issues**: registram tarefas, bugs e decisões. Mensagens como `Closes #12` fecham a Issue automaticamente quando o PR é mesclado.
- **Projects**: quadro (kanban) ligado às Issues e PRs para acompanhar o andamento.
- **Pull Requests**: revisão de código, discussão e verificações automáticas antes do merge. Combinados com *branch protection*, impedem merge sem revisão ou com testes falhando.

### Automação com GitHub Actions

Workflows em `.github/workflows/*.yml` que rodam a cada `push` ou Pull Request. Exemplo mínimo de integração contínua para um projeto Python:

```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: pytest
```

Para engenharia de dados, o mesmo princípio vale para validar SQL, testes de qualidade de dados e deploys de pipelines.

## 3. Cheat sheet

| Comando | Para que serve |
|---|---|
| `git log --oneline --graph --decorate --all` | Ver o histórico de todas as branches em formato de grafo |
| `git diff --staged` | Revisar o que vai entrar no próximo commit |
| `git blame -L 10,20 arquivo` | Ver quem mudou as linhas 10 a 20 e em qual commit |
| `git bisect start` / `bad` / `good` | Achar o commit que introduziu um bug |
| `git tag -a v1.0.0 -m "mensagem"` | Marcar uma versão estável |
| `git rebase -i HEAD~5` | Limpar os últimos 5 commits antes do PR |
| `git cherry-pick <sha>` | Copiar um commit específico para a branch atual |
| `git stash push -m "descrição"` | Guardar alterações em andamento (`git stash pop` devolve) |
| `git reflog` | Recuperar commits "perdidos" depois de um reset ou rebase |
| `git submodule update --init --recursive` | Baixar os submodules de um repositório recém-clonado |

## 5. Links

- Repositório do portfólio: https://github.com/GiovanniMelo7/portfolio-data-engineering
