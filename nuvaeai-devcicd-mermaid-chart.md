```mermaid
flowchart TD
  %% Define styles for clarity
  classDef stage fill:#D6EAF8,stroke:#3498DB,stroke-width:2px,color:#1B4F72,font-weight:bold;
  classDef step fill:#F9E79F,stroke:#B7950B,stroke-width:1.5px,color:#7D6608,font-style:italic;
  classDef decision fill:#ABEBC6,stroke:#239B56,stroke-width:2px,color:#1D8348,font-weight:bold;

  %% Start
  A[🔔 **Trigger: Code Push / Manual Run**<br>Starts pipeline when new code is pushed or manually triggered]
  class A stage;

  %% Checkout Code
  B[📥 **Checkout Repository**<br>Get full source code and git history (fetch-depth=0) to enable semantic-release analysis]
  class B step;

  %% Setup Node
  C[⚙️ **Setup Node.js Environment**<br>Prepare Node.js version 20 for semantic-release & npm commands]
  class C step;

  %% Setup Semantic Release
  D[📦 **Install Semantic Release Packages**<br>Install semantic-release & plugins locally in an isolated directory]
  class D step;

  %% Run Semantic Release
  E[🚦 **Run Semantic Release**<br>Analyze commits to decide if a new release should be created]
  class E step;

  %% Check Release Result
  F{❓ **New Release Published?**<br>Did semantic-release publish a new version?}
  class F decision;

  %% If yes: output release info
  G[📄 **Extract Release Version Details**<br>Parse major, minor, patch version numbers from release output]
  class G step;

  %% If no: get latest release version
  H[🔍 **Fetch Latest Release Version**<br>If no new release, retrieve latest version from GitHub releases or use 0.0.0]
  class H step;

  %% Output release info
  I[📢 **Output Release Version Info**<br>Display new or existing release version details for downstream jobs]
  class I step;

  %% Build AI Services
  J[🧠 **Build & Push AI Services Docker Image**<br>Copy Prisma schema, download tiktoken files, build & push image to Azure Container Registry]
  class J step;

  %% Build Web App
  K[🌐 **Build & Push Web App Docker Image**<br>Clean tenants folder, setup environment variables, build & push image to ACR]
  class K step;

  %% Build Workers
  L[⚡ **Build & Push Celery Workers Docker Image**<br>Download rate sheets, extract, build & push image to ACR]
  class L step;

  %% Build Prisma
  M[📚 **Build & Push Prisma Docker Image**<br>Build & push Prisma database layer image]
  class M step;

  %% Build HL7-to-FHIR Converter
  N[🔄 **Build & Push HL7-to-FHIR Converter Docker Image**<br>Setup Java 17, cache Maven deps, build & push image]
  class N step;

  %% Deploy Notification
  O[📲 **Send Microsoft Teams Notification**<br>Notify team about new release status with changelog and build summary]
  class O step;

  %% Deployment job
  P[🚀 **Deploy Services**<br>Deploy built images using Ansible or other orchestration tooling]
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

  %% Add style to deploy & notify
  class P,O stage;

  %% Add some extra detailed notes on builds
  subgraph "Build Components"
    direction TB
    J
    K
    L
    M
    N
  end
