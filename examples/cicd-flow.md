# Complete CI/CD Flow with ArgoCD

## Step-by-Step Process

### 1. Developer Workflow
```bash
# Developer makes changes
git add .
git commit -m "Update app to v2.0"
git push origin main
```

### 2. CI Pipeline (Continuous Integration)
```yaml
# .github/workflows/ci.yml triggers automatically
name: CI/CD Pipeline
on:
  push:
    branches: [main]

jobs:
  # STEP 1: Test the code
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
      - run: npm run lint

  # STEP 2: Build Docker image
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build and push
        run: |
          docker build -t ghcr.io/khoitranlord/my-app:${{ github.sha }} .
          docker push ghcr.io/khoitranlord/my-app:${{ github.sha }}

  # STEP 3: Update GitOps repo
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Update deployment manifest
        run: |
          # Clone GitOps repo
          git clone https://github.com/khoitranlord/argo-cicd.git
          cd argo-cicd
          
          # Update image tag
          sed -i "s|image: ghcr.io/khoitranlord/my-app:.*|image: ghcr.io/khoitranlord/my-app:${{ github.sha }}|" k8s/deployment.yaml
          
          # Commit and push to GitOps repo
          git add .
          git commit -m "Deploy ${{ github.sha }}"
          git push
```

### 3. ArgoCD Detects Change (Continuous Deployment)
```bash
# ArgoCD polls GitOps repo every 3 minutes
# Detects new commit in argo-cicd repo
# Compares desired state (Git) vs actual state (Kubernetes)
# Automatically syncs the changes
```

### 4. Kubernetes Deployment
```bash
# ArgoCD applies the new manifest
kubectl apply -f k8s/deployment.yaml

# Kubernetes performs rolling update
kubectl rollout status deployment/my-app
# → deployment "my-app" successfully rolled out
```

## Visual Flow

```
┌─────────────────┐
│   Developer     │
│   git push      │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐    ┌─────────────────┐
│   GitHub        │    │   CI Pipeline   │
│   Repository    │───▶│   (Actions)     │
│   (App Code)    │    │   - Test        │
└─────────────────┘    │   - Build       │
                       │   - Push Image  │
                       └─────────┬───────┘
                                 │
                                 ▼
┌─────────────────┐    ┌─────────────────┐
│   Container     │◀───│   Update        │
│   Registry      │    │   GitOps Repo   │
│   (Docker Hub,  │    │   (New Image)   │
│    GHCR, etc.)  │    └─────────┬───────┘
└─────────────────┘              │
                                 ▼
┌─────────────────┐    ┌─────────────────┐
│   ArgoCD        │───▶│   Kubernetes    │
│   (Watches Git) │    │   Cluster       │
│   - Detects     │    │   - Rolling     │
│   - Syncs       │    │     Update      │
│   - Deploys     │    │   - New Pods    │
└─────────────────┘    └─────────────────┘
```

## Benefits of This Approach

### Separation of Concerns
- **App Repo**: Source code, tests, build process
- **GitOps Repo**: Kubernetes manifests, configuration
- **ArgoCD**: Deployment automation and sync

### GitOps Principles
1. **Declarative**: Everything in Git
2. **Versioned**: All changes tracked
3. **Immutable**: Git is source of truth
4. **Observable**: ArgoCD UI shows status

### Security & Compliance
- **No kubectl access**: Developers can't access cluster directly
- **Audit trail**: All changes in Git history
- **Rollback**: Easy to revert using Git
- **Approval**: Can add PR reviews for production

## Real Example: Node.js App

### App Repository (my-node-app)
```
my-node-app/
├── src/
│   └── app.js
├── package.json
├── Dockerfile
├── .github/workflows/
│   └── ci.yml
└── tests/
    └── app.test.js
```

### GitOps Repository (argo-cicd) 
```
argo-cicd/
├── applications/
│   └── node-app.yaml      # ArgoCD Application
├── k8s/
│   ├── deployment.yaml    # Updated by CI
│   ├── service.yaml
│   └── configmap.yaml
└── projects/
    └── default.yaml
```

### Deployment Process
1. **Code Change**: Developer updates `src/app.js`
2. **CI Triggers**: Tests pass, Docker image built
3. **Image Tag**: `ghcr.io/khoitranlord/my-app:abc123`
4. **GitOps Update**: CI updates `k8s/deployment.yaml`
5. **ArgoCD Sync**: Detects change, deploys to cluster
6. **Rolling Update**: New pods with updated image

This creates a fully automated, secure, and auditable deployment pipeline!