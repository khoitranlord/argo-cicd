# Argo CD Setup

This directory contains the Argo CD configuration files for deploying applications to Kubernetes.

## Structure

```
.
├── application.yaml     # Argo CD Application manifest
├── project.yaml        # Argo CD Project configuration
├── kustomization.yaml  # Kustomize configuration
└── k8s/               # Kubernetes manifests
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

## Setup Instructions

1. **Apply the Project** (optional):
   ```bash
   kubectl apply -f project.yaml
   ```

2. **Apply the Application**:
   ```bash
   kubectl apply -f application.yaml
   ```

## Configuration

### Before deploying:

1. Update `application.yaml`:
   - Replace `https://github.com/your-username/your-repo.git` with your actual Git repository URL
   - Adjust the `path` if your manifests are in a different directory
   - Modify the `namespace` and `server` as needed

2. Update `project.yaml`:
   - Replace the repository URL with your actual repository
   - Adjust permissions and roles as needed

3. Update `k8s/` manifests:
   - Replace `nginx:latest` with your actual application image
   - Modify resource limits and requests
   - Update service and ingress configurations

### Accessing Argo CD UI

If you haven't already, access the Argo CD UI:

```bash
# Get the initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward to access the UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open https://localhost:8080 and login with username `admin`.

## Sync Policies

The application is configured with automatic sync enabled:
- `prune: true` - Remove resources that are no longer in Git
- `selfHeal: true` - Automatically fix drift between Git and cluster state
- `CreateNamespace=true` - Automatically create the target namespace if it doesn't exist

## Customization

Use Kustomize to customize deployments for different environments by creating overlay directories and modifying the `kustomization.yaml` file.