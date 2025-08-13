# 🎬 Review-Filmes - Pipeline CI/CD

![GitHub Actions](https://img.shields.io/github/actions/workflow/status/cleitonbarbosa21/review-filmes/main.yml?branch=main&label=CI/CD)
![SonarQube](https://img.shields.io/badge/Code%20Quality-SonarQube-blue)
![Kubernetes](https://img.shields.io/badge/Deploy-Kubernetes-blue)

Este repositório implementa um pipeline **CI/CD** completo usando **GitHub Actions**, com:

- 📦 **Build** do projeto em .NET 8
- 🧪 **Testes automatizados** (unitários e integração)
- 🔍 **Análise de qualidade** com SonarQube
- 🐳 **Build e push da imagem Docker** no Docker Hub
- 🛡️ **Análise de vulnerabilidades** com Trivy
- ☸️ **Deploy no Kubernetes** (DigitalOcean Kubernetes)

---

## 📋 Fluxo do Pipeline

1. **Build**
   - Compila a aplicação em .NET 8.

2. **Testes**
   - Testes unitários.
   - Testes de integração com PostgreSQL em container.
   - Verificação de qualidade no **SonarQube**.

3. **Release**
   - Análise do Dockerfile com Hadolint.
   - Build e push da imagem para o **Docker Hub**.
   - Scan de segurança da imagem com **Trivy**.
   - Upload do relatório de vulnerabilidades para o GitHub Security.

4. **Deploy**
   - Deploy automático no cluster Kubernetes (DigitalOcean).

---

## 🔑 Secrets necessários

No repositório, vá em **Settings > Secrets and variables > Actions** e configure:

| Nome              | Descrição                                                    |
|-------------------|--------------------------------------------------------------|
| `SONAR_HOST_URL`  | URL do servidor SonarQube                                     |
| `SONAR_TOKEN`     | Token de autenticação do SonarQube                            |
| `DOCKERHUB_USER`  | Usuário do Docker Hub                                         |
| `DOCKERHUB_PWD`   | Senha ou token do Docker Hub                                  |
| `K8S_CONFIG`      | `kubeconfig` do cluster DigitalOcean Kubernetes (Base64)      |

> Para gerar o `K8S_CONFIG` em base64:  
> ```bash
> doctl kubernetes cluster kubeconfig save <CLUSTER_NAME>
> cat ~/.kube/config | base64 -w 0
> ```

---

```text
📂 Estrutura do Projeto
.
├── src
│ ├── Review-Filmes.Web # Aplicação principal
│ ├── Review-Filmes.Test.Unit # Testes unitários
│ ├── Review-Filmes.Test.Integration # Testes de integração
│ ├── Review-Filmes.sln
│ └── Dockerfile
├── k8s
│ ├── deployment.yaml # Manifesto de deploy Kubernetes
│ 
└── .github
└── workflows
├── main.yml # Pipeline CI/CD principal
├── testes.yml # Execução de testes e SonarQube
└── deploy.yml # Deploy no Kubernetes
```


---

## 🚀 Executando o pipeline

O pipeline é executado automaticamente quando há **push** para a branch `main` ou manualmente via **workflow_dispatch**.

### 1️⃣ Build e Testes
- Compila o código.
- Executa testes unitários e de integração.
- Analisa a qualidade no SonarQube.

### 2️⃣ Release
- Verifica o Dockerfile com Hadolint.
- Cria e envia imagem para o Docker Hub.
- Executa scan de vulnerabilidades com Trivy.

### 3️⃣ Deploy
- Atualiza o deployment no Kubernetes.
- Executa rolling update com zero downtime.

---

📌 Boas práticas aplicadas
- Quality Gate no SonarQube para impedir deploy de código com baixa qualidade.
- Scan de segurança da imagem antes do deploy.
- Rolling updates no Kubernetes para evitar downtime.
- Versionamento automático das imagens usando github.run_number.
