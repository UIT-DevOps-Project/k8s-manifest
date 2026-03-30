# Hướng dẫn Apply Ingress Nginx trên KinD

## Yêu cầu tiên quyết

- KinD đã được cài đặt (`kind version`)
- Docker đã được cài đặt và chạy
- kubectl đã được cài đặt

## Bước 1: Tạo KinD Cluster

```bash
kind create cluster --config kind/kind-config.yaml
```

Kiểm tra cluster đã tạo thành công:

```bash
kubectl cluster-info
kubectl get nodes
```

## Bước 2: Apply Ingress Nginx

```bash
kubectl apply -f kind/ingress-nginx-kind.yaml
```

Kiểm tra Ingress Nginx đã chạy:

```bash
kubectl get pods -n ingress-nginx
kubectl get services -n ingress-nginx
```

Đợi cho đến khi tất cả pods có status `Running`:

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

## Bước 3: Xác nhận Ingress Controller

```bash
# Xem ingress controller deployment
kubectl get deployment -n ingress-nginx

# Xem ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

## Dọn dẹp

Xóa Ingress Nginx:

```bash
kubectl delete -f kind/ingress-nginx-kind.yaml
```

Xóa KinD cluster:

```bash
kind delete cluster
```

## Xử lý Sự cố

### Pods không khởi động

```bash
# Xem chi tiết
kubectl describe pod <pod-name> -n ingress-nginx

# Xem logs
kubectl logs <pod-name> -n ingress-nginx
```

### ImagePullBackOff

- Kiểm tra internet connection
- Kiểm tra Docker daemon đang chạy

### Port conflict

Nếu gặp lỗi port bận, chỉnh sửa `kind-config.yaml` để dùng port khác.
