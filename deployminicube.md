# Deploying nuvae to Minikube (Local Kubernetes)

This guide walks you through running the nuvae platform locally on Minikube using the Helm chart under `infra/k8s/helm/apps`. It is tailored to Windows PowerShell, but commands are similar on macOS/Linux.

## What you’ll get
- Minikube cluster with a local registry and ingress
- A single “tenant” namespace running:
  - PostgreSQL (StatefulSet + PVC)
  - RabbitMQ (Deployment + Service)
  - nuvae web app (Deployment + Service + Ingress)
  - ai-services (Deployment + Service + Ingress)
  - Workers (Deployment)
- Optional: HL7-to-FHIR converter
- Local port-forwarding or Ingress access for UIs/services

## Prerequisites
- Windows 10/11 with PowerShell
- Installed:
  - Minikube: https://minikube.sigs.k8s.io
  - kubectl: https://kubernetes.io/docs/tasks/tools
  - Helm v3+: https://helm.sh/docs/intro/install
  - Docker Desktop (recommended) or another Minikube driver

Verify versions:
```powershell
minikube version
kubectl version --client
helm version
```

## 1) Start Minikube
Start with adequate resources (adjust as needed):
```powershell
minikube start --cpus=4 --memory=8192 --disk-size=50g --driver=docker
```
Enable common addons:
```powershell
minikube addons enable ingress
minikube addons enable metrics-server
```
If you plan to use the chart's Traefik-based Ingress (ingressClassName: traefik), install Traefik via Helm:
```powershell
helm repo add traefik https://traefik.github.io/charts; helm repo update
helm upgrade --install traefik traefik/traefik --namespace traefik --create-namespace
```
If you prefer to use Minikube's default NGINX Ingress addon instead, change Ingress resources to use class "nginx" (or remove the `ingressClassName: traefik` lines) for local.
Optional (handy for testing):
```powershell
minikube addons enable dashboard
```

Check status:
```powershell
minikube status
```

## 2) Prepare a Tenant Values file
The chart is multi-tenant. Copy the template and fill in local/dev-safe values. From repo root:
```powershell
Copy-Item infra/k8s/helm/apps/values-template.yaml infra/k8s/helm/apps/values-minikube.yaml
```
Edit `infra/k8s/helm/apps/values-minikube.yaml`:
- Set `tenant.name` to a short lowercase word, e.g. `dev`
- For Minikube, use in-cluster service names for DB/RabbitMQ (the templates already do this). Use simple credentials for local only.
- For images, ensure `release_version: "dev"` matches tags available in your registry/ACR, or replace images with ones you can pull (see Images section).
- For `webApp.env` and `aiServices.env`, provide minimal required variables. For local bring-up, consider stubbing external services (OpenAI/Azure) or leave blank if not strictly required at start. Example minimal edits:

```yaml
release_version: "dev"

tenant:
  name: "dev"
  domain: "dev.local"   # Not used unless you wire DNS to Minikube ingress
  database:
    user: "devuser"
    password: "devpass"
    name: "nuvae_dev"
    host: "postgresql-dev.dev:5432"  # internal service
  rabbitmq:
    user: "devmq"
    password: "devmqpass"
    host: "rabbitmq-dev.dev:5672"    # internal service

webApp:
  env:
    DATABASE_URL: "postgresql://devuser:devpass@postgresql-dev.dev:5432/nuvae_dev"
    NEXTAUTH_URL: "http://dev.local"           # If using ingress+DNS; otherwise ok to leave
    NEXTAUTH_SECRET: "localdevsecret"
    OPENAI_API_KEY: ""
    AZURE_OPENAI_API_KEY: ""
    AZURE_OPENAI_ENDPOINT: ""
    AZURE_OPENAI_API_VERSION: ""
    AZURE_OPENAI_DEPLOYMENT_NAME: ""
    AZURE_SEARCH_ENDPOINT: ""
    AZURE_SEARCH_ADMIN_KEY: ""
    AZURE_STORAGE_CONNECTION_STRING: ""
    AZURE_STORAGE_CONTAINER_NAME: ""
    AZURE_STORAGE_ACCOUNT_NAME: ""
    AZURE_STORAGE_ACCOUNT_KEY: ""
    AI_SERVICES_ENDPOINT: "http://ai-services-dev.dev"  # internal service if needed
    EMAIL_SERVER: ""
    EMAIL_FROM: "noreply@local"
    NEXT_PUBLIC_AI_SERVICES_ENDPOINT: "http://ai-services-dev.dev"
    REJECTION_FILE_CONTAINER: ""
    E2E_TEST_URL: "http://dev.local/"
    ELASTICSEARCH_URL: ""
    ELASTICSEARCH_USERNAME: ""
    ELASTICSEARCH_PASSWORD: ""
    CONTRACT_SEARCH: ""
    EDI_FOLDER: "/mnt/edi"

aiServices:
  env:
    AZURE_OPENAI_API_KEY: ""
    AZURE_OPENAI_ENDPOINT: ""
    AZURE_OPENAI_API_VERSION: ""
    AZURE_OPENAI_DEPLOYMENT_NAME: ""
    AZURE_SEARCH_ADMIN_KEY: ""
    AZURE_SEARCH_ENDPOINT: ""
    AZURE_STORAGE_CONNECTION_STRING: ""
    DATABASE_URL: "postgresql://devuser:devpass@postgresql-dev.dev:5432/nuvae_dev"
    SERVICE_NAME: "ai-services-dev"
```

