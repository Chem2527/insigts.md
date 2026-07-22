# Zenus App Insights — exact setup checklist

Project: `ZB-CS - PP Digital Portal Solution`  
Same App Insights connection string **value** everywhere. Only the **variable name** changes (FE vs BE).

---

## Admin CI — confirmed from real template evaluation log

From **admin-webapp CI** build log (template evaluation):

| Parameter | Current value | Meaning |
|-----------|---------------|---------|
| `parameters['arguments']` | `''` (empty) | Passed straight into Docker@2 `arguments:` |
| Docker@2 Build image | `arguments: ''` | So `docker build` runs with **no** `--build-arg` today |

**Exact parameter name is `arguments`.**  
You do **not** need to edit the shared `pipelines` / `build-acr.yml` repo.  
Pass `arguments` from `fintech_admin_webapp/Pipelines/pipeline_CI.yml`.

Also confirmed: Dockerfile already has `ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING`.

---

# PART A — Frontends

## A1. Corporate Webapp (`fintech_webapp`)

### 1) Library variable groups
| Group | Variable | Secret |
|-------|----------|--------|
| `ZB-FintechWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

### 2) File `fintech_webapp/.env.required` — add:
```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```
Commit/push.

### 3) Run **Fintech-WebApp-CD**  
(No build-arg. CD already token-replaces `.env.required` → `.env.prod` before Docker.)

---

## A2. Retail Webapp (`fintech_retail_webapp`)

### 1) Library
| Group | Variable | Secret |
|-------|----------|--------|
| `ZB-FintechRetailWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechRetailWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

### 2) File `fintech_retail_webapp/.env.required` — add:
```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```
Commit/push.

### 3) Run **Fintech-Retail-Webapp-CD**

---

## A3. Admin Webapp (`fintech_admin_webapp`) — exact only

### 1) Library variable groups (add variable into both groups)
| Group | Variable | Secret |
|-------|----------|--------|
| `ZB-FintechAdminWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechAdminWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

(CD already uses these groups via `envConfig`. You are only adding the new variable.)

### 2) Optional but recommended — file `.env.required`
Add:
```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```
(Admin CI does not token-replace today; Insights for Admin is via Docker `arguments` below.)

### 3) Edit `Pipelines/pipeline_CI.yml` — replace the whole file content with this

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
    imageName: 'fintechadminwebapp'
    dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
    arguments: '--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)'
```

**What changed vs today (only 2 things):**
1. Variable groups: branch picks **QA or PROD** (not both at once).
2. Parameter `arguments: '--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)'`  
   (this is the real parameter name from the CI template log).

**Do not edit** the shared `pipelines` repo for this.

### 4) Confirm after CI
Rerun **admin-webapp CI** → open log **Build image** → must contain:
```text
--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=
```

### 5) Then run **admin-webapp CD**

---

# PART B — Backends / scripts

For each: add Library var → run CD.  
Variable name always: `APPLICATIONINSIGHTS_CONNECTION_STRING` (Secret).  
No `.env.required`. No build-arg.

| Repo | QA group | PROD group | CD pipeline |
|------|----------|------------|-------------|
| `fintech_business_management` | `ZB-FintechBusinessManagment-QA` | `ZB-FintechBusinessManagment-PROD` | **business-management CD** |
| `fintech_business_settings` | `ZB-FintechBusinessSettings-QA` | `ZB-FintechBusinessSettings-PROD` | **business-settings CD** |
| `fintech_documents_management` | `ZB-FintechDocumentsManagement-QA` | `ZB-FintechDocumentsManagement-PROD` | **Fintech-Documents-Management-CD** |
| `fintech_notifications_management` | `ZB-FintechNotificationsManagement-QA` | `ZB-FintechNotificationsManagement-PROD` | **Fintech-Notifications-Management-CD** |
| `fintech_super_admin` | `ZB-FintechSupeAdmin-QA` | `ZB-FintechSupeAdmin-PROD` | **Fintech-Super-Admin-CD** |
| `fintech_user_management` | `ZB-FintechUserManagment-QA` | `ZB-FintechUserManagment-PROD` | **user-management CD** |
| `fintech_statement_generator` | `ZB-FintechStatementGenerator-QA` | `ZB-FintechStatementGenerator-PROD` | **Fintech-Statement-Generator-CD** |
| `fintech_processing_scripts` | `ZB-FintechProcessingScripts-QA` | `ZB-FintechProcessingScripts-PROD` | **Fintech-Processing-Scripts-CD** |
| `fintech_management_migrations` | `ZB-FintechManagementMigrations-QA` | `ZB-FintechManagementMigrations-PROD` | **Fintech-Migrations-Management-CD** |

---

# Do not use for Insights

- `ZB-Common-QA` / `ZB-Common-PROD`
- `PlatformDetails` (keep it; it is ACR/service connections, not Insights)

---

# Order

1. Add all Library variables (QA first).  
2. Update Webapp + Retail `.env.required` (and Admin optional).  
3. Update Admin `Pipelines/pipeline_CI.yml` as above.  
4. Run Webapp CD, Retail CD, Admin CI → CD.  
5. Run all backend CDs.  
6. Click apps → check App Insights.  
7. Prod later.

---

# Retention

Set in Azure Portal → App Insights / Log Analytics (not in git).
