# Telemetry & Deployment — QA to Production Migration Guide


---

## SECTION 1: QA FIXES SUMMARY (COMPLETED)

### Item 1: `Pipelines/pipeline_CD.yml` (`fintech_statement_generator`)
- Secret mapping `APPLICATIONINSIGHTS_CONNECTION_STRING: $(APPLICATIONINSIGHTS_CONNECTION_STRING)` added under `secretEnv:`.

### Item 2: `.gitignore` and `package-lock.json`
- Completed in PR 2416 (`fintech_admin_webapp`), PR 2417 (`fintech_webapp`), and PR 2421 (`fintech_statement_generator`).

### Item 3: Azure DevOps Library Setup (`ZB-FintechStatementGenerator-QA`)
- Variables added: `APP_ENV=qa`, `NODE_ENV=production`, `NEXT_RUNTIME=nodejs`, `TELEMETRY_COMPONENT=fintech_statement_generator`.

### Item 4: Azure DevOps Pipeline Permissions (`PlatformDetails`)
- Allowed permissions granted for `webapp CI` and `Fintech-WebApp-CD`.

---

## SECTION 2: PRODUCTION MIGRATION BLUEPRINT (3 REPOSITORIES)

Moving **`fintech_webapp`**, **`fintech_admin_webapp`**, and **`fintech_statement_generator`** from QA to Production requires the following 3 steps:

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

1. **GitHub**: Merge tested code from `sandbox_qa` (or `staging`) into **`main`** (Production branch).
2. **Azure DevOps**: PRs automatically sync to **`main`** branch in Azure DevOps.

---

### Step 3: Production Pipeline Trigger & Deployment

1. **Admin Webapp**: Run `admin-webapp CI` on `main` branch -> Trigger `admin-webapp CD` (Production stage).
2. **Corporate Webapp**: Run `Fintech-WebApp-CD` on `main` branch (Production stage).
3. **Statement Generator**: Run `statement-generator CI` on `main` branch -> Trigger `statement-generator CD` (Production stage).
4. Verify Production telemetry in Azure Portal under resource **`insight-zb-purpleplum-prod-eastus`**.
