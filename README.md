# 🚀 CI/CD com GitHub Actions + SonarQube + Kubernetes (DigitalOcean)

![GitHub Actions](https://img.shields.io/github/actions/workflow/status/usuario/repositorio/ci-cd.yml?branch=main&label=CI/CD)
![SonarQube Quality Gate](https://img.shields.io/badge/SonarQube-Quality%20Gate-blue)
![Kubernetes](https://img.shields.io/badge/Kubernetes-DigitalOcean-blue)

Este projeto implementa um **pipeline CI/CD** automatizado usando **GitHub Actions** para:

- 📦 **Build** da aplicação
- 🧪 **Testes automatizados**
- 🔍 **Análise de qualidade** com SonarQube
- 🐳 **Build e push da imagem Docker**
- ☸️ **Deploy no Kubernetes** na DigitalOcean

---

## 📋 Fluxo do Pipeline

1. **Build e testes**
   - Compila a aplicação.
   - Executa testes unitários.
   - Gera relatórios de cobertura.

2. **Análise de qualidade**
   - Integração com **SonarQube**.
   - Aplica **Quality Gates** para evitar deploy de código ruim.

3. **Build e push Docker**
   - Cria imagem Docker.
   - Publica no **DigitalOcean Container Registry (DOCR)** ou **Docker Hub**.

4. **Deploy no Kubernetes**
   - Aplica manifestos no **DigitalOcean Kubernetes (DOKS)**.
   - Usa rolling updates para zero downtime.

---

## 🔑 Pré-requisitos

- Conta no [GitHub](https://github.com)
- Conta no [DigitalOcean](https://www.digitalocean.com/)
- **Cluster Kubernetes** criado no DigitalOcean
- **DigitalOcean Container Registry** configurado
- **SonarQube** instalado na AWS
- Arquivos de manifesto Kubernetes (`deployment.yaml`, `service.yaml` etc.)

---

## ⚙️ Configuração de Secrets no GitHub

No repositório, acesse **Settings > Secrets and variables > Actions** e adicione:

| Nome                  | Descrição                                                                 |
|-----------------------|---------------------------------------------------------------------------|
| `DOCKER_USERNAME`     | Usuário do DOCR ou Docker Hub                                             |
| `DOCKER_PASSWORD`     | Token de acesso do DOCR ou senha do Docker Hub                            |
| `SONAR_HOST_URL`      | URL do servidor SonarQube                                                 |
| `SONAR_TOKEN`         | Token gerado no SonarQube                                                 |
| `KUBE_CONFIG`         | Conteúdo do `kubeconfig` do cluster DOKS em Base64                        |

> Para gerar o Base64 do kubeconfig do DigitalOcean:  
> ```bash
> doctl kubernetes cluster kubeconfig save <CLUSTER_NAME>
> cat ~/.kube/config | base64 -w 0
> ```

---

## 📄 Workflow (`.github/workflows/testes.yml`)

```yaml

testes.yml

name: Execução de Testes

on:
    workflow_call: 

jobs:
    unit-test:
        name: Teste de Unidade
        runs-on: ubuntu-latest
        steps:
          - name: Obtendo o código do projeto
            uses: actions/checkout@v4
          - name: Setup dotnet
            uses: actions/setup-dotnet@v4
            with:
              dotnet-version: "8.0.300"
          - name: Execução de Teste de Unidade
            working-directory: ./src
            run: dotnet test ./Review-Filmes.Test.Unit/Review-Filmes.Test.Unit.csproj

    integration-test:
        name: Teste de Integração
        runs-on: ubuntu-latest
        services:
            postgre:
              image: postgres:latest
              ports:
                  - 5432:5432
              env:
                  POSTGRES_USER: review
                  POSTGRES_PASSWORD: postgrespwd
                  POSTGRES_DB: review-filmes
        steps:
              - name: Obtendo o código do projeto
                uses: actions/checkout@v4
              - name: Setup dotnet
                uses: actions/setup-dotnet@v4
                with:
                  dotnet-version: "8.0.300"

              - name: Execução de Teste de Integração
                working-directory: ./src
                run: dotnet test ./Review-Filmes.Test.Integration/Review-Filmes.Test.Integration.csproj
                env:  
                  ConnectionStrings__DefaultConnection: "Host=localhost;Port=5432;Database=review-filmes;Username=review;Password=postgrespwd"           
    sonarqube:
        name: Scan com o Sonarqube
        runs-on: ubuntu-latest
        steps:
            - name: Obtendo o código do projeto
              uses: actions/checkout@v4
            - name: Setup JDK
              uses: actions/setup-java@v4
              with:
                distribution: adopt
                java-version: '21'

            - name: Setup dotnet
              uses: actions/setup-dotnet@v4
              with:
                dotnet-version: "8.0.300"
            
            - name: Instalação do Sonarqube Scanner
              run: dotnet tool install --global dotnet-sonarscanner

            - name: Build e analise
              working-directory: ./src
              run: |
                dotnet sonarscanner begin /k:"filme" /d:sonar.host.url="${{ secrets.SONAR_HOST_URL }}"  /d:sonar.login="${{ secrets.SONAR_TOKEN }}"
                dotnet build Review-Filmes.sln
                dotnet sonarscanner end /d:sonar.login="${{ secrets.SONAR_TOKEN }}"
            
            - name: Verificação do Quality Gate
              uses: sonarsource/sonarqube-quality-gate-action@master
              id: sonarqube-quality-gate-check
              env:
                SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
                SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
              with:
                scanMetadataReportFile: ./src/.sonarqube/out/.sonar/report-task.txt
            - name: "Exibir status quality gate"
              if: ${{ always() }}
              run: echo "O status do quality gate é ${{ steps.sonarqube-quality-gate-check.outputs.quality-gate-status }}"


## 📄 Workflow (`.github/workflows/main.yml`)
```yaml

name: Pipeline CI/CD
run-name: Pipeline CI/CD executada por ${{ github.actor }} em {{ github.run_number }}

on:
  workflow_dispatch:
  push:
    branches:
      - main
permissions:
  contents: read
  security-events: write

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Obtendo o código do projeto
        uses: actions/checkout@v4
      - name: Setup dotnet
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: "8.0.300"
      - name: Execução do comando build
        working-directory: ./src
        run: dotnet build Review-Filmes.sln
  testes:
    needs: [build]
    uses: cleitonbarbosa21/pipeline-actions/.github/workflows/testes.yml@main
    secrets: inherit

  release:
    name: Criação de Release
    runs-on: ubuntu-latest
    needs: [testes]
    steps:
      - name: Obtendo o códio do projeto
        uses: actions/checkout@v4

      - name: Analise do Dockerfile
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: ./src/Review-Filmes.Web/Dockerfile

      - name: Efetuando o Login no Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_PWD }}

      - name: Build e Push da Imagem Docker
        uses: docker/build-push-action@v6
        with:
          push: true
          context: ./src
          file: ./src/Review-Filmes.Web/Dockerfile
          tags: |
            cleitonbarbosa21/projeto-devops:latest
            cleitonbarbosa21/projeto-devops:v${{ github.run_number }}

      - name: Executar o Trivy
        uses: aquasecurity/trivy-action@0.28.0
        with:
          scan-type: 'image'
          image-ref: cleitonbarbosa21/projeto-devops:v${{ github.run_number }}
          format: sarif
          output: trivy-docker-result.sarif
          severity: 'UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL'

      - name: Upload do arquivo Trivy para o Github
        uses: github/codeql-action/upload-sarif@v3
        with:
          category: docker-result
          sarif_file: trivy-docker-result.sarif


  deploy:
    needs: [release]
    uses: cleitonbarbosa21/pipeline-actions/.github/workflows/deploy.yml@main
    secrets: inherit

## 📄 Workflow (`.github/workflows/deploy.yml`)
```yaml

deploy.yml
name: Deploy no Kubernetes
on:
    workflow_call: 

jobs:
    deploy:
        name: "Deploy"
        runs-on: ubuntu-latest
        steps:
            - name: Obtendo o código
              uses: actions/checkout@v4
            
            - name: Configuração do context
              uses: azure/k8s-set-context@v4
              with:
                method: kubeconfig
                kubeconfig: ${{ secrets.K8S_CONFIG }}
            
            - name: Deploy no Kubernetes
              uses: Azure/k8s-deploy@v5
              with:
                manifests: |
                    k8s/deployment.yaml
                images: |
                    cleitonbarbosa21/projeto-devops:v${{ github.run_number }}
