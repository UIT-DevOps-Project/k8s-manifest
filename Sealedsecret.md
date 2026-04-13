# Hướng dẫn cấu hình Sealed Secrets cho Team

Tài liệu này hướng dẫn cách đồng bộ chìa khóa mã hóa (Sealed Secrets Key) giữa các thành viên trong team, giúp tất cả các cluster local trong dự án đều có thể giải mã được các file Sealed Secrets chung (kể cả trên Git).

## ⚠️ Cảnh báo Bảo mật QUAN TRỌNG
**KHÔNG BAO GIỜ** được commit file `private-key.yaml` lên Git hoặc các nền tảng chia sẻ public. File này là "chìa khóa Master" để giải mã toàn bộ thông tin nhạy cảm của dự án (như DB Password, Github Token). Chỉ chia sẻ nội bộ qua các kênh an toàn.

File `pub-cert.pem` là Public Key, hoàn toàn an toàn và đã được lưu trữ sẵn trên Git.

---

## Dành cho thành viên team (Cấu hình máy Local)

Nếu bạn vừa tạo một cụm Kubernetes local mới (KinD, Minikube, Docker Desktop...), cụm của bạn sẽ tự động sinh ra một key mới và KHÔNG THỂ giải mã được các files Sealed Secrets của nhóm. Để đồng bộ, bạn cần import Private Key của team bằng các bước sau:

### Bước 1: Cài đặt Sealed Secrets Controller
Nếu cụm K8s của bạn chưa cài Sealed Secrets Controller, hãy chạy lệnh gốc này để cài đặt:
```bash
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/latest/download/controller.yaml
```

### Bước 2: Apply Private Key của Team vào Kube-system
1. Nhận file `private-key.yaml` trực tiếp từ bạn/đồng nghiệp trong team. Đảm bảo file này nằm ở thư mục an toàn.
2. Nạp key vào cụm K8s local của bạn bằng lệnh:
```bash
kubectl apply -f private-key.yaml
```

### Bước 3: Khởi động lại Controller
Để Controller bỏ qua khóa mặc định nạp Private Key mới, bạn bắt buộc phải khởi động lại (restart) deployment:
```bash
kubectl rollout restart deployment sealed-secrets-controller -n kube-system
kubectl rollout status deployment sealed-secrets-controller -n kube-system
```
✅ Sau khi pod chạy lên lại, cụm K8s của bạn lúc này đã dùng chung 1 "ổ khóa" với team và có thể giải mã thành công các file SealedSecret qua ArgoCD hoặc `kubectl apply`.

---

## Hướng dẫn tự tạo và mã hóa (Seal) một Secret mới

Bất kỳ thành viên nào cũng có thể đóng gói (seal) Secret mới bằng Public Certificate (`cert/pub-cert.pem`). Bạn cần cài công cụ `kubeseal` trên máy trước khi thao tác.

**Cách 1: Seal từ một file yaml Secret thuần:**
```bash
# 1. Tạo file secret yaml raw (đừng commit file này)
kubectl create secret generic my-secret --namespace=production \
  --from-literal=password=my-super-secret \
  --dry-run=client -o yaml > my-secret.yaml

# 2. Seal data vào file mới (dùng pub-cert.pem)
kubeseal --cert cert/pub-cert.pem --format yaml < my-secret.yaml > my-sealed-secret.yaml

# 3. Apply lên cụm hoặc commit file my-sealed-secret.yaml lên Git
kubectl apply -f my-sealed-secret.yaml
```

**Cách 2: Đóng gói bí mật nhanh trong 1 dòng lệnh:**
```bash
echo -n "my-super-secret" | kubectl create secret generic my-secret --namespace=staging \
  --dry-run=client --from-file=password=/dev/stdin -o yaml | \
  kubeseal --cert cert/pub-cert.pem --format yaml > my-sealed-secret.yaml
```