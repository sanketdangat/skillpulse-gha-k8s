# SkillPulse — GitOps-Based Three-Tier Application on Amazon EKS

![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/argo-%23168ECA.svg?style=for-the-badge&logo=argo&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)
![Go](https://img.shields.io/badge/go-%2300ADD8.svg?style=for-the-badge&logo=go&logoColor=white)
![MySQL](https://img.shields.io/badge/mysql-%234479A1.svg?style=for-the-badge&logo=mysql&logoColor=white)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)
![Prometheus](https://img.shields.io/badge/prometheus-%23E6522C.svg?style=for-the-badge&logo=prometheus&logoColor=white)

---

## Project Overview

SkillPulse is a production-oriented, cloud-native three-tier application deployed on Amazon EKS using modern DevOps and GitOps practices.

The application enables users to track the skills they are learning along with the time invested in developing each skill.

The application stack consists of:

- Go backend API
- Vanilla JavaScript frontend served through Nginx
- MySQL database

Beyond the application itself, the primary focus of this repository is the Kubernetes platform engineering ecosystem built around it, including:

- Amazon EKS
- Terraform infrastructure provisioning
- GitOps deployments using ArgoCD
- GitHub Actions CI/CD automation
- AWS-native ingress, storage, and secret management
- Monitoring and observability
- Security automation and vulnerability scanning

---

## Original Project Foundation

This repository is derived from `LondheShubham153/github-actions-kubernetes-masterclass`, which originally provided:

- Dockerfile for the Go backend service
- Dockerfile for the Nginx frontend service
- `docker-compose.yml` for local development
- A basic GitHub Actions pipeline
- SSH-based deployment workflow using Docker Compose on a remote VM

The original implementation has since been extensively redesigned into a production-style Kubernetes and GitOps platform running on Amazon EKS.

---

## Enhancements

This repository **significantly extends and redesigns** that baseline application into a production-oriented Kubernetes platform by implementing:

- Terraform-based AWS infrastructure provisioning
- Amazon EKS cluster automation
- GitHub OIDC authentication with AWS IAM
- GitOps deployments using ArgoCD
- Kubernetes-native deployment architecture
- AWS Secrets Manager integration via CSI Driver and ASCP
- EBS CSI dynamic persistent storage provisioning
- Horizontal Pod Autoscaling
- AWS Load Balancer Controller ingress management
- Route53 + ACM TLS integration
- Centralized monitoring stack deployment
- Security scanning pipelines using:
  - Gitleaks
  - Hadolint
  - Govulncheck
  - Trivy
- Immutable image versioning using Git commit SHAs
- Fully automated CI/CD workflows using GitHub Actions

The result is a **complete cloud-native deployment platform** that reflects modern DevOps and GitOps operational practices rather than a simple containerized application deployment.

---

# SkillPulse Architecture Diagram

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                          EXTERNAL ENDPOINTS (Internet)                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────┐  ┌────────────────────────────────┐     │
│  │ skillpulse.cloud2devops.online │  │ argocd.cloud2devops.online     │     │
│  └────────────────────────────────┘  └────────────────────────────────┘     │
│                                                                             │
│               ┌────────────────────────────────┐                            │
│               │ grafana.cloud2devops.online    │                            │
│               └────────────────────────────────┘                            │
│                                   ↓                                         │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │          Amazon Route53 (DNS) + AWS ACM (TLS Certificates)            │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│              AWS REGION (ap-south-1) - VPC Infrastructure                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                AWS Application Load Balancer (ALB)                   │   │
│  │         [Shared ALB Group - cloud2devops-ingress-alb]                │   │
│  │         Managed by AWS Load Balancer Controller (LBC)                │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│      Routes traffic based on hostname rules                                 │ 
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         Amazon EKS Cluster                           │   │
│  │              (Kubernetes Control Plane - AWS Managed)                │   │
│  ├──────────────────────────────────────────────────────────────────────┤   │
│  │                                                                      │   │
│  │  ┌────────────────────────────────────────────────────────────────┐  │   │
│  │  │                EKS Worker Nodes (Private)                      │  │   │
│  │  │ • Auto Scaling Groups • Pod Identity Agent • CloudWatch        │  │   │
│  │  │                                                                │  │   │
│  │  │ ┌────────────────────────┐ ┌───────────────────────────────┐   │  │   │
│  │  │ │ Skillpulse Namespace   │ │ System & Addon Namespaces     │   │  │   │
│  │  │ ├────────────────────────┤ ├───────────────────────────────┤   │  │   │
│  │  │ │                        │ │ • kube-system                 │   │  │   │
│  │  │ │ Frontend Deployment    │ │ • kube-public                 │   │  │   │
│  │  │ │ (Nginx + JavaScript)   │ │ • argocd                      │   │  │   │
│  │  │ │ └─ Service (ClusterIP) │ │ • monitoring                  │   │  │   │
│  │  │ │                        │ │                               │   │  │   │
│  │  │ │ Backend Deployment     │ │ Core Add-ons:                 │   │  │   │
│  │  │ │ (Go API)               │ │ • AWS Load Balancer           │   │  │   │
│  │  │ │ └─ Service (ClusterIP) │ │   Controller (LBC)            │   │  │   │
│  │  │ │ └─ HPA (Horizontal     │ │ • EBS CSI Driver              │   │  │   │
│  │  │ │    Pod Autoscaling)    │ │ • Metrics Server              │   │  │   │
│  │  │ │                        │ │ • Secrets Store CSI Driver    │   │  │   │
│  │  │ │ MySQL StatefulSet      │ │ • ASCP (AWS Secrets           │   │  │   │
│  │  │ │ • Service (ClusterIP)  │ │   Manager Provider)           │   │  │   │
│  │  │ │ • PVC (EBS Volume)     │ │                               │   │  │   │
│  │  │ │ • ConfigMap            │ │ ArgoCD:                       │   │  │   │
│  │  │ │ • ServiceAccount       │ │ • argocd-server               │   │  │   │
│  │  │ │                        │ │ • argocd-repo-server          │   │  │   │
│  │  │ │ └─ Secrets (from AWS   │ │ • argocd-controller           │   │  │   │
│  │  │ │    Secrets Manager)    │ │ • argocd-dex-server           │   │  │   │
│  │  │ │                        │ │ • Ingress Resource            │   │  │   │
│  │  │ │ Ingress Resource       │ │   (argocd-ingress)            │   │  │   │
│  │  │ │ (skillpulse-ingress)   │ │                               │   │  │   │
│  │  │ │                        │ │ Monitoring Stack:             │   │  │   │
│  │  │ │                        │ │ • Prometheus                  │   │  │   │
│  │  │ │                        │ │ • Grafana                     │   │  │   │
│  │  │ │                        │ │ • Node Exporter               │   │  │   │
│  │  │ │                        │ │ • Kube-State-Metrics          │   │  │   │
│  │  │ │                        │ │ • Ingress Resource            │   │  │   │
│  │  │ │                        │ │   (grafana-ingress)           │   │  │   │
│  │  │ └────────────────────────┘ └───────────────────────────────┘   │  │   │
│  │  │                                                                │  │   │
│  │  │ ┌──────────────────────────────────────────────────────────┐   │  │   │
│  │  │ │         Kubernetes Add-ons & CSI Drivers                 │   │  │   │
│  │  │ │ • AWS Load Balancer Controller (LBC)                     │   │  │   │
│  │  │ │ • EBS CSI Driver (Dynamic Storage Provisioning)          │   │  │   │
│  │  │ │ • Secrets Store CSI Driver + ASCP                        │   │  │   │
│  │  │ │ • Metrics Server (HPA & Resource Monitoring)             │   │  │   │
│  │  │ │ • Pod Identity Agent (IAM for Pods)                      │   │  │   │
│  │  │ └──────────────────────────────────────────────────────────┘   │  │   │
│  │  └────────────────────────────────────────────────────────────────┘  │   │
│  │                                                                      │   │
│  │  ┌──────────────────────────────────────────────────────────────┐    │   │
│  │  │                     VPC Networking                           │    │   │
│  │  │ • Private Subnets (2 AZs) for EKS Nodes                      │    │   │
│  │  │ • Public Subnets (2 AZs) for ALB & NAT Gateways              │    │   │
│  │  │ • NAT Gateways for Egress Traffic                            │    │   │
│  │  │ • Security Groups for EKS & ALB                              │    │   │
│  │  └──────────────────────────────────────────────────────────────┘    │   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                 AWS External Services & Storage                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ • Amazon ECR (Elastic Container Registry)                                   │
│   - Stores Docker images for backend & frontend                             │
│                                                                             │
│ • AWS Secrets Manager                                                       │
│   - Manages database credentials and secrets                                │
│                                                                             │
│ • Amazon EBS (Elastic Block Storage)                                        │
│   - Persistent volumes for MySQL database                                   │
│                                                                             │
│ • AWS S3 (Terraform State Backend)                                          │
│   - Stores Terraform state files (vpc & eks)                                │
│                                                                             │
│ • AWS Systems Manager Parameter Store                                       │
│   - Optional parameter storage alongside Secrets Manager                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# Data Flow Architecture

## 1. User Request Flow

```text
Internet User
    ↓
skillpulse.cloud2devops.online
    ↓
Route53 DNS Resolution
    ↓
AWS Application Load Balancer (ALB)
    ├─ ACM TLS Certificate Attached
    └─ TLS Termination Happens Here
    ↓
ALB Listener Rules (Host-based Routing)
    ↓
Kubernetes Ingress Resource
    ↓
Frontend Service (ClusterIP)
    ↓
Frontend Pod (Nginx + JavaScript)
    ↓
Backend Service (ClusterIP)
    ↓
Backend Pod (Go API)
    ↓
MySQL Service
    ↓
MySQL StatefulSet Pod

(Response travels back through Backend → Frontend → Services → Ingress → ALB)

    ↓
HTTPS Response Returned to User
```

---

## 2. CI/CD Deployment Flow (GitOps)
```text
Developer / DevOps Engineer
    ↓
Pushes Code to GitHub
    ↓
GitHub Actions CI Pipeline Triggered

    ├─ Security Job
    │   ├─ Checkout Repository
    │   ├─ Gitleaks Secret Scan
    │   ├─ Hadolint Scan (Backend Dockerfile)
    │   ├─ Hadolint Scan (Frontend Dockerfile)
    │   ├─ Setup Go Environment
    │   └─ Govulncheck Scan
    │
    ├─ Build, Scan & Push Job (Matrix Strategy)
    │   ├─ Matrix Services
    │   │    ├─ backend
    │   │    └─ frontend
    │   │
    │   ├─ Docker Buildx Setup
    │   ├─ Generate Image Tag using Git Commit SHA
    │   ├─ Configure AWS Authentication (OIDC)
    │   ├─ Login to Amazon ECR
    │   ├─ Build Docker Images
    │   ├─ Trivy Vulnerability Scan
    │   └─ Push Images to Amazon ECR
    │
    └─ CI Pipeline Completed Successfully

    ↓

GitHub Actions Updates GitOps Repository
    ↓
Update Kubernetes Manifests (k8s/)
    ├─ Update Backend Image Tag
    ├─ Update Frontend Image Tag
    └─ Pin Images using Git Commit SHA
    ↓

Commit & Push Updated Manifests to GitHub
    ↓
ArgoCD Watches Git Repository
    ↓
Detects Manifest Changes
    ↓
ArgoCD Reconciles Desired State
    ↓
Kubernetes Applies Updated Manifests
    ├─ Deploy New Pods
    ├─ Terminate Old Pods
    └─ HPA Adjusts Replicas Automatically
    ↓

Application Updated on Amazon EKS Cluster
```

---

## 3. Secrets Management Flow

```text
AWS Secrets Manager
    ↓
ASCP (AWS Secrets Manager Provider)
    ↓
Secrets Store CSI Driver
    ↓
SecretProviderClass Resource
    ↓
Mounted as Volume Inside Pod
    ↓
Backend Pod / MySQL StatefulSet Pod
    ↓
Backend / MySQL Access Credentials Available Securely
```

---

## 4. Storage & Persistence Flow

```text
Kubernetes PersistentVolumeClaim (PVC)
    ↓
EBS CSI Driver
    ↓
Amazon EBS API
    ↓
Amazon EBS Volume (gp3) Provisioned
    ↓
PersistentVolume (PV) Created & Bound to PVC
    ↓
Mounted into MySQL StatefulSet Pod
    ↓
Persistent MySQL Storage
```
---

## 5. Monitoring & Observability Flow

```text
Kubernetes Cluster
    ↓
Metrics Exporters & Resource Metrics
    ├─ Application Metrics
    ├─ Node Exporter Metrics
    ├─ kube-state-metrics
    └─ Kubernetes Metrics Server
    ↓
Prometheus Scrapes & Stores Metrics
    ↓
Grafana Pod Queries Prometheus

────────────────────────

DevOps Engineer
    ↓
grafana.cloud2devops.online
    ↓
Route53 DNS Resolution
    ↓
AWS Application Load Balancer (ALB)
    ├─ ACM TLS Certificate Attached
    └─ TLS Termination Happens Here
    ↓
ALB Listener Rules (Host-based Routing)
    ↓
Grafana Ingress Resource
    ↓
Grafana Service
    ↓
Grafana Pod
    ↓
Grafana Dashboards Visualize Metrics
```
---

## 6. ArgoCD Access & GitOps Reconciliation Flow
```text
DevOps Engineer
    ↓
argocd.cloud2devops.online
    ↓
Route53 DNS Resolution
    ↓
AWS Application Load Balancer (ALB)
    ├─ ACM TLS Certificate Attached
    └─ TLS Termination Happens Here
    ↓
ALB Listener Rules (Host-based Routing)
    ↓
ArgoCD Ingress Resource
    ↓
argocd-server Service
    ↓
argocd-server Pod
    ↓
ArgoCD Watches Git Repository
    ↓
Detects Kubernetes Manifest Changes
    ↓
ArgoCD Reconciles Desired State
    ↓
Kubernetes Cluster State Updated
```

---

## Tech Stack

| Category | Technology |
|---|---|
| Cloud Provider | AWS |
| Container Orchestration | Amazon EKS |
| Infrastructure as Code | Terraform |
| GitOps | ArgoCD |
| CI/CD | GitHub Actions |
| Container Registry | Amazon ECR |
| Backend | Go |
| Frontend | Nginx + JavaScript |
| Database | MySQL |
| Ingress | AWS Load Balancer Controller |
| Secrets Management | AWS Secrets Manager |
| Kubernetes Secrets Integration | Secrets Store CSI + ASCP |
| Persistent Storage | Amazon EBS CSI Driver |
| Monitoring | Prometheus + Grafana |
| DNS | Route53 |
| TLS Certificates | AWS ACM |
| Security Scanning | Trivy, Gitleaks, Hadolint, Govulncheck |

---

## Kubernetes Components

- Deployments
- StatefulSets
- Services
- Ingress Resources
- Horizontal Pod Autoscaler (HPA)
- PersistentVolumeClaims (PVC)
- ConfigMaps
- ServiceAccounts
- SecretProviderClass

---

## Features

- Terraform-managed AWS infrastructure
- Amazon EKS cluster provisioning
- GitOps-based Kubernetes deployments
- ArgoCD automated reconciliation
- GitHub Actions CI/CD pipelines
- Secure GitHub OIDC authentication
- AWS Application Load Balancer (ALB) ingress
- Route53 DNS integration
- ACM-managed HTTPS/TLS
- Secrets Manager integration using CSI Driver
- Dynamic EBS persistent storage provisioning
- Kubernetes Horizontal Pod Autoscaling
- Immutable Docker image deployments
- Security scanning integrated into CI
- Monitoring stack deployment with Grafana & Prometheus
- Production-style Kubernetes manifest orchestration using ArgoCD sync waves

---

## Security Highlights

- GitHub OIDC federation with AWS IAM
- No long-lived AWS credentials stored in GitHub
- Secrets stored securely in AWS Secrets Manager
- Pod-level IAM access using EKS Pod Identity
- Vulnerability scanning integrated into CI pipeline
- HTTPS enforced using ACM certificates
- Kubernetes readiness, liveness, and startup probes
- Immutable image deployments using commit SHAs

---

## Monitoring Stack

The cluster includes:

- **Prometheus** — Metrics collection and querying
- **Grafana** — Visualization dashboards
- **Kubernetes Metrics Server** — Resource utilization metrics

Monitoring dashboards are exposed securely through AWS ALB Ingress with HTTPS enabled using ACM certificates.

---

## Public Endpoints

| Service | URL |
|---|---|
| SkillPulse Application | https://skillpulse.cloud2devops.online |
| ArgoCD Dashboard | https://argocd.cloud2devops.online |
| Grafana Dashboard | https://grafana.cloud2devops.online |

> Note: Public endpoints are exposed securely through AWS ALB with HTTPS termination using ACM-managed TLS certificates.
---

## Getting Started

Follow the documentation in this order to set up the complete platform:

1. **[prerequisites.md](prerequisites.md)** — Configure AWS OIDC, GitHub Secrets, Route53, ACM, ECR, S3, and Secrets Manager.
2. **[infra.md](infra.md)** — Provision VPC and EKS cluster using Terraform.
3. **[deployment.md](deployment.md)** — Connect to EKS, verify add-ons, deploy ingress, configure IAM, and deploy the application via ArgoCD.
4. **[github-actions.md](github-actions.md)** — Understand the CI/CD pipeline for automated builds and deployments.
