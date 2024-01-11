# PoC: Semgrep (SAST) + ZAP (DAST) para GitHub Actions

Este repositório utiliza GitHub Actions para implementar pipelines de segurança usando Semgrep para SAST e OWASP ZAP (Zed Attack Proxy) para DAST.

## Semgrep (SAST)

### Configuração do Pipeline

O pipeline é acionado por três eventos diferentes:

1. **pull_request:**  
   - Analisa arquivos modificados em pull requests, tornando-o ciente de diferenças e focado apenas na análise das modificações.

2. **workflow_dispatch:**  
   - Permite a execução sob demanda através da interface do GitHub Actions. Pode ser acionado manualmente quando necessário.

3. **push:**  
   - Analisa branches principais (`master` e `main`) e relata todos os findings.

### Configuração do Job

O job utiliza um contêiner Docker com a ferramenta Semgrep pré-instalada. Não altere a imagem Docker especificada (`returntocorp/semgrep`) para garantir compatibilidade com o pipeline.

#### Passos do Job

1. **Ignorar PRs do Dependabot:**
   - Usa uma declaração condicional (`if: (github.actor != 'dependabot[bot]')`) para ignorar PRs criados pelo Dependabot e evitar problemas de permissão.

2. **Buscar o Código-Fonte do Projeto:**
   - Utiliza a ação GitHub Checkout (`actions/checkout@v3`) para obter o código-fonte do projeto.

3. **Executar Análise do Semgrep:**
   - Executa o comando `semgrep ci` dentro do contêiner Docker. Esse comando inicia a análise estática do Semgrep no projeto.

#### Variável de Ambiente

- **SEMGREP_APP_TOKEN:**
  - O pipeline se conecta à Plataforma Semgrep Cloud usando o token de aplicativo fornecido. Esse token é armazenado de forma segura nos secrets do GitHub e é recuperado durante a execução do pipeline.
  - Certifique-se de que o token de aplicativo do Semgrep foi gerado na Plataforma Semgrep Cloud em Configurações.
  - Adicione o token gerado aos segredos do repositório GitHub com o nome `SEMGREP_APP_TOKEN`.

## ZAP (DAST) Pipeline

### Configuração do Pipeline

O pipeline é acionado por três eventos diferentes:

1. **pull_request:**  
   - Analisa arquivos modificados em pull requests, tornando-o ciente de diferenças e focado apenas na análise das modificações.

2. **workflow_dispatch:**  
   - Permite a execução sob demanda através da interface do GitHub Actions. Você pode acionar manualmente o pipeline quando necessário.

3. **push:**  
   - Analisa branches principais (`master` e `main`) e relata todas as descobertas. Isso é útil para monitorar continuamente a segurança do seu aplicativo.

### Configuração do Job

1. **Checkout do Código-Fonte:**
   - Utiliza a ação GitHub Checkout (`actions/checkout@v2`) para obter o código-fonte do projeto a partir do branch `master`.

2. **Execução do ZAP Scan:**
   - Utiliza a ação `zaproxy/action-baseline@v0.10.0` para realizar a varredura com o ZAP.
     - O token de autenticação é fornecido através do `secrets.GITHUB_TOKEN`.
     - A app para realizar a varredura foi definida como 'https://juice-shop.herokuapp.com/' para ganhar tempo, mas lembre de alterar para a URL real da sua app.
     - O arquivo de regras é especificado como '.zap/rules.tsv'.
     - Opções adicionais do comando ZAP podem ser configuradas usando `cmd_options`.
