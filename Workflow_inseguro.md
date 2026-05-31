# Auditoria de Segurança: Workflow Inseguro

Este documento apresenta a análise de um workflow de Pull Request inseguro, identifica os principais riscos e mostra uma versão endurecida (hardening) do pipeline.

---

## 1. Visão Geral

Durante a análise do pipeline de validação de Pull Requests (`pull_request`), foram identificadas quatro vulnerabilidades críticas que expunham o ambiente a riscos severos de segurança.

## 2. Vulnerabilidades Encontradas

### 2.1 Permissão Global `permissions: write-all`
- **Severidade:** Crítica
- **Risco:** concede permissões totais de escrita sobre o repositório, pacotes e segredos.
- **Impacto:** qualquer Pull Request, inclusive de forks maliciosos, pode executar ações com controle completo do repositório.

### 2.2 Uso de `runs-on: [self-hosted]`
- **Severidade:** Alta
- **Risco:** executa código de terceiros em servidores privados sem auditoria prévia.
- **Impacto:** aumenta a chance de ataques laterais na rede, persistência maliciosa e comprometimento do host.

### 2.3 Dependência Mutável `actions/checkout@main`
- **Severidade:** Média
- **Risco:** apontar para um branch flutuante (`@main`) deixa o pipeline vulnerável a ataques de cadeia de suprimentos.
- **Impacto:** se o repositório da action for comprometido, código malicioso pode ser injetado automaticamente na execução.

### 2.4 Injeção de Script via contexto Bash
- **Severidade:** Crítica
- **Risco:** usar `${{ github.event.pull_request.title }}` diretamente dentro de um bloco `run: |` permite injeção de comandos.
- **Impacto:** um invasor pode alterar o título do Pull Request para executar payloads arbitrários.

**Workflow Inseguro**

```yaml
name: Workflow Inseguro

on:
  pull_request:
    types: [opened]

permissions: write-all

jobs:
  processar-pr:
    runs-on: [self-hosted]
    steps:
      - uses: actions/checkout@main

      - name: Exibir informações do PR
        run: |
          echo "PR aberto por: ${{ github.actor }}"
          echo "Título: ${{ github.event.pull_request.title }}"
          echo "Branch: ${{ github.head_ref }}"
```

---

## 3. Workflow Protegido (Hardening)

Para neutralizar os riscos identificados, a proposta de correção aplica as seguintes práticas:

- permissões mínimas explícitas
- runner hospedado pelo GitHub
- uso de SHA imutável para actions oficiais
- sanitização via variáveis de ambiente

```yaml
name: Workflow Seguro e Auditado

on:
  pull_request:
    types: [opened]

permissions:
  contents: read

jobs:
  processar-pr:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@692973e3d937129bcbf40652eb9f2f61becf3332 # v4.1.7

      - name: Exibir informações do PR (Protegido contra Injeção)
        env:
          PR_TITLE: ${{ github.event.pull_request.title }}
          PR_ACTOR: ${{ github.actor }}
          PR_HEAD_REF: ${{ github.head_ref }}
        run: |
          echo "PR aberto por: $PR_ACTOR"
          echo "Título: $PR_TITLE"
          echo "Branch: $PR_HEAD_REF"
```

---

## 4. Boas Práticas Adotadas

- **Isolamento de Contexto:** nunca expanda dados do usuário diretamente no corpo de `run:`. O uso de `env:` garante que o Bash trate os valores como dados, não como comandos.

- **Ambientes Efêmeros:** prefira `ubuntu-latest` em vez de runners self-hosted para garantir execução em máquinas limpas e descartáveis.

- **Congelamento de Dependências:** use SHAs imutáveis para actions de terceiros, evitando alterações inesperadas ou comprometimento de código.

---

## 5. Observações Finais

Este documento foi desenvolvido como parte da auditoria de segurança de pipelines DevOps e serve como referência para melhorar a proteção de workflows no GitHub Actions.
"""