```mermaid
flowchart TD
  %% Styling
  classDef trigger fill:#FFD966,stroke:#B58900,color:#5C4500,font-weight:bold;
  classDef job fill:#D0E8F2,stroke:#2176C7,color:#0B2948,font-weight:bold;
  classDef step fill:#FFF2CC,stroke:#B58900,color:#5C4500,font-style:italic;
  classDef artifact fill:#CFF0CC,stroke:#239B56,color:#145A32,font-weight:bold;
  classDef deploy fill:#FADBD8,stroke:#C0392B,color:#78281F,font-weight:bold;

  %% Trigger node
  A[Push on branches: dev, feature/*, bugfix/*, hotfix/*]:::trigger

  %% detect-changes job and steps
  B[detect-changes job]:::job
  B1[Checkout repo with full history]:::step
  B2[Run dorny/paths-filter to detect changes]:::step
  B3[Outputs changed paths flags]:::artifact

  %% build-and-deploy-ai-services job and steps
  C[AI Services Build job]:::job
  C1[Checkout repo with submodules]:::step
  C2[Setup Docker Buildx]:::step
  C3[Login to Azure Container Registry]:::step
  C4[Copy Prisma schema to AI services folder]:::step
  C5[Create tiktoken data directory]:::step
  C6[Download & verify tiktoken files]:::step
  C7[Generate Docker tags]:::artifact
  C8[Build & push AI Services Docker image]:::step

  %% build-and-deploy-web-app job and steps
  D[Web App Build job]:::job
  D1[Checkout repo with submodules]:::step
  D2[Setup Docker Buildx]:::step
  D3[Login to Azure Container Registry]:::step
  D4[Clean tenants folder]:::step
  D5[Generate Docker tags]:::artifact
  D6[Build & push Web App Docker image with env variables]:::step

  %% build-and-deploy-celery-app job and steps
  E[Celery Workers Build job]:::job
  E1[Checkout repo with submodules]:::step
  E2[Setup Docker Buildx]:::step
  E3[Login to Azure Container Registry]:::step
  E4[Download rates file zip]:::step
  E5[Extract rates data to worker folder]:::step
  E6[Generate Docker tags]:::artifact
  E7[Build & push Celery Workers Docker image]:::step

  %% build-and-deploy-prisma job and steps
  F[Prisma Build job]:::job
  F1[Checkout repo with submodules]:::step
  F2[Setup Docker Buildx]:::step
  F3[Login to Azure Container Registry]:::step
  F4[Generate Docker tags]:::artifact
  F5[Build & push Prisma Docker image with DATABASE_URL]:::step

  %% build-and-deploy-hl7-converter job and steps
  G[HL7-to-FHIR Converter Build job]:::job
  G1[Checkout repo with submodules]:::step
  G2[Setup Docker Buildx]:::step
  G3[Login to Azure Container Registry]:::step
  G4[Setup JDK 17]:::step
  G5[Cache Maven dependencies]:::step
  G6[Build Java app (mvn clean package)]:::step
  G7[Generate Docker tags]:::artifact
  G8[Build & push HL7 Converter Docker image]:::step

  %% deploy job and steps
  H[Deploy job]:::deploy
  H1[Checkout repo with submodules]:::step
  H2[Install Ansible]:::step
  H3[Run Ansible playbook to deploy]:::step

  %% Links and dependencies
  A --> B
  B --> B1 --> B2 --> B3

  B3 --> C
  C --> C1 --> C2 --> C3 --> C4 --> C5 --> C6 --> C7 --> C8
  B3 --> D
  D --> D1 --> D2 --> D3 --> D4 --> D5 --> D6
  B3 --> E
  E --> E1 --> E2 --> E3 --> E4 --> E5 --> E6 --> E7
  B3 --> F
  F --> F1 --> F2 --> F3 --> F4 --> F5
  B3 --> G
  G --> G1 --> G2 --> G3 --> G4 --> G5 --> G6 --> G7 --> G8

  C8 --> H
  D6 --> H
  E7 --> H
  F5 --> H
  G8 --> H

  H --> H1 --> H2 --> H3
