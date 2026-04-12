# Ingress resources

- This folder is managed by Argo CD application `ingress-resources`.
- All application routing ingress objects are centralized here.
- Application Helm charts no longer include ingress templates.
- Current routing split:
	- `fe.app.local`: production frontend (`/`) and production backend (`/api`)
	- `staging.fe.app.local`: staging frontend (`/`) and staging backend (`/api`)


