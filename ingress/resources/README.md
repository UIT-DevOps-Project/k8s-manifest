# Ingress resources note

- Argo CD manages only the ingress controller app: ingress-nginx.
- Application ingress objects are expected to be managed by Helm charts per environment.
- Keep this folder only for optional manual ingress manifests when needed.
- If you add manual ingress manifests here, place files directly under ingress/resources.