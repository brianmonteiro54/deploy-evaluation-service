# Deploy Evaluation Service

Repositório GitOps contendo os manifestos Kubernetes do **Evaluation Service** da plataforma **ToggleMaster**.

---

## Visão Geral

Este repositório é monitorado pelo **ArgoCD** e contém exclusivamente os manifestos declarativos de deploy do Evaluation Service. Qualquer alteração nos manifests dispara uma sincronização automática no cluster **AWS EKS**.

A tag da imagem Docker é atualizada automaticamente pelo pipeline de CI do repositório [`evaluation-service`](https://github.com/brianmonteiro54/evaluation-service) a cada push na branch `main`.

## Manifestos

| Arquivo | Descrição |
|---|---|
| `deployment.yaml` | Deployment com rolling update, probes e variáveis para Redis, SQS e serviços internos |
| `service.yaml` | Service ClusterIP na porta 8004 |
| `ingress.yaml` | Ingress para exposição externa |
| `hpa.yaml` | HorizontalPodAutoscaler com escala de 2 a 20 réplicas por CPU |
| `secretstore.yaml` | SecretStore para integração com AWS Secrets Manager |
| `externalsecret.yaml` | ExternalSecret para injeção segura de credenciais |
#### Variáveis de Ambiente

| Variável | Origem | Descrição |
|---|---|---|
| `PORT` | Hardcoded | Porta da aplicação (`8004`) |
| `APP_ENV` | Hardcoded | Ambiente (`production`) |
| `AWS_REGION` | Hardcoded | Região AWS (`us-east-1`) |
| `REDIS_URL` | Init Container | URL completa do Redis (montada em runtime) |
| `FLAG_SERVICE_URL` | Secret | URL do serviço de feature flags |
| `TARGETING_SERVICE_URL` | Secret | URL do serviço de targeting |
| `AWS_SQS_URL` | Secret | URL da fila SQS |
| `SERVICE_API_KEY` | Secret | Chave de API interna entre serviços |
| `AWS_ACCESS_KEY_ID` | Secret | Credencial AWS |
| `AWS_SECRET_ACCESS_KEY` | Secret | Credencial AWS |
| `AWS_SESSION_TOKEN` | Secret | Token de sessão AWS (Academy) |

### `service.yaml`

Expõe o Evaluation Service internamente no cluster como **ClusterIP** na porta **8004**.

### `ingress.yaml`

Configura o Ingress NGINX para roteamento externo:

- **Host:** `toggle.pt`
- **Path:** `/evaluate(/|$)(.*)`
- **Backend:** `evaluation-service:8004`
- **SSL Redirect:** desabilitado

### `hpa.yaml`

Configura o **HorizontalPodAutoscaler** para escalabilidade automática:

- **Réplicas:** mínimo 2, máximo 20
- **Métrica:** utilização de CPU com target de 80%
- **Scale Up:** até 2 pods a cada 15s, sem janela de estabilização
- **Scale Down:** 1 pod a cada 30s, com janela de estabilização de 60s

### `secretstore.yaml`

Define o **SecretStore** para integração com o **AWS Secrets Manager** na região `us-east-1`, autenticando-se via secret `aws-credentials`.

### `externalsecret.yaml`

Configura o **ExternalSecret** que sincroniza credenciais do AWS Secrets Manager para o Kubernetes Secret `evaluation-service-config`. As seguintes chaves são sincronizadas:

- `REDIS_URL`
- `REDIS_PASSWORD`
- `SQS_QUEUE_URL`
- `DYNAMODB_TABLE`
- `AUTH_SERVICE_URL`
- `FLAG_SERVICE_URL`
- `TARGETING_SERVICE_URL`
- `SERVICE_API_KEY`

O intervalo de refresh é de **1 hora**.

## Fluxo GitOps

```
evaluation-service (CI)
        │
        ▼
  Push na main
        │
        ▼
  Pipeline builda imagem Docker
        │
        ▼
  Push para Amazon ECR
        │
        ▼
  Atualiza tag da imagem neste repositório
        │
        ▼
  ArgoCD detecta alteração e sincroniza
        │
        ▼
  Deploy no cluster AWS EKS
```


## Tecnologias

- **Kubernetes** — orquestração de containers
- **ArgoCD** — GitOps e entrega contínua
- **AWS EKS** — cluster Kubernetes gerenciado
- **AWS ECR** — registry de imagens Docker
- **AWS Secrets Manager** — gerenciamento de credenciais
- **AWS SQS** — fila de mensagens
- **External Secrets Operator** — sincronização de secrets
- **NGINX Ingress Controller** — roteamento de tráfego externo

## Contexto

Este repositório faz parte do **Tech Challenge Fase 3 — POSTECH**, que implementa práticas de IaC (Terraform), CI/CD com DevSecOps e GitOps (ArgoCD) para os microsserviços do **ToggleMaster**, rodando em AWS EKS via AWS Academy.