Storage mounts (`storage.contracts`, `storage.rates`, `storage.edi`) use Azure Blob CSI in templates. For Minikube (no Azure), set them to disabled:
```yaml
storage:
  contracts:
    enabled: false
  rates:
    enabled: false
  edi:
    enabled: false
```

PersistentVolumeClaims in PostgreSQL and RabbitMQ templates reference a cloud StorageClass (e.g., `managed-csi-premium`). Minikube doesn't have this. Use one of these approaches:

- Option A (fast): Create a StorageClass alias named `managed-csi-premium` that uses Minikube's default provisioner.
  ```yaml
  # managed-csi-premium-sc.yaml
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: managed-csi-premium
  provisioner: k8s.io/minikube-hostpath
  volumeBindingMode: Immediate
  reclaimPolicy: Delete
  ```
  Apply it:
  ```powershell
  kubectl apply -f managed-csi-premium-sc.yaml
  ```

- Option B (local-only edit): Temporarily remove or change `storageClassName:` in the `templates/postgresql.yaml` and `templates/rabbitmq.yaml` PVCs to use the cluster default (often `standard`) and re-run Helm.

- Option C (advanced): Maintain a Helm values/patch overlay to substitute the StorageClass for local.

RabbitMQ Service is `LoadBalancer` in template. Minikube maps this via Tunnel. You can leave it or change to `ClusterIP` locally by editing the template or overriding via a patch (advanced). For simplicity, leave as-is and use port-forwarding.

## 3) Images for local
By default, templates reference Azure Container Registry images like:
- `walecontainerregistry.azurecr.io/nuvae-web-client:dev`
- `walecontainerregistry.azurecr.io/nuvae-ai-services:dev`
- `walecontainerregistry.azurecr.io/nuvae-celery-workers:dev`

Options:
- If you have ACR access and Docker is logged in, no change needed.
- Otherwise, retag to Docker Hub images you can pull or build locally and push to Minikube’s cache:

Build locally and load into Minikube:
```powershell
# From repo root, example for web app Dockerfile (adjust path/image)
docker build -t nuvae-web-client:dev .\apps\nuvae-web-app
minikube image load nuvae-web-client:dev

# Repeat for ai-services and workers if needed
```
Then override image in values with:
- For web-app deployment (edit `templates/nuvae-web-app.yaml` is not recommended). Prefer a values override by adding to `values-minikube.yaml`:

```yaml
images:
  webApp: "nuvae-web-client:dev"
  aiServices: "nuvae-ai-services:dev"
  workers: "nuvae-celery-workers:dev"
```

If the templates are hardcoded, you can temporarily change them locally or use `--set-string` to override `release_version` to match an image that exists in the registry you can pull from.

## 4) Install the chart
From repo root:
```powershell
helm upgrade --install dev-nuvae infra/k8s/helm/apps -f infra/k8s/helm/apps/values-minikube.yaml --create-namespace --namespace dev
```
Note: The templates use `{{ .Values.tenant.name }}` as namespace for many objects. Ensure your chosen namespace matches `tenant.name` (e.g., `dev`). If not, add `--namespace dev` and set `tenant.name: dev`.

Check resources:
```powershell
kubectl get all -n dev
kubectl get pvc,pv -n dev
kubectl describe pod -n dev
```

Wait for pods to be Running/Ready.

If any Service of type LoadBalancer is present (e.g., RabbitMQ in this chart) and you want an external IP, run Minikube Tunnel in a separate elevated PowerShell window:
```powershell
minikube tunnel
```
Alternatively, prefer port-forwarding (see below) for local access.

## 5) Database init (optional)
If your app requires schema:
- Exec into a workers/web pod and run migrations, or
- Port-forward Postgres and run migrations from your machine.

Port-forward Postgres:
```powershell
kubectl port-forward svc/dev-postgresql -n dev 5432:5432
```
Now you can connect via `postgresql://devuser:devpass@localhost:5432/nuvae_dev`.

Run migrations based on your project tooling (Prisma, Alembic, etc.). Check repository docs for exact steps.

## 6) Accessing services locally

