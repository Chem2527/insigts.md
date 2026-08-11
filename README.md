# Telemetry & Deployment — QA Status & Production Migration Guide

**Organization:** `https://dev.azure.com/ZenusBankInternational`  
**Project:** `ZB-CS - PP Digital Portal Solution`  

---

## SECTION 1: QA STATUS & ONLY PENDING ACTION ITEM

### ❌ ONLY PENDING QA ACTION ITEM: Unlock Variable Padlock in Azure DevOps Library & Re-run CI

> [!CAUTION]
> **Live Checked Issue (`isSecret: True`):**
> Live check of Azure DevOps API confirms that `REACT_APP_APPINSIGHTS_CONNECTION_STRING` in `ZB-FintechAdminWebApp-QA` and `ZB-FintechWebApp-QA` is currently **LOCKED as a Secret 🔒 (`isSecret: True`)**.
> 
> Because it is locked as a Secret, Azure DevOps **refuses to expand `$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)`** inside pipeline template arguments, causing Docker to compile the literal string `"$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)"` into JavaScript, which throws `Error: Please provide instrumentation key`.

#### 2-Step Fix:

1. **Unlock Variable in Azure DevOps**:
   * Go to Azure DevOps -> **Pipelines** -> **Library**.
   * Open **`ZB-FintechAdminWebApp-QA`** (and `ZB-FintechWebApp-QA`).
   * Click the **Padlock icon 🔒** next to `REACT_APP_APPINSIGHTS_CONNECTION_STRING` to **Unlock it (Make Non-Secret / isSecret: False)**.
   * Click **Save**. *(Frontend App Insights connection strings are public keys embedded in browser JS files anyway).*

2. **Re-run Pipelines**:
   * Go to Pipelines -> **`admin-webapp CI`** -> Click **Run pipeline** (on `qa` branch).
   * Go to Pipelines -> **`webapp CI`** -> Click **Run pipeline** (on `qa` branch).

---

### ✅ COMPLETED QA ITEMS SUMMARY
- `statement-generator` CD pipeline updated & deployed (`APPLICATIONINSIGHTS_CONNECTION_STRING` mapped) ✅
- `ZB-FintechStatementGenerator-QA` Library setup (`APP_ENV`, `NODE_ENV`, `NEXT_RUNTIME`, `TELEMETRY_COMPONENT`) ✅
- `PlatformDetails` Pipeline permissions authorized for `webapp CI` & `CD` ✅
- `Dockerfile` updated with `ARG` & `ENV` for `fintech_admin_webapp` & `fintech_webapp` ✅
- `pipeline_CI.yml` updated with `--build-arg` for `fintech_admin_webapp` & `fintech_webapp` ✅

---

## SECTION 2: PRODUCTION MIGRATION BLUEPRINT (3 REPOSITORIES)

Moving `fintech_webapp`, `fintech_admin_webapp`, and `fintech_statement_generator` from QA to Production requires the following 3 steps:

### Step 1: Azure DevOps Production Variable Groups

In Azure DevOps -> Pipelines -> Library, verify/add the Production App Insights connection strings:

#### 1. `ZB-FintechWebApp-PROD`
- Add Secret Variable: `REACT_APP_APPINSIGHTS_CONNECTION_STRING` = `[PROD App Insights Connection String]`

#### 2. `ZB-FintechAdminWebApp-PROD`
- Add Secret Variable: `REACT_APP_APPINSIGHTS_CONNECTION_STRING` = `[PROD App Insights Connection String]`

#### 3. `ZB-FintechStatementGenerator-PROD`
- Add Secret Variable: `APPLICATIONINSIGHTS_CONNECTION_STRING` = `[PROD App Insights Connection String]`
- Add Non-Secret Variables:
  - `APP_ENV` = `prod`
  - `NODE_ENV` = `production`
  - `NEXT_RUNTIME` = `nodejs`
  - `TELEMETRY_COMPONENT` = `fintech_statement_generator`

---

### Step 2: Git Code Merges

1. **GitHub**: Merge tested code from `sandbox_qa` (or `staging`) into `main` (Production branch).
2. **Azure DevOps**: PRs automatically sync to `main` branch in Azure DevOps.

---

### Step 3: Production Pipeline Trigger & Deployment

1. **Admin Webapp**: Run `admin-webapp CI` on `main` branch -> Trigger `admin-webapp CD` (Production stage).
2. **Corporate Webapp**: Run `Fintech-WebApp-CD` on `main` branch (Production stage).
3. **Statement Generator**: Run `statement-generator CI` on `main` branch -> Trigger `statement-generator CD` (Production stage).
4. Verify Production telemetry in Azure Portal under resource `insight-zb-purpleplum-prod-eastus`.
