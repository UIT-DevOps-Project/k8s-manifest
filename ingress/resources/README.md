# Ingress resources managed by Argo CD

- Put staging ingress manifests in `ingress/resources/staging`.
- Put production ingress manifests in `ingress/resources/production`.
- Argo CD apps `ingress-staging` and `ingress-production` auto-sync these paths.

Example staging ingress file path:

`ingress/resources/staging/my-service-ingress.yaml`