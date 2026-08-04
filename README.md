# Zenus App Insights — updated setup checklist (review)

**Project:** [ZB-CS - PP Digital Portal Solution](https://dev.azure.com/ZenusBankInternational/ZB-CS%20-%20PP%20Digital%20Portal%20Solution)  
**Org:** `https://dev.azure.com/ZenusBankInternational`

Same App Insights connection string **value** everywhere in an environment (QA vs PROD).  
Only the **variable name** changes:

| Layer | Library variable name |
|-------|------------------------|
| Frontends (Webapp / Retail / Admin) | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` |
| Backends / scripts | `APPLICATIONINSIGHTS_CONNECTION_STRING` |

---

## Status as of audit (read-only)

| Item | Status |
|------|--------|
| Library Insights vars (all FE/BE groups) | **Not added yet** |
| Webapp / Retail `.env.required` Insights line | **Not added yet** |
| Admin CI Insights `--build-arg` | **Not added yet** |
| Telemetry code (`telemtry-08` PRs) | Must be on `qa` before Insights can bake into images |

---

## Why “ADD” instead of “replace whole Admin CI file”?

Admin CI on `qa`/`main` **already** passes Docker build-args for app config:

- `REACT_APP_APP_BASE_URL`
- `REACT_APP_DEVELOPMENT`
- `REACT_APP_ENC_ALGO`
- `REACT_APP_ENC_KEY`

If you **replace** the whole YAML with an Insights-only `arguments:` line, those four build-args disappear and **Admin build/runtime config breaks**.

**Use of adding:** keep every existing `--build-arg`, and append **one more** for App Insights so the Dockerfile can receive `REACT_APP_APPINSIGHTS_CONNECTION_STRING` at image build time.

---

# PART 0 — Prerequisite (do first)

1. Merge telemetry PRs / ensure `qa` has Insights Dockerfile support (`ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING` + write into `.env.prod`).
2. Get QA App Insights **Connection string** from Azure Portal (Overview).
3. Use that same string value for all QA Library vars below.

---

# PART A — Frontends

## A1. Corporate Webapp (`fintech_webapp`)

### 1) Library variable groups

| Group | Variable | Secret |
|-------|----------|--------|
| `ZB-FintechWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

### 2) File `.env.required` — current vs update

**CURRENT (`qa`) today:**

```env
REACT_APP_APP_BASE_URL=__REACT_APP_APP_BASE_URL__
REACT_APP_DEVELOPMENT=__REACT_APP_DEVELOPMENT__
REACT_APP_ENC_ALGO=__REACT_APP_ENC_ALGO__
REACT_APP_ENC_KEY=__REACT_APP_ENC_KEY__
```

**UPDATE — add this one line (do not remove existing lines):**

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

**AFTER (expected):**

```env
REACT_APP_APP_BASE_URL=__REACT_APP_APP_BASE_URL__
REACT_APP_DEVELOPMENT=__REACT_APP_DEVELOPMENT__
REACT_APP_ENC_ALGO=__REACT_APP_ENC_ALGO__
REACT_APP_ENC_KEY=__REACT_APP_ENC_KEY__
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

Commit/push to the branch CD builds from (`qa`).

### 3) Run **Fintech-WebApp-CD**

CD token-replaces `.env.required` → `.env.prod` before Docker. No Admin-style CI build-arg required for Webapp.

---

## A2. Retail Webapp (`fintech_retail_webapp`)

### 1) Library

| Group | Variable | Secret |
|-------|----------|--------|
| `ZB-FintechRetailWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechRetailWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

### 2) File `.env.required` — same pattern as Webapp

**CURRENT:** same four lines (BASE_URL / DEVELOPMENT / ENC_ALGO / ENC_KEY).

**UPDATE — add only:**

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

### 3) Run **Fintech-Retail-Webapp-CD**

---

## A3. Admin Webapp (`fintech_admin_webapp`)

### 1) Library variable groups

| Group | Variable | Secret |
|-------|----------|--------|
| `ZB-FintechAdminWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechAdminWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

### 2) Optional — `.env.required`

Admin Insights is driven by **CI Docker `arguments`**, not CD token-replace. Adding `.env.required` is optional/docs-only.

### 3) File `Pipelines/pipeline_CI.yml` — current vs update

**CURRENT (`qa` / `main`) today:**

```yaml
trigger:
  branches:
    include:
      - main
      - qa

resources:
  repositories:
    - repository: pipelines
      type: git
      name: ZB-CS - PP Digital Portal Solution/pipelines
      ref: refs/heads/main

variables:
  - group: PlatformDetails
  - group: ZB-FintechAdminWebApp-PROD

extends:
  template: pipeline-ci/flow/build-acr.yml@pipelines
  parameters:
    buildMode: docker
    imageName: "fintechadminwebapp"
    dockerfilePath: "$(Build.SourcesDirectory)/Dockerfile"
    arguments: >-
      --build-arg REACT_APP_APP_BASE_URL=$(REACT_APP_APP_BASE_URL)
      --build-arg REACT_APP_DEVELOPMENT=$(REACT_APP_DEVELOPMENT)
      --build-arg REACT_APP_ENC_ALGO=$(REACT_APP_ENC_ALGO)
      --build-arg REACT_APP_ENC_KEY=$(REACT_APP_ENC_KEY)
```

**What to change (ADD / adjust — do not wipe existing build-args):**

1. Prefer branch-based Library groups (QA vs PROD).
2. **Add** Insights `--build-arg` to the existing `arguments` list.

**UPDATED file (recommended):**

```yaml
trigger:
  branches:
    include:
      - main
      - qa

resources:
  repositories:
    - repository: pipelines
      type: git
      name: ZB-CS - PP Digital Portal Solution/pipelines
      ref: refs/heads/main

variables:
  - group: PlatformDetails
  - ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
    - group: ZB-FintechAdminWebApp-PROD
  - ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/qa') }}:
    - group: ZB-FintechAdminWebApp-QA

extends:
  template: pipeline-ci/flow/build-acr.yml@pipelines
  parameters:
    buildMode: docker
    imageName: "fintechadminwebapp"
    dockerfilePath: "$(Build.SourcesDirectory)/Dockerfile"
    arguments: >-
      --build-arg REACT_APP_APP_BASE_URL=$(REACT_APP_APP_BASE_URL)
      --build-arg REACT_APP_DEVELOPMENT=$(REACT_APP_DEVELOPMENT)
      --build-arg REACT_APP_ENC_ALGO=$(REACT_APP_ENC_ALGO)
      --build-arg REACT_APP_ENC_KEY=$(REACT_APP_ENC_KEY)
      --build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)
