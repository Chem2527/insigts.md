# Zenus / Purple Plum — Azure App Insights setup guide

Exact steps for all repos. Read-only verification was done against Azure DevOps project  
`ZB-CS - PP Digital Portal Solution` (nothing was modified in ADO by this guide).

**Use the same App Insights connection string value everywhere.**  
Only the **variable name** differs (FE vs BE).

---

## Admin CI — QA vs PROD variable groups (confirmed)

### Do you add both groups?

**No. Do not add both at the same time.**

| Branch being built | Use this group only |
|--------------------|---------------------|
| `qa` | `ZB-FintechAdminWebApp-QA` |
| `main` (prod) | `ZB-FintechAdminWebApp-PROD` |

Adding **both** QA + PROD together is wrong — same variable name exists in both groups and values can conflict.

### Recommended YAML pattern (branch-based, one group)

In `fintech_admin_webapp` → `Pipelines/pipeline_CI.yml`:

```yaml
variables:
- group: PlatformDetails
- ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
  - group: ZB-FintechAdminWebApp-PROD
- ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/qa') }}:
  - group: ZB-FintechAdminWebApp-QA
```

### Is adding only `- group: ZB-FintechAdminWebApp-QA` enough?

**No — not by itself.**

What was verified (read-only):

| Check | Result |
|-------|--------|
| Current Admin CI variables | Only `PlatformDetails` |
| Admin CI `.env.required` token replace before Docker? | **Not present** in `Pipelines/pipeline_CI.yml` |
| Shared repo `.../pipelines` (`build-acr.yml`) | **Not found** in project repo list — cannot confirm template does `.env` replace |
| Admin Dockerfile | Expects **`ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING`** at image build |

So for Admin you must:

1. Add the Insights variable to **QA and PROD variable groups** (Library), and  
2. Wire CI so the value is available at **docker build** via **build-arg** (see Admin section below).

Webapp / Retail already create `.env.prod` from `.env.required` in **CD** before Docker build — Admin does not (in the visible CI YAML).

---

## PART A — Frontends

### A1. Corporate Webapp (`fintech_webapp`)

#### Variable groups (Pipelines → Library)

| Group | Variable name | Secret |
|-------|---------------|--------|
| `ZB-FintechWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

#### Repo file

File: `.env.required`  
Add:

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

Commit/push to the branch CD builds from (`develop` / `main`).

#### Pipeline to run

- **Fintech-WebApp-CD**

No `--build-arg` needed (CD already: `replacetokens` on `.env.required` → `.env.prod` → Docker build).

---

### A2. Retail Webapp (`fintech_retail_webapp`)

#### Variable groups

| Group | Variable name | Secret |
|-------|---------------|--------|
| `ZB-FintechRetailWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechRetailWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

#### Repo file

File: `.env.required`  
Add:

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

#### Pipeline to run

- **Fintech-Retail-Webapp-CD**

No `--build-arg` needed.

---

### A3. Admin Webapp (`fintech_admin_webapp`)

#### Variable groups

| Group | Variable name | Secret |
|-------|---------------|--------|
| `ZB-FintechAdminWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechAdminWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

#### Repo file

File: `.env.required`  
Add (for consistency / future):

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

#### Pipeline file to change

File: `Pipelines/pipeline_CI.yml`

**1) Link the correct Admin group by branch** (not both at once):

```yaml
variables:
- group: PlatformDetails
- ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
  - group: ZB-FintechAdminWebApp-PROD
- ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/qa') }}:
  - group: ZB-FintechAdminWebApp-QA
```

**2) Pass Docker build-arg** (required for Admin today)

Your Dockerfile already has:

```dockerfile
ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING
```

So CI must pass it. Exact parameter name depends on shared template `pipeline-ci/flow/build-acr.yml@pipelines`  
(that `pipelines` repo was **not listed** in the project — ask team where it lives, or open a recent Admin CI log and search for `docker build`).

Typical pattern if the template supports extra build args:

```yaml
extends:
  template: pipeline-ci/flow/build-acr.yml@pipelines
  parameters:
    buildMode: docker
    imageName: 'fintechadminwebapp'
    dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
    # confirm exact parameter name in build-acr.yml
    extraBuildArguments: '--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)'
```

If the template has **no** build-arg parameter, add a CI step before Docker build (same idea as Webapp CD):

```yaml
# conceptual steps — place where your CI builds the image
- task: replacetokens@6
  inputs:
    sources: '.env.required'
    tokenPattern: 'doubleunderscores'
