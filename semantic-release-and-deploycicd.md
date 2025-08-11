# 🚀 Semantic Release & Deploy – Step-by-Step Guide

## 🗺️ Pipeline Overview

```mermaid
flowchart TD
    A[📥 Code Pushed to Main / Manual Trigger] --> B[🤖 Semantic Release]
    B -->|New Release| C[🛠️ Build AI Services]
    B -->|New Release| D[🛠️ Build Web App]
    B -->|New Release| E[🛠️ Build Workers]
    B -->|New Release| F[🛠️ Build Prisma]
    B -->|New Release| G[🛠️ Build HL7 Converter]
    C & D & E & F & G --> H[📦 Push Docker Images to ACR]
    H --> I[💬 Notify Microsoft Teams]
````

---

## 1️⃣ Triggering the Workflow

**Simple Explanation**
The process starts when new code is pushed to the main branch or when we start it manually. Once triggered, it checks permissions and prepares to run all the jobs.

**Technical Explanation**

* **Trigger Events**:

  * `push` to `main` branch.
  * `workflow_dispatch` for manual runs.
* **Permissions**:

  * `contents: write`
  * `issues: write`
  * `pull-requests: read`
  * `id-token: write`

---

## 2️⃣ Semantic Release Job

**Simple Explanation**
This step decides if we need a new version of the software. If yes, it creates a new version number and release notes.

**Technical Explanation**

* Checks out repository with SSH key and GitHub token.
* Sets up Node.js 20.
* Installs `semantic-release` with plugins.
* Runs release analysis to:

  * Determine if a new release is needed.
  * Generate new version (`new-release-version`).
  * Identify type: major, minor, patch.
* If no new release, fetches the current version for reference.

---

## 3️⃣ Build & Deploy AI Services

**Simple Explanation**
If there’s a new release, it builds the AI services, downloads needed models, checks them, and stores the build in our container registry.

**Technical Explanation**

* Path: `./apps/ai-services`
* Downloads **tiktoken** model files from OpenAI.
* Verifies checksum of downloaded files.
* Builds Docker image with:

  * `IMAGE_TAG` set to release version.
  * Model files included in the build context.
* Pushes image to **Azure Container Registry (ACR)**.

---

## 4️⃣ Build & Deploy Web App

**Simple Explanation**
Builds the main web app, adds all required settings like database and API keys, and uploads the image to the registry.

**Technical Explanation**

* Path: `./apps/nuvae-web-app`
* Removes unused tenant folders.
* Builds Docker image with `IMAGE_TAG` as release version.
* Build arguments include:

  * AI service endpoints.
  * Database URL.
  * Storage account credentials.
  * API keys.
* Pushes image to ACR.

---

## 5️⃣ Build & Deploy Workers

**Simple Explanation**
Prepares background worker services by downloading required data, building them, and sending them to the registry.

**Technical Explanation**

* Downloads `AVMC_RateSheets.zip` dataset.
* Extracts contents for build context.
* Builds worker Docker images.
* Pushes to ACR.

---

## 6️⃣ Build & Deploy Prisma

**Simple Explanation**
Builds and uploads the database management service (Prisma) with the correct connection details.

**Technical Explanation**

* Path: `./data/prisma`
* Uses `DATABASE_URL` as a build argument.
* Builds Docker image with release version tag.
* Pushes image to ACR.

---

## 7️⃣ Build & Deploy HL7-to-FHIR Converter

**Simple Explanation**
Builds the healthcare data converter service and sends it to the registry.

**Technical Explanation**

* Path: `./apps/hl7-to-fhir-converter`
* Sets up Java 17 environment.
* Caches Maven dependencies.
* Builds JAR package.
* Builds Docker image with `IMAGE_TAG` set to release version.
* Pushes image to ACR.

---

## 8️⃣ Notify Microsoft Teams

**Simple Explanation**
Sends a summary of what happened to the team in Microsoft Teams, including the version, changes, and build results.

**Technical Explanation**

* Collects changelog from release notes.
* Summarizes status for each job (success, fail, skipped).
* Sends Adaptive Card to Teams with:

  * Release version.
  * Status summary.
  * Docker image details.
  * Changelog preview.
  * Links to release and workflow run.

---

## ✅ End Result

* Automatic versioning.
* All updated services built and pushed to registry.
* Full team notified of deployment status.

```


