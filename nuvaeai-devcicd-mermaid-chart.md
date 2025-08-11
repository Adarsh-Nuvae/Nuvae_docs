```mermaid
flowchart TD
  %% Define styles for clarity
  classDef stage fill:#D6EAF8,stroke:#3498DB,stroke-width:2px,color:#1B4F72,font-weight:bold;
  classDef step fill:#F9E79F,stroke:#B7950B,stroke-width:1.5px,color:#7D6608,font-style:italic;
  classDef decision fill:#ABEBC6,stroke:#239B56,stroke-width:2px,color:#1D8348,font-weight:bold;

  %% Start
  A["Trigger: Code Push or Manual Run - Starts pipeline when new code is pushed or manually triggered"]
  class A stage;

  %% Checkout Code
  B["Checkout Repository - Get full source code and git history (fetch-depth=0) to enable semantic-release analysis"]
  class B step;

  %% Setup Node
  C["Setup Node.js Environment - Prepare Node.js version 20 for semantic-release and npm commands"]
  class C step;

  %% Setup Semantic Release
  D["Install Semantic Release Packages - Install semantic-release and plugins locally in an isolated directory"]
  class D step;

  %% Run Semantic Release
  E["Run Semantic Release - Analyze commits to decide if a new release should be created"]
  class E step;

  %% Check Release Result
  F{"New Release Published? - Did semantic-release publish a new version?"}
  class F decision;

  %% If yes: output release info
  G["Extract Release Version Details - Parse major, minor, patch version numbers from release output"]
  class G step;

  %% If no: get latest release version
  H["Fetch Latest Release Version - If no new release, retrieve latest version from GitHub releases or use 0.0.0"]
  class H step;

  %% Output release info
  I["Output Release Version Info - Display new or existing release version details for downstream jobs"]
  class I step;

  %% Build AI Services
  J["Build and Push AI Services Docker Image - Copy Prisma schema, download tiktoken files, build and push image to Azure Container Registry"]
  class J step;

  %% Build Web App
  K["Build and Push Web App Docker Image - Clean tenants folder, setup environment variables, build and push image to ACR"]
  class K step;

  %% Build Workers
  L["Build and Push Celery Workers Docker Image - Download rate sheets, extract, build and push image to ACR"]
  class L step;

  %% Build Prisma
  M["Build and Push Prisma Docker Image - Build and push Prisma database layer image"]
  class M step;

  %% Build HL7-to-FHIR Converter
  N["Build and Push HL7-to-FHIR Converter Docker Image - Setup Java 17, cache Maven dependencies, build and push image"]
  class N step;

  %% Deploy Notification
  O["Send Microsoft Teams Notification - Notify team about new release status with changelog and build summary"]
  class O step;

  %% Deployment job
  P["Deploy Services - Deploy built images using Ansible or other orchestration tooling"]
  class P step;

  %% Workflow: start to checkout
  A --> B;
  B --> C;
  C --> D;
  D --> E;
  E --> F;

  %% Semantic release decision
  F -->|Yes| G;
  F -->|No| H;

  %% Both branches continue to output version info
  G --> I;
  H --> I;

  %% After version info, parallel builds start if a new release or manual trigger
  I --> J;
  I --> K;
  I --> L;
  I --> M;
  I --> N;

  %% After builds finish, deploy and notify
  J --> P;
  K --> P;
  L --> P;
  M --> P;
  N --> P;

  P --> O;

  %% Add parallel styling to builds
  class J,K,L,M,N stage;

  %% Add style to deploy and notify
  class P,O stage;

  %% Group build steps
  subgraph "Build Components"
    direction TB
    J
    K
    L
    M
    N
  end
