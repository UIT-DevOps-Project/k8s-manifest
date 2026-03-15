# ArgoCD Installation & Deployment Guide

## Prerequisites

- Kubernetes cluster (v1.20 or later)
- `kubectl` configured to access your cluster
- `git` installed on your machine
- Docker/Container runtime

## Installation Steps

### 1. Create ArgoCD & staging & production Namespace

```bash
kubectl create namespace argocd
kubectl create namespace staging
kubectl create namespace production

```

### 2. Install ArgoCD using kubectl

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


### 3. Verify ArgoCD Installation

```bash
# Check if all pods are running
kubectl get pods -n argocd

# Expected output: server, repo-server, controller, dex-server, redis pods should be running
```

## Access ArgoCD UI



###  Port Forwarding (Recommended for testing)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```

Then access: `https://localhost:8080`



## Get Initial Password

```bash
# Get the auto-generated initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Username: admin
# Password: (output from above command)
```

## Configure Repository


### 1. Deploy Root Application

Apply the root application manifest:

```bash
kubectl apply -f argocd/root-app.yaml
```

Verify deployment:
```bash
kubectl get application -n argocd
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