```

**Do not edit** the shared `pipelines` / `build-acr.yml` repo.  
Parameter name is still `arguments`.

### 4) Dockerfile must accept the ARG (on the branch CI builds)

On `telemtry-08` this already exists. On current `qa` Admin Dockerfile it was **missing** at audit time — merge telemetry first, or ensure `qa` has:

```dockerfile
ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING
ENV REACT_APP_APPINSIGHTS_CONNECTION_STRING=$REACT_APP_APPINSIGHTS_CONNECTION_STRING
# and append into .env.prod used by the React build
```

### 5) Confirm after CI

Rerun **admin-webapp CI** → log **Build image** must contain:

```text
--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=
```

(and still contain the other four `--build-arg` lines).

### 6) Then run **admin-webapp CD**

---

# PART B — Backends / scripts

For each: add Library var → run CD.  
Variable name always: `APPLICATIONINSIGHTS_CONNECTION_STRING` (Secret).  
No `.env.required`. No build-arg.

| Repo | QA group | PROD group | CD pipeline |
|------|----------|------------|-------------|
| `fintech_business_management` | `ZB-FintechBusinessManagment-QA` | `ZB-FintechBusinessManagment-PROD` | **business-management CD** |
| `fintech_business_settings` | `ZB-FintechBusinessSettings-QA` | `ZB-FintechBusinessSettings-PROD` | **business-settings CD** |
| `fintech_documents_management` | `ZB-FintechDocumentsManagement-QA` | `ZB-FintechDocumentsManagement-PROD` | **Fintech-Documents-Management-CD** (or `documents-management CD`) |
| `fintech_notifications_management` | `ZB-FintechNotificationsManagement-QA` | `ZB-FintechNotificationsManagement-PROD` | **Fintech-Notifications-Management-CD** (or `notifications-management CD`) |
| `fintech_super_admin` | `ZB-FintechSupeAdmin-QA` | `ZB-FintechSupeAdmin-PROD` | **Fintech-Super-Admin-CD** (or `super-admin CD`) |
| `fintech_user_management` | `ZB-FintechUserManagment-QA` | `ZB-FintechUserManagment-PROD` | **user-management CD** |
| `fintech_statement_generator` | `ZB-FintechStatementGenerator-QA` | `ZB-FintechStatementGenerator-PROD` | **Fintech-Statement-Generator-CD** (or `statement-generator CD`) |
| `fintech_processing_scripts` | `ZB-FintechProcessingScripts-QA` | `ZB-FintechProcessingScripts-PROD` | **Fintech-Processing-Scripts-CD** (or `processing-scripts CD`) |
| `fintech_management_migrations` | `ZB-FintechManagementMigrations-QA` | `ZB-FintechManagementMigrations-PROD` | **Fintech-Migrations-Management-CD** (or `management-migrations CD`) |

---

# Do not use for Insights

- `ZB-Common-QA` / `ZB-Common-PROD`
- `PlatformDetails` (keep it; ACR/service connections only)

---

# Recommended order

1. Merge telemetry to `qa` (Dockerfile / app code).
2. Add all Library variables (**QA first**).
3. Update Webapp + Retail `.env.required` (add one line each).
4. Update Admin `Pipelines/pipeline_CI.yml` (**add** Insights build-arg + optional branch groups).
5. Run Webapp CD, Retail CD, Admin CI → CD.
6. Run backend CDs.
7. Click apps → verify in App Insights (`RestRequestProcessed`, `operationId`, `component`).
8. Prod later (same vars in `*-PROD` groups).

---

# Retention

Set in Azure Portal → App Insights / Log Analytics (not in git).
