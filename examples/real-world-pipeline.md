# Real-World CI/CD Pipeline with ArgoCD

## Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  App Repository │    │ GitOps Repository│    │   Kubernetes    │
│                 │    │  (argo-cicd)    │    │    Cluster      │
│  - Source Code  │    │  - K8s Manifests│    │  - ArgoCD       │
│  - Dockerfile   │───▶│  - Applications │───▶│  - Applications │
│  - CI Pipeline  │    │  - Projects     │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Step-by-Step Integration

### 1. Application Repository Setup

Create a new repository for your application:

```bash
# Create new repo
gh repo create my-node-app --public --clone
cd my-node-app

# Add application files (from examples above)
# - package.json
# - server.js  
# - Dockerfile
# - .github/workflows/ci.yml
```

### 2. CI Pipeline Triggers

The pipeline automatically:
1. **Tests** your code on every PR
2. **Builds** Docker image on main branch
3. **Updates** GitOps repo with new image tag
4. **ArgoCD** detects change and deploys

### 3. ArgoCD Application Configuration

```yaml
# application-node-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: node-app
  namespace: argocd
spec:
  project: simple-test-project
  source:
    repoURL: https://github.com/khoitranlord/argo-cicd.git
    targetRevision: HEAD
    path: apps/node-app  # Specific app folder
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### 4. Directory Structure (GitOps Repo)

```
argo-cicd/
├── applications/          # ArgoCD Applications
│   ├── nginx-app.yaml
│   └── node-app.yaml
├── apps/                  # App-specific manifests
│   ├── nginx/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── node-app/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── configmap.yaml
├── projects/              # ArgoCD Projects
│   └── default-project.yaml
└── system/               # System components
    └── argocd/
```

### 5. Image Update Strategies

**Option A: Direct Update (Current)**
- CI updates K8s manifests directly
- Simple but couples build and deploy

**Option B: ArgoCD Image Updater**
- Watches container registries
- Updates images automatically
- Decouples build from deploy

**Option C: Helm with Values**
- Use Helm charts
- CI updates values.yaml only
- More flexible configuration

### 6. Environment Promotion

```
┌─────────┐    ┌─────────┐    ┌─────────┐
│   DEV   │───▶│ STAGING │───▶│  PROD   │
└─────────┘    └─────────┘    └─────────┘
     │              │              │
     ▼              ▼              ▼
apps/dev/      apps/staging/   apps/prod/
```

### 7. Security & Best Practices

- **Secrets**: Use Sealed Secrets or External Secrets Operator
- **RBAC**: Limit ArgoCD permissions per environment
- **Policies**: Use OPA Gatekeeper for compliance
- **Scanning**: Integrate Trivy for image vulnerability scanning

### 8. Monitoring & Observability

- **ArgoCD Notifications**: Slack/Teams alerts
- **Prometheus**: Collect ArgoCD metrics
- **Grafana**: Dashboards for deployment status
- **Logging**: Centralized application logs