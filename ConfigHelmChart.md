# Cấu hình helm chart cho apps
## Tạo các file yaml
- backend: backend-deployment và backend-service
    + backend-deployment: đang có strategy: rollingupdate (để tạm có muốn đổi strategy gì đổi), gắn selector_nodes chọn node_apps trong kind cluster, gắn ghcr-secret để cấp quyền pull image private từ package, sử dụng key từ postgres-secret để kết nối tới database.
    + backend-service: dùng clusterIP để cluster cấp IP cố định giao tiếp trong cụm
- frontend: frontend-deployment và frontend-service
    + frontend-deployment: tương tự backend nhưng không dùng postgres-secret để kết nối database
    + frontend-service: dùng clusterIp cấp Ip cố địnhk giao tiếp trong cụm
- postgres: postgres-statefulset và postgres-serivce
    + postgres-statefulset: dùng statfulset để cấp ID cố định cho pod và dùng volume PV và PVC lưu dữ liệu đang request 1Gi (có thể sửa tùy ý)
    + postgres-service: đang dùng clusterIP để giao tiếp trong cụm nhưng không cấp IP
- ghcr-secret
    + tạo secret để cấp secret cho pod pull image về từ package private
    + thêm username, personal_token_github có quyền kéo image trong package của uit-devops-project
- namespace
    + tạo namespace để dễ quản lý
- postgres-secret
    + tạo file secret lưu thông tin cấu hình của postgres database.
## Sử dụng kubeseal để mã hóa secret và apply sealed-Secret-Controller vào cluster để dùng
- Cách hoạt động tạo secret bằng kubectl. Sau khi tạo secret dùng lệnh kubeseal để mã hóa secret và lưu vào file *-sealedsecret chuyển tiếp vào folder base/ .Khi cluster online thì sẽ gửi request tới sealed-secret-controller mã hóa file *-sealedsecret thành *-secret và sử dụng nó. 
=> Nhưng nếu cluster không online thì sẽ không thể mã hóa được file secret. Vì vậy có tích hợp cert/pub-cert.pem đây là public key lấy từ controller hỗ trợ mã hóa không cần gọi tới cluster thuận lợi cho CI/CD.
### workflow triển khai
- ``` kind-config-> nginx-controller-> base-> apps ```
## Cài đặt Kubeseal Controller để mã hóa kubesealsecret
``` kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/latest/download/controller.yaml ```
- Sau đó apply privatekey.pem để giải mã sealedsecret
- ``` kubectl apply -f privatekey.pem ```\
- Sau đó apply base trước
- ``` kubectl apply -R -f base/ ``` 
- để tạo các tài nguyên như namespace và apply secret
## Sử dụng helm chart để apply backend, frontend, database
- Apply môi trường staging test thử:
- ```helm install backend apps/backend -n staging -f apps/backend/values-staging.yaml ```
- Apply môi trường production test thử:
- ```helm install backend apps/backend -n production -f apps/backend/values-production.yaml```
- Nếu thay đổi thì có thể dùng lệnh này để upgrade
- ``` helm upgrade backend apps/backend -n staging -f apps/backend/values-production.yaml```
- Check version helm chart:
- ``` helm list -A ```
- Check pods:
- ```kubectl get pods -n devops-app ```

## Workflow apply:
- kind: dùng để config và create cluster
- nginx-controller: hỗ trợ cho ingress trong cluster
- base: apply cấu hình cơ bản cho cluster để deploy app như namespace, ingress, secret
- apps: dùng để deploy gồm backend, frontend, database
- cert: lưu pub-cert mã hóa secret để sử dụng thuận lợi cho CI/CD không cần connect tới cluster



