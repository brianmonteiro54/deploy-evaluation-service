# Deploy Evaluation Service

Repositório GitOps contendo os manifestos Kubernetes do **Evaluation Service** da plataforma ToggleMaster.

## Visão Geral

Este repositório é monitorado pelo **ArgoCD** e contém exclusivamente os manifestos declarativos de deploy do Evaluation Service. Qualquer alteração nos manifests dispara uma sincronização automática no cluster EKS.

A tag da imagem Docker é atualizada automaticamente pelo pipeline de CI do repositório [`evaluation-service`](https://github.com/brianmonteiro54/evaluation-service) a cada push na branch `main`.

## Manifestos

| Arquivo | Descrição |
|---|---|
| `deployment.yaml` | Deployment com rolling update, probes e variáveis para Redis, SQS e serviços internos |
| `service.yaml` | Service ClusterIP na porta 8004 |
| `ingress.yaml` | Ingress para exposição externa |
| `secretstore.yaml` | SecretStore para integração com AWS Secrets Manager |
| `externalsecret.yaml` | ExternalSecret para injeção segura de credenciais |

## Fluxo GitOps

```
evaluation-service (CI) → push na main → pipeline builda imagem → atualiza tag aqui → ArgoCD sincroniza → EKS
```

## Estrutura

```
├── manifests/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── secretstore.yaml
│   └── externalsecret.yaml
└── README.md
```

## Contexto

Este repositório faz parte do **Tech Challenge Fase 3 — POSTECH**, que implementa práticas de IaC (Terraform), CI/CD com DevSecOps e GitOps (ArgoCD) para os 5 microsserviços do ToggleMaster, rodando em AWS EKS via AWS Academy.
