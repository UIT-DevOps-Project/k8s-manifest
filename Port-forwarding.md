# Port Forwarding
## Argocd
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```
## Grafana
```bash
kubectl port-forward -n monitoring svc/monitoring-stack-grafana 3000:80
```