### Option A: Port-forwarding (easy)
- Web App:
```powershell
kubectl port-forward svc/dev-nuvae-web-app -n dev 8080:80
# Visit http://localhost:8080
```
- AI Services:
```powershell
kubectl port-forward svc/dev-ai-services -n dev 8081:80
# Visit http://localhost:8081
```
- RabbitMQ Management:
```powershell
kubectl port-forward svc/dev-rabbitmq -n dev 15672:15672
# Visit http://localhost:15672 (default user/pass from your values)
```
If you need AMQP port from your host:
```powershell
kubectl port-forward svc/dev-rabbitmq -n dev 5672:5672
```

### Option B: Ingress (requires DNS or hosts entry)
The templates create Ingress resources with hosts like `dev.nuvae.ai` and `ai-services-dev.nuvae.ai`. For Minikube, either:
- Change hosts in the templates/values to something like `dev.local` and `ai-services.dev.local`, or
- Add hosts entries pointing to Minikube IP.

Get Minikube IP:
```powershell
minikube ip
```
Add to `C:\Windows\System32\drivers\etc\hosts` (run editor as Admin):
```
<MINIKUBE_IP> dev.local ai-services.dev.local rabbitmq.dev.local
```
Ensure your Ingress hosts match these. Then open:
- http://dev.local
- http://ai-services.dev.local

If using TLS secrets in templates (e.g., `wildcard-nuvae-ai-...`), either disable TLS in templates for local, or create dummy self-signed cert secrets (advanced). For local dev, consider removing `tls:` blocks in Ingress manifests or setting `ingress.tls: false` if you add such a value.

## 7) Logs and troubleshooting
- Describe pods for events:
```powershell
kubectl describe pod <pod-name> -n dev
```
- View logs:
```powershell
kubectl logs deploy/dev-nuvae-web-app -n dev --tail=200 -f
kubectl logs deploy/dev-ai-services -n dev --tail=200 -f
kubectl logs deploy/dev-workers -n dev --tail=200 -f
kubectl logs statefulset/dev-postgresql -n dev --tail=200 -f
kubectl logs deploy/dev-rabbitmq -n dev --tail=200 -f
```
- Common issues:
  - ImagePullBackOff: ensure images are accessible; use `minikube image load` or Docker login to ACR.
  - CrashLoopBackOff: missing env vars; check `values-minikube.yaml` and ConfigMaps/Secrets.
  - Ingress not routing: check `minikube addons enable ingress`, hosts file, and Ingress class annotations. If using Traefik class, ensure you installed Traefik.
  - PVC pending: ensure a suitable StorageClass exists. For this chart, create the `managed-csi-premium` StorageClass alias (see above) or edit PVCs to use the default class.

## 8) Optional components
- HL7-to-FHIR Converter: Controlled by `hl7ToFhirConverter.enabled`. The values template shows required envs and secrets. For local dev, you can disable it by setting `enabled: false` to avoid extra dependencies.
- Azure Blob Storage mounts: Disable in `values-minikube.yaml` (see step 2) since Azure CSI is not available on Minikube by default.

## 9) Uninstall
```powershell
helm uninstall dev-nuvae -n dev
kubectl delete namespace dev
```
Optionally stop Minikube:
```powershell
minikube stop
```

## 10) Cheatsheet (Windows PowerShell)
- Get namespace resources:
```powershell
kubectl get all -n dev
```
- Tail logs:
```powershell
kubectl logs deploy/dev-nuvae-web-app -n dev -f
```
- Port-forward web app:
```powershell
kubectl port-forward svc/dev-nuvae-web-app -n dev 8080:80
```
- Edit values on the fly (local only):
```powershell
helm upgrade --install dev-nuvae infra/k8s/helm/apps `
  -f infra/k8s/helm/apps/values-minikube.yaml `
  --namespace dev --create-namespace
```

## Notes mapping to this chart
- Names are constructed as `<tenant>-<component>` via the `resourceName` helper (see `_helpers.tpl`). With `tenant.name: dev`:
  - Web App Service: `dev-nuvae-web-app`
  - AI Services Service: `dev-ai-services`
  - PostgreSQL Service: `dev-postgresql`
  - RabbitMQ Service: `dev-rabbitmq`
  - Workers Deployment: `dev-workers`
- The chart expects environment variables via ConfigMaps and Secrets (see `nuvae-web-app-env-configmap.yaml`, `workers-env-configmap.yaml`, `templates/secrets.yaml`). Ensure required keys are present in your values file.
- Storage mounts templates use Azure Blob CSI and blank storageClassName; these are not suitable for Minikube by default—disable them locally.
- PVC StorageClass: PostgreSQL and RabbitMQ PVCs reference `managed-csi-premium`; on Minikube create an alias StorageClass or remove the explicit class for local use.

If you want, we can add a dedicated `values-minikube.yaml` with sensible defaults and `ingress` toggles to make this even smoother.
