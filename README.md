# k8s-manifest

Kubernetes manifests cho DevOps project sử dụng KinD (Kubernetes in Docker).

## Cấu trúc thư mục

```
k8s-manifest/
├── kind/          # Cấu hình KinD cluster và Ingress Nginx
├── argocd/        # Cài đặt và cấu hình ArgoCD
├── apps/          # ArgoCD Application manifests
└── monitoring/    # Grafana, Prometheus, FluentBit, OpenSearch
```

## Hướng dẫn

- [Cài đặt KinD cluster và Ingress Nginx](kind/setup.md)
- [Cài đặt ArgoCD](argocd/setup.md)