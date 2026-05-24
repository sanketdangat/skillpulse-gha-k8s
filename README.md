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

SkillPulse is a three-tier web application that allows users to track the skills they are learning and the time invested in each skill. The application layer is intentionally lightweight, consisting of a Go backend API, a vanilla JavaScript frontend served via Nginx, and a MySQL database.

The primary value of this repository lies in the platform engineering layer built around the application. It focuses on secure infrastructure provisioning, GitOps-based delivery, automated security enforcement, secret management, and fully automated cloud-native deployments on Kubernetes.

---

## What Already Existed

This project is forked from `LondheShubham153/github-actions-kubernetes-masterclass`, which provided the initial application baseline and a simple deployment approach.

It included:

- Dockerfile for the Go backend service
- Dockerfile for the Nginx frontend service
- docker-compose.yml for local three-tier development
- A basic CI/CD pipeline that deployed the application via SSH and executed `docker compose up` on every push to `main`

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

## SkillPulse Architecture

```mermaid
graph TD

%% =====================================================
%% NETWORK SUMMARY (DETAILED CIDR INFO)
%% =====================================================

CIDR_VPC["AWS VPC: 10.0.0.0/16<br/>Used by: EC2 Worker Nodes, Pods (via VPC CNI using VPC IPs), ALB, NAT Gateway"]
CIDR_SVC["Kubernetes Service CIDR: 172.20.0.0/16<br/>Used by: ClusterIP Services (frontend, backend, mysql)<br/>Internal-only Kubernetes service networking (not routable in VPC)"]

%% =====================================================
%% EDGE / INTERNET LAYER
%% =====================================================

User["End User (https://skillpulse.cloud2devops.online)"] --> Route53["Amazon Route 53"]
Route53 --> ALB["AWS Application Load Balancer"]

ACM["AWS Certificate Manager"] -.->|TLS Certificate| ALB

%% =====================================================
%% AWS CLOUD
%% =====================================================

subgraph AWS["AWS Cloud (ap-south-1)"]

    %% =================================================
    %% EKS CLUSTER
    %% =================================================

    subgraph EKS["Amazon EKS Cluster (dev-skillpulse-eks)"]

        %% CONTROL PLANE
        subgraph ControlPlane["Managed EKS Control Plane"]

            APIServer["Kubernetes API Server"]

        end

        %% ADDONS
        subgraph Addons["Cluster Add-ons"]

            LBC["AWS Load Balancer Controller"]
            EBSCSI["Amazon EBS CSI Driver"]
            ASCP["Secrets Store CSI Driver + AWS Provider"]
            PodIdentity["EKS Pod Identity Agent"]
            Metrics["Metrics Server"]

        end

        %% INGRESS
        Ingress["Kubernetes Ingress"]

        %% =============================================
        %% APPLICATION NAMESPACE
        %% =============================================

        subgraph SkillPulse["Namespace: skillpulse"]

            FE_SVC["frontend-service (ClusterIP)"]
            FE_POD["Frontend Pod"]

            BE_SVC["backend-service (ClusterIP)"]
            BE_POD["Backend Pod"]

            MYSQL_SVC["mysql-service (ClusterIP)"]
            MYSQL_STS["MySQL StatefulSet"]

            PVC["PersistentVolumeClaim"]

            SA["skillpulse-sa"]
            SPC["SecretProviderClass"]

            FE_SVC --> FE_POD
            BE_SVC --> BE_POD

            MYSQL_SVC --> MYSQL_STS

            FE_POD -->|API Calls| BE_SVC
            BE_POD -->|SQL Queries| MYSQL_SVC

            MYSQL_STS --> PVC

        end

        %% =============================================
        %% ARGOCD
        %% =============================================

        subgraph ArgoNS["Namespace: argocd"]

            ArgoSvc["ArgoCD Server Service"]
            ArgoCD["ArgoCD Server"]

            ArgoSvc --> ArgoCD

        end

        %% =============================================
        %% MONITORING
        %% =============================================

        subgraph MonitorNS["Namespace: monitoring"]

            GrafSvc["grafana-service"]
            Grafana["Grafana Pod"]

            Prometheus["Prometheus Pod"]

            GrafSvc --> Grafana

            Grafana -->|Query Metrics| Prometheus

            Prometheus -->|Scrape Metrics| FE_POD
            Prometheus -->|Scrape Metrics| BE_POD

        end

    end

    %% =================================================
    %% STORAGE
    %% =================================================

    EBS["Amazon EBS Volume (gp3)"]

    PVC --> EBSCSI
    EBSCSI --> EBS

    %% =================================================
    %% SECURITY
    %% =================================================

    subgraph Security["Security & Identity"]

        ASM["AWS Secrets Manager"]
        IAMRole["IAM Role: skillpulse-db-secrets-role"]

    end

    %% =================================================
    %% AMAZON ECR
    %% =================================================

    ECR["Amazon ECR"]

end

%% =====================================================
%% INGRESS / ALB FLOW
%% =====================================================

LBC -->|Creates & Manages| ALB

ALB --> Ingress

Ingress -->|skillpulse.cloud2devops.online| FE_SVC
Ingress -->|argocd.cloud2devops.online| ArgoSvc
Ingress -->|grafana.cloud2devops.online| GrafSvc

%% =====================================================
%% POD IDENTITY & SECRETS FLOW
%% =====================================================

BE_POD --> SA
SA -->|Pod Identity Association| IAMRole
IAMRole --> ASM

SPC --> ASCP
ASCP --> ASM
ASCP -->|Mount Secrets| BE_POD

%% =====================================================
%% IMAGE PULL FLOW
%% =====================================================

FE_POD -.->|Pull Image| ECR
BE_POD -.->|Pull Image| ECR

%% =====================================================
%% GITHUB ACTIONS / CI-CD
%% =====================================================

subgraph GitHub["GitHub Actions CI-CD"]

    GitRepo["GitHub Repository"]

    CI["CI Pipeline<br/>- Security Scan<br/>- Build Docker Image<br/>- Push to ECR"]

    CD["CD Pipeline<br/>- Update Kubernetes Manifests<br/>- Commit GitOps Changes"]

end

Developer["Developer"] -->|Push Code| GitRepo

GitRepo --> CI

CI --> ECR

CI --> CD

CD --> GitRepo

%% =====================================================
%% GITOPS FLOW
%% =====================================================

GitRepo -->|Watch Desired State| ArgoCD

ArgoCD -->|Apply Kubernetes Manifests| APIServer

APIServer --> FE_POD
APIServer --> BE_POD
APIServer --> MYSQL_STS
APIServer --> Grafana
```

