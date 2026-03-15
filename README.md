# Cấu hình apps
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

chạy những lệnh sau để apply cấu hình của k8s:
``` kubectl apply -f apps/ -R ```
kiểm tra các pods đã được tạo
``` kubectl get pods -n devops-app ```
kiểm tra tất cả tài nguyên đã được tạo:
``` kubectl get all -n devops-app ```