- script: mv .env.required .env.prod
```

…and ensure `npm run build` inside Docker uses that `.env.prod` (your Dockerfile also appends from ARG).

#### Pipelines to run

1. **admin-webapp CI** (rebuild image — mandatory)  
2. **admin-webapp CD** (deploy)

---

## PART B — Backends / scripts (no build-arg)

For every service below:

1. Library → open **QA** group → Add  
   `APPLICATIONINSIGHTS_CONNECTION_STRING` = Insights string → **Secret** → Save  
2. Repeat for **PROD** group  
3. Rerun that service’s **CD** pipeline  

No `.env.required` change. No `--build-arg`.

| # | Repo | QA variable group | PROD variable group | CD pipeline to run |
|---|------|-------------------|---------------------|--------------------|
| 1 | `fintech_business_management` | `ZB-FintechBusinessManagment-QA` | `ZB-FintechBusinessManagment-PROD` | **business-management CD** |
| 2 | `fintech_business_settings` | `ZB-FintechBusinessSettings-QA` | `ZB-FintechBusinessSettings-PROD` | **business-settings CD** |
| 3 | `fintech_documents_management` | `ZB-FintechDocumentsManagement-QA` | `ZB-FintechDocumentsManagement-PROD` | **Fintech-Documents-Management-CD** |
| 4 | `fintech_notifications_management` | `ZB-FintechNotificationsManagement-QA` | `ZB-FintechNotificationsManagement-PROD` | **Fintech-Notifications-Management-CD** |
| 5 | `fintech_super_admin` | `ZB-FintechSupeAdmin-QA` | `ZB-FintechSupeAdmin-PROD` | **Fintech-Super-Admin-CD** |
| 6 | `fintech_user_management` | `ZB-FintechUserManagment-QA` | `ZB-FintechUserManagment-PROD` | **user-management CD** |
| 7 | `fintech_statement_generator` | `ZB-FintechStatementGenerator-QA` | `ZB-FintechStatementGenerator-PROD` | **Fintech-Statement-Generator-CD** |
| 8 | `fintech_processing_scripts` | `ZB-FintechProcessingScripts-QA` | `ZB-FintechProcessingScripts-PROD` | **Fintech-Processing-Scripts-CD** |
| 9 | `fintech_management_migrations` | `ZB-FintechManagementMigrations-QA` | `ZB-FintechManagementMigrations-PROD` | **Fintech-Migrations-Management-CD** |

Note: group names keep existing typos (`Managment`, `SupeAdmin`) — use exact names as in Library.

---

## PART C — Do NOT use for Insights

| Group | Why |
|-------|-----|
| `ZB-Common-QA` / `ZB-Common-PROD` | ACR / resource group / service connections only |
| `PlatformDetails` | ACR / deploy service connections only |

---

## PART D — Suggested order

1. Add all Library variables (QA first).  
2. Update FE `.env.required` (Webapp, Retail, Admin).  
3. Fix Admin CI (branch group + build-arg or token replace).  
4. Run Webapp CD, Retail CD, Admin CI → Admin CD.  
5. Run all backend CDs.  
6. Click around apps → check App Insights (Transaction search / Logs).  
7. Repeat PROD when QA is good.

---

## PART E — How to verify in App Insights

After deploy + some UI navigation / API calls, look for:

- `component` / `logger_name`: e.g. `PP-Corporate-Webapp`, `PP-Admin-Webapp`, `PP-Business-Management`
- `operationId` / `trace_id`
- `event`: `RestRequestProcessed`, `InboundRestRequestProcessed`
- FE page views / exceptions if App Insights SDK is initialized

---

## Quick answers

| Question | Answer |
|----------|--------|
| Add only `ZB-FintechAdminWebApp-QA`? | For **qa** builds only — yes (plus build-arg / env bake). |
| Also add `ZB-FintechAdminWebApp-PROD` in the same `variables:` list always? | **No.** Use branch condition; one group per run. |
| Is linking the Admin group enough with no other CI change? | **No.** Admin CI must pass the value into Docker build. |
| Webapp / Retail need `--build-arg`? | **No.** Use variable group + `.env.required` + CD. |
| Backends need `--build-arg`? | **No.** Variable group + CD only. |

---

## Retention (not code)

Set retention in Azure Portal → App Insights / Log Analytics workspace  
(e.g. 90 days). Document in Confluence for the ticket AC.
