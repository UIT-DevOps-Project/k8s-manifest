# Port Forwarding
## Argocd
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 &

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
## Grafana
```bash
kubectl port-forward -n monitoring svc/monitoring-stack-grafana 3000:80
```

## OpenSearch API
```bash
kubectl port-forward -n monitoring svc/opensearch-cluster-master 9200:9200
```

## OpenSearch Dashboards
```bash
kubectl port-forward -n monitoring svc/opensearch-dashboards 5601:5601
username: admin
pass : contact me :)))
```