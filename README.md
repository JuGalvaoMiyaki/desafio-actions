# 🚀 Laboratório de CI/CD: Automatizações com GitHub Actions

[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-Enabling_DevOps-blue.svg)](https://github.com/features/actions)
[![Node.js](https://img.shields.io/badge/Node.js-v18.x+-green.svg)](https://nodejs.org/)
[![CI/CD](https://img.shields.io/badge/Pipeline-CI%2FCD-orange.svg)]()
[![License](https://img.shields.io/badge/License-MIT-lightgrey.svg)]()

Este repositório funciona como um laboratório prático de engenharia DevOps focado no domínio de pipelines de Integração Contínua (CI) e Entrega Contínua (CD) utilizando **GitHub Actions**. 

Através dos múltiplos cenários configurados, o projeto demonstra desde automações básicas disparadas por eventos simples até arquiteturas de pipelines avançadas contendo controle de concorrência, persistência de artefatos, tratamento de falhas e deploys manuais parametrizados.

---

## 🗺️ Mapa de Gatilhos e Ecossistema de Workflows

Cada arquivo dentro do diretório `.github/workflows/` foi projetado para validar um comportamento específico do motor do GitHub Actions:

```text
📦 Repositório (Push / PR / Manual)
 │
 ├── PUSH (Qualquer branch) ──────────────► 🛠️ Hello Actions & Pipeline Módulo 2
 │                                           └─ Geram versionamento automático dinâmico
 │
 ├── PUSH (master/feature/**) ───────────► 🧪 CI Módulo 4 (Build ➔ Test ➔ Notify)
 │                                           └─ Controle de Concorrência & Compartilhamento de Artefatos
 │
 ├── PUSH (Somente na pasta src/**) ─────► 🔍 CI — Alterações em src/ (Filtro de Caminho)
 │
 ├── PULL REQUEST (Abertura) ────────────► 📑 Revisão — PR Aberto (Inspeção de metadados)
 │
 └── MANUAL (Workflow Dispatch) ─────────► 🚀 Deploy Manual (Ambientes: dev, staging, prod)
📂 Estrutura do Repositório
A organização dos arquivos reflete um ambiente de testes limpo e direto:

Plaintext
.
├── .github/workflows/
│   ├── hello-actions.yml     # Inicialização e exploração das variáveis de contexto
│   ├── pipeline-modulo2.yml  # Compartilhamento de outputs globais entre Jobs distintos
│   ├── ci-modulo4.yml        # Pipeline completo com gerenciamento de artefatos e relatórios de erro
│   ├── ci-src.yml            # Mecanismo de gatilho por filtro de paths modificados
│   ├── pr-opened.yml         # Automação de revisão acionada por eventos de Pull Request
│   └── deploy-manual.yml     # Pipeline interativo para execução de deploys sob demanda
├── app.js                    # Aplicação em JavaScript base para simulações do ambiente
└── README.md                 # Documentação completa do ecossistema (este arquivo)
```

## 🛠️ Detalhamento Técnico dos Workflows

**1. CI Módulo 4 (ci-modulo4.yml)**

Este é o pipeline mais completo do laboratório. Ele simula o ciclo de vida real de uma aplicação contendo três estágios interdependentes:

Job build: Compila o projeto, injeta o número da execução em uma variável global de saída ($GITHUB_OUTPUT) no formato 1.0.${{ github.run_number }} e empacota a pasta de distribuição usando a action oficial actions/upload-artifact@v4 com retenção de 7 dias.

Job test: Depende obrigatoriamente do build (needs: build), realiza o download do artefato para conferência e simula uma falha proposital de execução (exit 1) para testar o comportamento de tolerância da esteira.

Job notify: É disparado exclusivamente se houver alguma falha no fluxo (if: failure()). Ele consolida um relatório detalhado no console do terminal com informações cruciais para auditoria do desenvolvedor (Autor, Commit, Branch e URL direta do Log).

Gestão de Concorrência: Configurado com a diretiva cancel-in-progress: true escopado por branch, garantindo que se você realizar múltiplos pushes seguidos, as execuções antigas e redundantes serão canceladas para poupar minutos de runner.

**2. Pipeline Módulo 2 (pipeline-modulo2.yml)**
Focado em demonstrar a passagem de parâmetros através de chaves exaustivas. O estágio de build detecta dinamicamente a versão corrente da execução e a exporta através da propriedade outputs, permitindo que o estágio subsequente de testes herde e exiba esse valor em tempo de execução através da sintaxe ${{ needs.build.outputs.versao }}.

**3. Deploy Manual (deploy-manual.yml)**

Uma esteira interativa que introduz o conceito de workflow_dispatch. Ao acessar a aba "Actions" no painel do GitHub, o operador pode acionar manualmente o gatilho escolhendo através de uma caixa de seleção (choice) o ambiente de destino do deploy (dev, staging ou prod). O fluxo valida a string escolhida e aplica alertas customizados caso o ambiente crítico de produção seja selecionado.

**4. Filtro de Caminho (ci-src.yml)**

Demonstra otimização de custo e tempo de máquina. Este workflow só é acordado pelo GitHub caso ocorra uma modificação explícita nos arquivos localizados dentro da estrutura de diretórios do código-fonte principal (src/). Alterações exclusivas na documentação ou arquivos de configuração ignoram este fluxo.

**5. Revisão de Pull Request (pr-opened.yml)**

Configurado para reagir especificamente ao gatilho pull_request quando o status for marcado como opened. É ideal para acionar robôs de code-review, inspecionar se o título obedece aos padrões de Conventional Commits e extrair metadados como a branch de origem (head) e a branch de destino (base).

**⚙️ Aprendizados Práticos Demonstrados**

Este repositório cobre os principais pilares de infraestrutura como código para automações:

Contextos de Execução: Mapeamento profundo do objeto ${{ github }} para auditoria de runtime (identificação de autores, hashes de commits e links diretos).

Persistência Temporária: Salvamento e recuperação de estados e binários gerados em tempo de execução via Artifacts.

Encadeamento Sequencial: Uso da tag needs para traçar caminhos dependentes de execução ordenada.

Condicionais Dinâmicas: Aplicação de regras lógicas de controle de fluxo baseadas em status (if: failure()).

**👩‍💻 Autoria e Contribuições**

Desenvolvedora Responsável: Juliana Galvão Miyaki

E-mail de Contato: juliana.galvao@tbxtech.com

Repositório focado em estudos de automação e engenharia de software contínua.