---

## GitOps Deployment Model

This project follows GitOps principles using ArgoCD.

**Deployment flow:**

```text
GitHub Actions builds container images
            ↓
Images pushed to Amazon ECR
            ↓
CD workflow updates Kubernetes manifests
            ↓
ArgoCD detects repository changes
            ↓
Kubernetes workloads automatically reconcile
```

This provides:

- Declarative deployments
- Auditability
- Rollback capability
- Immutable release tracking
- Automated synchronization

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

## Features

- GitOps-based Kubernetes deployments
- Terraform-managed AWS infrastructure
- Secure GitHub OIDC authentication
- Amazon EKS cluster provisioning
- AWS-native ingress with ALB
- Route53 DNS integration
- ACM-managed HTTPS/TLS
- Kubernetes Horizontal Pod Autoscaling
- Dynamic EBS persistent storage provisioning
- Secrets Manager integration using CSI Driver
- Immutable Docker image deployments
- Automated GitHub Actions CI/CD pipelines
- Security scanning integrated into CI
- ArgoCD automated reconciliation
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

- **Prometheus** — Metrics collection and alerting
- **Grafana** — Visualization dashboards
- **Kubernetes Metrics Server** — Resource utilization metrics

Monitoring dashboards are exposed securely through AWS ALB Ingress with HTTPS enabled using ACM certificates.

---

## Application, ArgoCD & Grafana Endpoints

| Service | URL |
|---|---|
| SkillPulse Application | https://skillpulse.cloud2devops.online |
| ArgoCD Dashboard | https://argocd.cloud2devops.online |
| Grafana Dashboard | https://grafana.cloud2devops.online |

---

## Getting Started

Follow the documentation in this order to set up the complete platform:

1. **[prerequisites.md](prerequisites.md)** — Configure AWS OIDC, GitHub Secrets, Route53, ACM, ECR, S3, and Secrets Manager.
2. **[infra.md](infra.md)** — Provision VPC and EKS cluster using Terraform.
3. **[deployment.md](deployment.md)** — Connect to EKS, verify add-ons, deploy ingress, configure IAM, and deploy the application via ArgoCD.
4. **[github-actions.md](github-actions.md)** — Understand the CI/CD pipeline for automated builds and deployments.

---

## Documentation

| Document | Description |
|---|---|
| [prerequisites.md](prerequisites.md) | AWS, GitHub, Route53, ACM, and ECR setup |
| [infra.md](infra.md) | Terraform infrastructure provisioning |
| [deployment.md](deployment.md) | Application deployment and post-deployment steps |
| [github-actions.md](github-actions.md) | CI/CD pipeline explanation |
