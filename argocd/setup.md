# ArgoCD Installation & Deployment Guide

## Prerequisites

- Kubernetes cluster (v1.20 or later)
- `kubectl` configured to access your cluster
- `git` installed on your machine
- Docker/Container runtime

## Installation Steps

### 1. Create ArgoCD Namespace

```bash
kubectl create namespace argocd
```

### 2. Install ArgoCD using kubectl

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Hoặc nếu muốn sử dụng version cụ thể:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.11.0/manifests/install.yaml
```

### 3. Verify ArgoCD Installation

```bash
# Check if all pods are running
kubectl get pods -n argocd

# Expected output: server, repo-server, controller, dex-server, redis pods should be running
```

## Access ArgoCD UI

### Option 1: Expose via Service (NodePort)

```bash
# Change service type to NodePort
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

# Get the NodePort
kubectl get svc -n argocd argocd-server
```

### Option 2: Port Forwarding (Recommended for testing)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```

Then access: `https://localhost:8080`

### Option 3: Ingress Setup

```bash
# Patch ArgoCD server to work with HTTPS Ingress
kubectl patch svc argocd-server -n argocd -p '{"spec": {"sessionAffinity": "ClientIP"}}'

# Create ingress (see example in kind/ingress-nginx-kind.yaml)
```

## Get Initial Password

```bash
# Get the auto-generated initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Username: admin
# Password: (output from above command)
```

## Configure Repository

### 1. Add Repository to ArgoCD

Via CLI:
```bash
argocd login <ARGOCD_SERVER>
argocd repo add https://github.com/UIT-DevOps-Project/k8s-manifest \
  --username <your-github-username> \
  --password <your-github-token> \
  --insecure-skip-server-verification
```

Via UI:
1. Go to Settings → Repositories
2. Click "Connect Repo"
3. Select Connection method: HTTPS
4. Fill in repository details:
   - Repository URL: `https://github.com/UIT-DevOps-Project/k8s-manifest`
   - Username: GitHub username
   - Password: GitHub personal access token
5. Click Connect

### 2. Create GitHub Personal Access Token (if needed)

1. Go to GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token with `repo` scope
3. Copy the token (you won't see it again)

## Deploy Applications

### 1. Deploy Root Application

Apply the root application manifest:

```bash
kubectl apply -f argocd/root-app.yaml
```

Verify deployment:
```bash
kubectl get application -n argocd
```

### 2. Verify Child Applications

Watch the deployment progress:

```bash
# Check app status
argocd app list

# Get detailed status
argocd app get root-app -n argocd

# Watch sync progress
kubectl get application -n argocd -w
```

### 3. Check Application Status via UI

1. Open ArgoCD UI
2. Navigate to Applications
3. Click on `root-app` to see:
   - Sync status (Synced/Out of sync)
   - Health status
   - Resources managed by the app
   - Update log

## Troubleshooting

### Check ArgoCD Server Logs

```bash
kubectl logs -n argocd svc/argocd-server
```

### Check ArgoCD App Controller Logs

```bash
kubectl logs -n argocd svc/argocd-application-controller
```

### Check Repository Connection

```bash
argocd repo list
argocd repo get https://github.com/UIT-DevOps-Project/k8s-manifest
```

### Debug Application Sync Issues

```bash
# Get application details
kubectl describe application root-app -n argocd

# Check if resources are created
kubectl get all -n <namespace>
```

## Useful Commands

```bash
# Login to ArgoCD
argocd login <ARGOCD_SERVER> --insecure

# Change admin password
argocd account update-password --account admin --new-password <new_password>

# List all applications
argocd app list

# Sync specific application manually
argocd app sync root-app

# Delete application
argocd app delete root-app

# Get application manifest
argocd app get root-app --show-params

# Force refresh from repository
argocd app refresh root-app
```

## Project Structure

```
argocd/
├── root-app.yaml              # Root application - entry point
├── apps/
│   ├── backend.yaml           # Backend application definition
│   └── frontend.yaml          # Frontend application definition
└── setup.md                   # This file
```

## Application Management

### Root App (root-app.yaml)
- Serves as the entry point for ArgoCD
- Points to `argocd/apps` directory
- Uses automated sync policy:
  - `prune: true` - deletes resources removed from Git
  - `selfHeal: true` - automatically syncs if cluster differs from Git

### Child Applications (backend.yaml, frontend.yaml)
- Managed by root-app
- Each defines its own source, destination, and sync policy
- Changes to these files automatically trigger syncs

## Next Steps

1. Ensure repository is properly configured in ArgoCD
2. Verify all applications are syncing successfully
3. Set up ingress for application access
4. Configure monitoring and notifications (optional)
5. Implement GitOps workflow for your team

## References

- [ArgoCD Official Documentation](https://argo-cd.readthedocs.io/)
- [ArgoCD GitHub Repository](https://github.com/argoproj/argo-cd)
- [Application CRD Documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/)
