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

## 📄 Workflow (`.github/workflows/ci-cd.yml`)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

permissions:
  contents: read
  security-events: write

jobs:
  build-test-sonar-docker-k8s:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
