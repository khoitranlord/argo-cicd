# PR Review Workflow for Infrastructure Deployments

## Strategy Overview

Instead of direct pushes to main, use Pull Requests for infrastructure changes:

```
Developer Code → CI Build → PR to GitOps → Review → Merge → ArgoCD Deploy
```

## 1. Modified CI Pipeline (Creates PR instead of direct push)

```yaml
# In your app repository: .github/workflows/ci.yml
name: Build and Create Deployment PR

on:
  push:
    branches: [main]

jobs:
  build-and-create-pr:
    runs-on: ubuntu-latest
    steps:
      - name: Build and push image
        run: |
          docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .
          docker push ghcr.io/${{ github.repository }}:${{ github.sha }}

      - name: Create deployment PR
        uses: peter-evans/create-pull-request@v5
        with:
          token: ${{ secrets.GITOPS_TOKEN }}
          repository: khoitranlord/argo-cicd
          branch: deploy/${{ github.sha }}
          title: "🚀 Deploy ${{ github.repository }}:${{ github.sha }}"
          body: |
            ## Deployment Request
            
            **Application**: ${{ github.repository }}
            **Image**: `ghcr.io/${{ github.repository }}:${{ github.sha }}`
            **Commit**: ${{ github.sha }}
            
            ### Changes
            - Updated container image to latest build
            - Commit: ${{ github.event.head_commit.message }}
            
            ### Testing
            - [ ] Code tests passed ✅
            - [ ] Image build successful ✅ 
            - [ ] Security scan completed
            - [ ] Performance review pending
            
            **⚠️ This PR requires approval before deployment to production**
          commit-message: |
            Deploy ${{ github.repository }}:${{ github.sha }}
            
            Image: ghcr.io/${{ github.repository }}:${{ github.sha }}
          files: |
            k8s/deployment.yaml
```

## 2. Environment-Based Approval Strategy

### Development Environment (Auto-deploy)
```yaml
# apps/dev/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-dev
spec:
  source:
    repoURL: https://github.com/khoitranlord/argo-cicd.git
    targetRevision: main  # Auto-deploy from main
    path: apps/dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Production Environment (Manual approval)
```yaml
# apps/prod/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-prod
spec:
  source:
    repoURL: https://github.com/khoitranlord/argo-cicd.git
    targetRevision: prod  # Deploy from prod branch only
    path: apps/prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 3. GitHub Branch Protection Rules

Configure in GitHub Settings → Branches:

```yaml
# .github/branch-protection.yml (using Probot)
protection_rules:
  main:
    required_status_checks:
      strict: true
      contexts:
        - "ci/tests"
        - "security/scan"
    enforce_admins: false
    required_pull_request_reviews:
      required_approving_review_count: 2
      dismiss_stale_reviews: true
      restrict_dismissals: true
      required_review_from_code_owners: true
    restrictions:
      users: []
      teams: ["platform-team", "sre-team"]
```

## 4. CODEOWNERS File

```bash
# .github/CODEOWNERS
# Global rules
* @platform-team

# Production deployments require SRE approval
apps/prod/ @sre-team @platform-lead
k8s/prod/ @sre-team @platform-lead

# Security configs require security team
security/ @security-team
rbac/ @security-team

# ArgoCD configurations
applications/ @platform-team
projects/ @platform-team
```

## 5. Multi-Environment Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│     DEV     │    │   STAGING   │    │    PROD     │
│             │    │             │    │             │
│ Auto-deploy │───▶│ Auto-deploy │───▶│ PR Required │
│ from main   │    │ from main   │    │ Manual      │
│             │    │ + tests     │    │ Approval    │
└─────────────┘    └─────────────┘    └─────────────┘
```

### Promotion Pipeline
```yaml
# .github/workflows/promote.yml
name: Promote to Production

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
        - staging
        - prod

jobs:
  promote:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment }}  # Requires approval
    steps:
      - name: Promote to ${{ github.event.inputs.environment }}
        run: |
          # Copy configs from staging to production
          cp -r apps/staging/* apps/prod/
          git commit -am "Promote to ${{ github.event.inputs.environment }}"
```

## 6. ArgoCD Sync Policies by Environment

### Development (Immediate)
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

### Production (Manual)
```yaml
syncPolicy:
  # No automated sync - manual approval required
  syncOptions:
  - CreateNamespace=true
```

## 7. Review Process Flow

1. **Developer pushes code** → App repository
2. **CI builds image** → Creates PR in GitOps repo
3. **Platform team reviews** → Checks deployment safety
4. **SRE approves** → For production changes
5. **PR merges** → ArgoCD deploys automatically
6. **Monitor deployment** → Rollback if issues

## 8. PR Template for Deployments

```markdown
<!-- .github/pull_request_template.md -->
## Deployment Checklist

### Pre-deployment
- [ ] Image security scan passed
- [ ] Performance tests completed
- [ ] Database migrations (if any) reviewed
- [ ] Rollback plan prepared

### Deployment Details
**Application**: 
**Environment**: 
**Image Tag**: 
**Expected Impact**: 

### Post-deployment
- [ ] Health checks verified
- [ ] Monitoring alerts configured
- [ ] Performance metrics baseline established

### Approvals Required
- [ ] Platform Team (@platform-team)
- [ ] SRE Team (@sre-team) - for production only
- [ ] Security Team (@security-team) - for security changes

**Risk Level**: [ ] Low [ ] Medium [ ] High
```

This ensures all infrastructure changes go through proper review before reaching production!