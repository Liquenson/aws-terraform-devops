# AWS Terraform DevOps

Infraestructura AWS provisionada con Terraform, ejecutando una app Flask en Amazon EKS con RDS PostgreSQL. Pipeline CI/CD con quality gates obligatorios y seguridad aplicada en cada capa.

[![CI/CD](https://github.com/lra-cloud-ops/aws-terraform-devops/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/lra-cloud-ops/aws-terraform-devops/actions/workflows/ci-cd.yml)
[![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=Liquenson_aws-terraform-devops-lab&metric=alert_status)](https://sonarcloud.io/dashboard?id=Liquenson_aws-terraform-devops-lab)
[![Terraform](https://img.shields.io/badge/Terraform-1.9.8-7B42BC?logo=terraform)](https://www.terraform.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Arquitectura

```
Internet → Load Balancer → EKS (Flask, 2–10 pods, HPA) → RDS PostgreSQL (Multi-AZ ready)
```

| Capa | Implementación |
|---|---|
| IaC | Terraform modular, state remoto S3 + DynamoDB lock |
| Cómputo | EKS v1.31, auto-scaling 1–4 nodos |
| Datos | RDS PostgreSQL 15, encriptación en reposo |
| Redes | VPC en 3 AZs, subnets públicas/privadas segregadas |
| CI/CD | GitHub Actions, auth OIDC (sin access keys) |
| Calidad | SonarCloud, quality gate obligatorio |

---

## Seguridad — lo que realmente importa aquí

Este proyecto trata la seguridad como código, no como paso posterior:

| Control | Detalle |
|---|---|
| **Mínimo privilegio** | Roles IAM dedicados por componente (cluster, nodos) — sin permisos amplios compartidos |
| **Sin credenciales estáticas** | CI/CD autenticado vía OIDC; trust policy acotada a este repo |
| **Red segmentada** | RDS solo accesible desde dentro de la VPC (puerto 5432 restringido) |
| **Secrets fuera del código** | GitHub Secrets — nada hardcodeado en el repositorio |
| **Contenedores endurecidos** | Usuario no-root, imagen base mínima |
| **Cambios auditables** | `terraform plan` automático en CI; el `apply` es manual y deliberado |
| **Calidad como gate** | Cobertura > 80% y cero vulnerabilidades críticas bloquean el merge |

## Quick Start

```bash
git clone https://github.com/lra-cloud-ops/aws-terraform-devops.git
cd aws-terraform-devops/terraform

export TF_VAR_db_password="<contraseña-segura>"
terraform init
terraform plan  -var-file=../environments/dev/terraform.tfvars
terraform apply -var-file=../environments/dev/terraform.tfvars

aws eks update-kubeconfig --region eu-west-1 --name dev-cluster
kubectl get nodes
```

Requiere: Terraform 1.9.8+, AWS CLI 2.x, kubectl 1.31+, credenciales AWS con permisos sobre VPC/EKS/RDS/ECR/IAM/CloudWatch.

## Estructura

```
terraform/      # Orquestación raíz
modules/        # vpc, eks, rds, ecr, iam, cloudwatch, s3_bucket — un módulo por servicio
environments/   # Variables por entorno (dev, prod)
docker/         # App Flask + Dockerfile multi-stage
kubernetes/     # deployment, service, hpa
monitoring/     # Prometheus + Grafana
```

## Pipeline

```
push a main → SonarCloud → terraform plan → build Docker → push ECR → deploy EKS (solo tags v*)
```

Gates: cobertura >80%, cero vulnerabilidades críticas, `plan` limpio.

## Multi-cloud (en progreso)

Este repo empieza a extenderse más allá de AWS bajo el mismo state de Terraform:

| Recurso | Estado |
|---|---|
| Provider `google` configurado junto a `aws` | ✅ |
| Módulo `modules/gcs_bucket` (Cloud Storage, acceso público bloqueado, versionado) | ✅ verificado con `terraform plan` |
| Workload Identity Federation para CI/CD (equivalente a OIDC en AWS) | ✅ |
| `terraform apply` en GCP | ⏳ pendiente |

## Roadmap

- [x] IaC completo, CI/CD, observabilidad, auto-scaling, OIDC
- [x] Provider GCP + primer módulo (GCS)
- [x] Autenticación GCP en CI/CD (Workload Identity Federation)
- [ ] ArgoCD (GitOps)
- [ ] Escaneo de imágenes (Trivy) + políticas de admisión (Kyverno)

---

**Ruben Liquenson** — DevOps/Cloud Engineer | AWS · Terraform · Kubernetes
[LinkedIn](https://www.linkedin.com/in/ruben-liquenson-490961269/) · [GitHub](https://github.com/Liquenson) · [lracloudops.com](https://lracloudops.com/es/) · Las Palmas de Gran Canaria, España

Licencia: MIT