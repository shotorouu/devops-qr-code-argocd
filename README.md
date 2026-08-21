# Production-Grade GitOps QR Code Generator Platform

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.31.14-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Containerization-Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![GitLab CI](https://img.shields.io/badge/CI%2FCD-GitLab_CI-FCA121?logo=gitlab&logoColor=white)](https://about.gitlab.com/)
[![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-EF7B4D?logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![Gateway API](https://img.shields.io/badge/Networking-Gateway_API_(kgateway)-555555?logo=kubernetes&logoColor=white)](https://gateway-api.sigs.k8s.io/)
[![MetalLB](https://img.shields.io/badge/LoadBalancer-MetalLB-0052CC?logo=kubernetes&logoColor=white)](https://metallb.universe.tf/)
[![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/)
[![cAdvisor](https://img.shields.io/badge/Container_Metrics-cAdvisor-326CE5?logo=kubernetes&logoColor=white)](https://github.com/google/cadvisor)
[![Grafana](https://img.shields.io/badge/Observability-Grafana-F46800?logo=grafana&logoColor=white)](https://grafana.com/)
[![AWS S3](https://img.shields.io/badge/Storage-AWS_S3-FF9900?logo=amazonaws&logoColor=white)](https://aws.amazon.com/s3/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A complete, production-ready **Cloud-Native & DevOps** portfolio project demonstrating a full lifecycle microservices deployment.Built on a self-hosted **Kubernetes (kubeadm) cluster** using modern **GitOps (ArgoCD)** principles, **Kubernetes Gateway API**, automated **GitLab CI/CD** pipelines and full-stack **Prometheus/Grafana** observability.

**Take a look at my first (Application) related repository:**  
**GitHub:** https://github.com/shotorouu/devops-qr-code/  
**GitLab:** https://gitlab.com/shotorouu/devops-qr-code/  

---

### 🌐 Modern Ingress & Networking

1. Gateway API Integration: Shifted from legacy ingress-nginx to kgateway (HTTPRoute / Gateway resources) for role-oriented traffic management.
2. Bare-Metal Load Balancing: Deployed MetalLB in Layer-2 mode to assign real external IPs without cloud provider lock-in.

### 🚀 Production GitOps & Delivery

1. Multi-Repo Architecture: Isolated application source code (devops-qr-code) from deployment state manifests (devops-qr-code-argocd).
2. ArgoCD Automated Sync: Enforced zero-drift reconciliation via selfHeal: true, prune: true, and ServerSideApply=true.
3. Declarative Overlays: Utilized Kustomize for dynamic image tagging and environment configs without Helm overhead.

### 🔒 Security & Container Resilience

1. Non-Root Runtime: Enforced runAsNonRoot: true (UID 10001) across minimal multi-stage builds (python:3.11-slim / node:20-alpine).
2. High Availability: Implemented topologySpreadConstraints, zero-downtime rolling updates (maxSurge: 25%, maxUnavailable: 0), and full 3-stage health probes.

### 📊 Full-Stack Observability

1. Application Telemetry: Integrated native /metrics endpoints via prometheus_fastapi_instrumentator and Next.js metric handlers.
2. Dynamic Discovery: Auto-scraped by Prometheus via Pod annotations, paired with cAdvisor container metrics under minimal RBAC permissions.

---

# 🗽 Architecture & Workflow

```mermaid
flowchart TB

%% --- Themes & Styling ---

classDef ciFill fill:#1e293b,stroke:#fca121,stroke-width:2px,color:#fff

classDef gitopsFill fill:#1e293b,stroke:#ef7b4d,stroke-width:2px,color:#fff

classDef k8sNet fill:#0f172a,stroke:#326ce5,stroke-width:2px,color:#fff

classDef k8sApp fill:#1e293b,stroke:#38bdf8,stroke-width:2px,color:#fff

classDef k8sObs fill:#1e1b4b,stroke:#e6522c,stroke-width:2px,color:#fff

classDef extFill fill:#26180a,stroke:#ff9900,stroke-width:2px,color:#fff



%% --- Stage 1: CI Pipeline ---

subgraph CI ["🚀 1. Continuous Integration (GitLab CI Pipeline)"]

direction TB

DEV["👨‍💻 Developer"] -->|1. Git Commit / Push| REPO_APP["📁 App Repository<br/>devops-qr-code"]

REPO_APP -->|2. Webhook Trigger| RUNNER["⚙️ GitLab CI Runner"]


subgraph PIPELINE ["CI Pipeline Stages (.gitlab-ci.yml)"]

direction TB

STAGE_TEST["🧪 Test Stage<br/>pytest / test_main.py"] --> STAGE_BUILD["🔒 Build Stage<br/>Docker Multi-Stage Build"]

STAGE_BUILD --> STAGE_SCAN["🛡️ Security Scan<br/>Container Vulnerability Check"]

STAGE_SCAN --> STAGE_PUSH["🏷️ Tag & Push<br/>Update SHA / Image Tag"]

end

RUNNER --> PIPELINE

end



%% --- Stage 2: Artifacts & GitOps Engine ---

subgraph GITOPS ["🔄 2. GitOps Delivery & State Management"]

direction TB

REGISTRY[("📦 Container Registry<br/>GitlabCi registry")]

REPO_MANIFESTS["📁 GitOps Manifest Repository<br/>devops-qr-code-argocd<br/>(Source of Truth)"]

ARGO["🐙 ArgoCD Controller<br/>(Self-Heal: True | Prune: True)"]



PIPELINE -->|3a. Push Container Image| REGISTRY

PIPELINE -->|3b. Update Kustomize Tag| REPO_MANIFESTS

REPO_MANIFESTS -->|4. Poll / Sync State| ARGO

end



%% --- Stage 3: Bare-Metal Kubernetes Cluster ---

subgraph K8S ["☸️ 3. Bare-Metal Kubernetes Cluster (v1.31)"]

direction TB



%% Ingress & Networking Subgraph

subgraph NETWORKING ["🌐 Ingress & Traffic Routing Layer"]

direction TB

METALLB["⚖️ MetalLB (L2 Mode)<br/>IPAddressPool: 192.168.1.200-250"]

KGATEWAY["🚪 Gateway API Controller (kgateway)<br/>GatewayClass / Gateway Resource"]

HTTPROUTE["🔀 HTTPRoute (qrcode namespace)<br/>Matches: Host & Path Patterns"]



METALLB -->|Assigns External IP| KGATEWAY

KGATEWAY -->|Binds Route Rules| HTTPROUTE

end



%% Workloads Subgraph

subgraph WORKLOADS ["🚀 Microservices Layer (qrcode namespace)"]

direction TB


subgraph FRONTEND_POD ["Frontend Pod (1 Replica)"]

direction TB

FE_SVC["🔌 frontend-service<br/>ClusterIP: Port 3000"] --> FE["🎨Frontend (Next.js)<br/>node:20-alpine<br/>Security: runAsNonRoot (UID 10001)<br/>Probes: Startup / Readiness / Liveness"]

end



subgraph BACKEND_POD ["Backend Pod (2 Replicas)"]

direction TB

BE_SVC["🔌 fastapi-service<br/>ClusterIP: Port 8000"] --> BE["⚡Backend (FastAPI)<br/>python:3.11-slim<br/>Security: runAsNonRoot<br/>TopologySpreadConstraints (HA)<br/>Probes: Startup / Readiness / Liveness"]

end

end



%% Observability Subgraph

subgraph OBSERVABILITY ["📊 Full-Stack Observability Layer"]

direction TB


GRAFANA["📈 Grafana Dashboards<br/>NodePort: 30031 (Admin)"] -->|PromQL Queries| PROM["🔥 Prometheus Server<br/>Scrape Rules / RBAC / ConfigMap"]

PROM -->|Scrape Container Runtime| CADVISOR["🐳 cAdvisor<br/>Container Runtime Metrics"]

end



%% Cluster Internal Wiring

HTTPROUTE -->|Path: / | FE_SVC

HTTPROUTE -->|Path: /api | BE_SVC



REGISTRY -.->|Pull Authenticated Image| FE

REGISTRY -.->|Pull Authenticated Image| BE



PROM -.->|Scrape /metrics| BE

PROM -.->|Scrape /metrics| FE

end



%% --- External Infrastructure ---

subgraph AWS ["☁️ External Cloud Services"]

S3[("🪣 AWS S3 Bucket<br/>QR Code Asset Storage<br/>(boto3 SDK Integration)")]

end



%% --- Key System Cross-Layer Connections ---

ARGO -->|5. Declarative Kustomize Sync| K8S

BE -->|Uploads Generated Images| S3



%% --- Applying CSS Classes ---

class DEV,REPO_APP,RUNNER,STAGE_TEST,STAGE_BUILD,STAGE_SCAN,STAGE_PUSH ciFill

class REGISTRY,REPO_MANIFESTS,ARGO gitopsFill

class METALLB,KGATEWAY,HTTPROUTE k8sNet

class FE,FE_SVC,BE,BE_SVC k8sApp

class PROM,CADVISOR,GRAFANA k8sObs

class S3 extFill
```


# 🐙ArgoCD (GitOps) repository (https://github.com/shotorouu/devops-qr-code-argocd)

```
devops-qr-code-argocd/
├── application-infra.yml      # ArgoCD Application for Infrastructure (Gateway API & MetalLB)
├── application.yml            # ArgoCD Application for Microservices & Monitoring
├── dev/                       # Workload & Monitoring Manifests (Kustomize Overlay)
│   ├── fastapi_deployment.yml # FastAPI Deployment (Probes, TopologySpread, Non-Root)
│   ├── fastapi_service.yml    # ClusterIP Service for API
│   ├── frontend_deployment.yml# Next.js Deployment
│   ├── frontend_service.yml   # ClusterIP Service for Frontend
│   ├── httproute.yml          # Gateway API HTTPRoute definition
│   ├── kustomization.yml      # Kustomize manifest list & dynamic image tags
│   └── monitoring/            # Observability Stack
│       ├── grafana-deployment.yml
│       ├── grafana-service.yml
│       ├── prometheus-configmap.yml
│       ├── prometheus-deployment.yml
│       ├── prometheus-rbac.yml
│       └── prometheus-service.yml
├── infra/                     # Infrastructure Manifests
│   ├── gateway.yml            # Gateway API Class definition
│   ├── kustomization.yml      # Kustomize infra configuration
│   └── metallb.yml            # MetalLB IPAddressPool & L2Advertisement
└── README.md
```

# 🐍Application repository (https://github.com/shotorouu/devops-qr-code)

```
devops-qr-code/
├── api/
│   ├── Dockerfile             # Multi-stage non-root Python build
│   ├── main.py                # FastAPI app (Prometheus instrumented + AWS S3 SDK)
│   ├── requirements.txt
│   └── test_main.py           # Unit testing suite
├── front-end-nextjs/
│   ├── Dockerfile             # Multi-stage non-root Next.js build
│   ├── jsconfig.json
│   ├── next.config.js
│   ├── package.json
│   ├── package-lock.json
│   ├── postcss.config.js
│   ├── public/
│   ├── src/
│   │   └── app/
│   │       ├── metrics/       # Custom Next.js Prometheus metrics exporter route
│   │       ├── layout.js
│   │       └── page.js
│   └── tailwind.config.js
├── .gitlab-ci.yml             # Build, image push, and GitOps tag update pipeline
├── LICENSE
└── README.md
```

# 🚀 Complete Local Deployment & Setup Guide
## Prerequisites🎯:
1. A running Kubernetes cluster (v1.28+) (e.g., kubeadm, minikube, or k3s).
2. kubectl CLI installed and connected to your cluster.
3. ArgoCD installed in the argocd namespace.
4. AWS account with an active S3 bucket and IAM account with credentials.

### Step 1✅: Create AWS Credentials Secret. The application requires AWS credentials to save generated QR codes into an S3 bucket. Create the secret manually prior to syncing workloads:
```bash
# Option A: Create secret from an existing .env file
# Create a file named api/.env with:
# AWS_ACCESS_KEY="your-access-key"
# AWS_SECRET_KEY="your-secret-key"
# AWS_BUCKET_NAME="your-bucket-name"
kubectl create secret generic aws-creds --from-env-file=api/.env -n qrcode

# Option B: Create secret directly via command line
kubectl create secret generic aws-creds \
  --from-literal=AWS_ACCESS_KEY="YOUR_AWS_ACCESS_KEY" \
  --from-literal=AWS_SECRET_KEY="YOUR_AWS_SECRET_KEY" \
  --from-literal=AWS_BUCKET_NAME="YOUR_S3_BUCKET_NAME" \
  -n qrcode
```

### Step 2✅: Configure MetalLB IP RangeEdit infra/metallb.yml to reflect an available IP address range on your local network:
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: local-ip-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.1.200-192.168.1.250  # Update this range for your subnet
```

### Step 3✅: Deploy ArgoCD GitOps Applications
```bash
# 1. Deploy Infrastructure Application (MetalLB & Gateway API)
kubectl apply -f application-infra.yml

#2. Deploy Application Workloads & Observability Stack
kubectl apply -f application.yml
```

### Step 4✅: Verify Deployment & Networking
```bash
# Check ArgoCD sync status
kubectl get applications -n argocd

# Check running workloads
kubectl get pods -n qrcode

# Retrieve Gateway API external IP assigned by MetalLB
kubectl get gateway -n gateway-qrcode
kubectl get httproute -n qrcode
```

### Step 5: Accessing Services

| Component | Access Method | Endpoint / Address |
| :--- | :--- | :--- |
| **Frontend UI** | MetalLB LoadBalancer | `http://<METALLB_GATEWAY_IP>/` |
| **FastAPI Backend** | MetalLB Route / API | `http://<METALLB_GATEWAY_IP>/api/` |
| **Prometheus UI** | NodePort / Port-Forward | `http://<NODE_IP>:30090` |
| **Grafana UI** | NodePort | `http://<NODE_IP>:30031` |

```yaml
metadata:
  annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8000"
  prometheus.io/path: "/metrics"
```
To configure Grafana dashboards:
1. Log into Grafana at http://<NODE_IP>:30031.
2. Navigate to Connections > Data Sources and add Prometheus (http://prometheus-service.qrcode.svc.cluster.local:9090).
3. Import community dashboards for Kubernetes container metrics (e.g. ID 15760 for cAdvisor / Node exporter metrics).

### This project is licensed under the MIT License⚖️.
