```mermaid
flowchart TD
  %% Define styles for clarity
  classDef job fill:#D6EAF8,stroke:#2980B9,stroke-width:2px,color:#1B2631,font-weight:bold;
  classDef step fill:#F9E79F,stroke:#B7950B,stroke-width:1.5px,color:#7D6608,font-style:italic;
  classDef decision fill:#ABEBC6,stroke:#239B56,stroke-width:2px,color:#145A32,font-weight:bold;

  %% Workflow trigger
  A["Trigger on push to dev, feature/*, bugfix/*, hotfix/* branches"]:::job

  %% detect-changes job
  B["Job: detect-changes\n- Runs on ubuntu-latest\n- Checkout full repo\n- Detect changed paths via dorny/paths-filter\nOutputs:\n - ai-services-changed\n - web-app-changed\n - prisma-changed\n - hl7-converter-changed"]:::job

  %% build-and-deploy-ai-services job
  C["Job: build-and-deploy-ai-services\n- Needs detect-changes\n- Checkout repo with submodules\n- Setup Docker Buildx\n- Login to Azure Container Registry\n- Copy Prisma schema\n- Create tiktoken data directory\n- Download & verify tiktoken files\n- Generate Docker tags\n- Build & push AI Services image"]:::job

  %% build-and-deploy-web-app job
  D["Job: build-and-deploy-web-app\n- Needs detect-changes\n- Checkout repo with submodules\n- Setup Docker Buildx\n- Login to Azure Container Registry\n- Clean tenants folder\n- Generate Docker tags\n- Build & push Web App image with many build args (env secrets, keys, URLs)"]:::job

  %% build-and-deploy-celery-app job
  E["Job: build-and-deploy-celery-app\n- Needs detect-changes\n- Checkout repo\n- Setup Docker Buildx\n- Login to Azure Container Registry\n- Download rate sheets zip\n- Extract rate sheets\n- Generate Docker tags\n- Build & push Celery Workers image"]:::job

  %% build-and-deploy-prisma job
  F["Job: build-and-deploy-prisma\n- Needs detect-changes\n- Checkout repo\n- Setup Docker Buildx\n- Login to Azure Container Registry\n- Generate Docker tags\n- Build & push Prisma image with DATABASE_URL build arg"]:::job

  %% build-and-deploy-hl7-converter job
  G["Job: build-and-deploy-hl7-converter\n- Needs detect-changes\n- Checkout repo\n- Setup Docker Buildx\n- Login to Azure Container Registry\n- Setup JDK 17\n- Cache Maven dependencies\n- Build Java application (mvn clean package)\n- Generate Docker tags\n- Build & push HL7-to-FHIR Converter image"]:::job

  %% deploy job
  H["Job: deploy\n- Needs all build jobs\n- Checkout repo\n- Install Ansible\n- Run Ansible playbook for deployment to development"]:::job

  %% Connections
  A --> B
  B --> C
  B --> D
  B --> E
  B --> F
  B --> G

  C --> H
  D --> H
  E --> H
  F --> H
  G --> H
