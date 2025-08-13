# 🚀 CI/CD com GitHub Actions + SonarQube + Docker Hub

![GitHub Actions](https://img.shields.io/github/actions/workflow/status/usuario/repositorio/ci-cd.yml?branch=main&label=CI/CD)
![SonarQube Quality Gate](https://img.shields.io/badge/SonarQube-Quality%20Gate-blue)
![Docker Hub](https://img.shields.io/badge/Docker%20Hub-Automated%20Build-blue)

Este projeto utiliza **GitHub Actions** para implementar um pipeline completo de **Integração Contínua (CI)** e **Entrega Contínua (CD)**, incluindo:

- 📦 **Build** da aplicação
- 🧪 **Testes automatizados**
- 🔍 **Análise de qualidade** com SonarQube
- 🐳 **Deploy da imagem** no Docker Hub

---

## 📋 Fluxo do Pipeline

1. **Build e testes**
   - Compila a aplicação.
   - Executa testes unitários.
   - Gera relatórios de cobertura.

2. **Análise de qualidade**
   - Usa **SonarQube** para avaliar código.
   - Aplica **Quality Gates**.

3. **Build e Deploy no Docker Hub**
   - Cria imagem Docker.
   - Faz push da imagem para o repositório no Docker Hub.

---

## 🔑 Pré-requisitos

- Conta no [GitHub](https://github.com)
- Conta no [Docker Hub](https://hub.docker.com/)
- Servidor **SonarQube** configurado (local ou remoto)
- Docker instalado localmente para desenvolvimento

---

## ⚙️ Configuração de Secrets no GitHub

No repositório, acesse **Settings > Secrets and variables > Actions** e crie:

| Nome                | Descrição                                  |
|---------------------|--------------------------------------------|
| `DOCKERHUB_USERNAME`| Usuário do Docker Hub                      |
| `DOCKERHUB_TOKEN`   | Token ou senha do Docker Hub                |
| `SONAR_HOST_URL`    | URL do servidor SonarQube                   |
| `SONAR_TOKEN`       | Token gerado no SonarQube                   |

---

## 📄 Workflow (`.github/workflows/ci-cd.yml`)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build-test-sonar-docker:
    runs-on: ubuntu-latest

    steps:
      # Checkout do código
      - name: Checkout repository
        uses: actions/checkout@v4

      # Configuração do Java (exemplo para Maven)
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      # Build e Testes
      - name: Build and Test
        run: |
          ./mvnw clean install
          ./mvnw test

      # Análise com SonarQube
      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@v2
        with:
          args: >
            -Dsonar.projectKey=meu-projeto
            -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }}
            -Dsonar.login=${{ secrets.SONAR_TOKEN }}

      # Login no Docker Hub
      - name: Docker Login
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # Build da imagem Docker
      - name: Build Docker image
        run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/meu-app:latest .

      # Push da imagem para o Docker Hub
      - name: Push Docker image
        run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/meu-app:latest
