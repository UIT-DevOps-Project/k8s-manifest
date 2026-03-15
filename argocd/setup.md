# Hướng dẫn Cài đặt ArgoCD trên KinD

## Yêu cầu tiên quyết

- KinD cluster đã được tạo và chạy (`kubectl cluster-info`)
- Ingress Nginx đã được cài đặt (xem `kind/setup.md`)
- kubectl đã được cài đặt

## Bước 1: Tạo Namespace và Cài đặt ArgoCD

Tạo namespace `argocd`:

```bash
kubectl apply -f argocd/argocd-namespace.yaml
```

Cài đặt ArgoCD từ manifest chính thức:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Kiểm tra các pods đã chạy:

```bash
kubectl get pods -n argocd
```

Đợi cho đến khi tất cả pods có status `Running`:

```bash
kubectl wait --namespace argocd \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/name=argocd-server \
  --timeout=120s
```

## Bước 2: Expose ArgoCD qua Ingress

Ingress sử dụng chế độ `ssl-passthrough`, nghĩa là Ingress Nginx chuyển tiếp TLS trực tiếp tới ArgoCD server mà không terminate — không cần thay đổi cấu hình ArgoCD server.

Apply Ingress cho ArgoCD UI:

```bash
kubectl apply -f argocd/argocd-ingress.yaml
```

Thêm entry vào `/etc/hosts` để truy cập qua hostname:

```bash
echo "127.0.0.1 argocd.local" | sudo tee -a /etc/hosts
```

Kiểm tra Ingress:

```bash
kubectl get ingress -n argocd
```

## Bước 3: Lấy Mật khẩu Admin

Lấy mật khẩu mặc định của tài khoản `admin`:

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

## Bước 4: Truy cập ArgoCD UI

Mở trình duyệt và truy cập: **http://argocd.local**

- **Username:** `admin`
- **Password:** (lấy từ bước 3)

Hoặc có thể port-forward để truy cập trực tiếp mà không cần Ingress:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Sau đó truy cập: **https://localhost:8080**

## Bước 5: Đổi Mật khẩu Admin (Khuyến nghị)

Sau khi đăng nhập lần đầu, nên đổi mật khẩu admin:

```bash
# Cài đặt ArgoCD CLI (nếu chưa có)
# Linux amd64:
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

# macOS (dùng Homebrew):
# brew install argocd

# Đăng nhập vào ArgoCD (nhập mật khẩu theo prompt để tránh lộ trong shell history)
argocd login argocd.local --username admin

# Đổi mật khẩu
argocd account update-password
```

## Bước 6: Kết nối Repository Git

Để ArgoCD theo dõi repository k8s-manifest:

```bash
argocd repo add https://github.com/UIT-DevOps-Project/k8s-manifest \
  --username <git-username> \
  --password <git-token>
# Lưu ý: có thể dùng SSH key thay vì username/password để an toàn hơn:
# argocd repo add git@github.com:UIT-DevOps-Project/k8s-manifest --ssh-private-key-path ~/.ssh/id_rsa
```

## Cấu trúc Thư mục `argocd/`

```
argocd/
├── argocd-namespace.yaml   # Namespace definition cho ArgoCD
├── argocd-ingress.yaml     # Ingress để expose ArgoCD UI qua nginx (host: argocd.local)
└── setup.md                # Tài liệu này
```

## Dọn dẹp

Xóa ArgoCD:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete -f argocd/argocd-ingress.yaml
kubectl delete -f argocd/argocd-namespace.yaml
```

## Xử lý Sự cố

### Pods không khởi động

```bash
# Xem chi tiết pod
kubectl describe pod <pod-name> -n argocd

# Xem logs
kubectl logs <pod-name> -n argocd
```

### Không truy cập được UI qua Ingress

```bash
# Kiểm tra Ingress controller đang chạy
kubectl get pods -n ingress-nginx

# Kiểm tra Ingress resource
kubectl describe ingress argocd-server-ingress -n argocd

# Kiểm tra /etc/hosts có entry argocd.local chưa
cat /etc/hosts | grep argocd
```

### Lỗi "x509: certificate signed by unknown authority"

Thêm flag `--insecure` khi dùng ArgoCD CLI (bỏ qua kiểm tra certificate):

```bash
argocd login argocd.local --username admin --insecure
```
