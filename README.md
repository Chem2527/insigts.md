# Telemetry & Deployment — BEFORE vs AFTER Replacement Guide

**Organization:** `https://dev.azure.com/ZenusBankInternational`  
**Project:** `ZB-CS - PP Digital Portal Solution`  

---

## Item 1: `Pipelines/pipeline_CD.yml` (`fintech_statement_generator`)

**File Path:** `Pipelines/pipeline_CD.yml`

### BEFORE (Current Code):
```yaml
variables:
  - group: PlatformDetails

extends:
  template: pipeline-cd/flow/containerapp-deploy.yml@pipelines
  parameters:
    renderEnv: true
    secretEnv:
      JWT_PASSPHRASE: $(JWT_PASSPHRASE)
      RESPONSE_ENCRYPTION_KEY: $(RESPONSE_ENCRYPTION_KEY)
    envConfig:
```

### AFTER (Replace With):
```yaml
variables:
  - group: PlatformDetails

extends:
  template: pipeline-cd/flow/containerapp-deploy.yml@pipelines
  parameters:
    renderEnv: true
    secretEnv:
      JWT_PASSPHRASE: $(JWT_PASSPHRASE)
      RESPONSE_ENCRYPTION_KEY: $(RESPONSE_ENCRYPTION_KEY)
      APPLICATIONINSIGHTS_CONNECTION_STRING: $(APPLICATIONINSIGHTS_CONNECTION_STRING)
    envConfig:
```

---

## Item 2: `.gitignore` and `package-lock.json` (COMPLETED)

**Status:** Completed in PR 2416 (`fintech_admin_webapp`), PR 2417 (`fintech_webapp`), and PR 2421 (`fintech_statement_generator`).  
No further changes needed for `.gitignore`.

---

## Item 3: Azure DevOps Library Setup (`ZB-FintechStatementGenerator-QA`)

**Location:** Azure DevOps -> Pipelines -> Library -> `ZB-FintechStatementGenerator-QA`

### Add these 4 Variables (Non-Secret):

| Variable Name | Value | Secret (Padlock)? |
|---|---|---|
| **APP_ENV** | `qa` | Off |
| **NODE_ENV** | `production` | Off |
| **NEXT_RUNTIME** | `nodejs` | Off |
| **TELEMETRY_COMPONENT** | `fintech_statement_generator` | Off |

---

## Item 4: Azure DevOps Pipeline Permissions (`PlatformDetails`)

**Location:** Azure DevOps -> Pipelines -> Library -> `PlatformDetails`

### BEFORE:
Allowed pipelines list has `admin-webapp CI` & `admin-webapp CD`, but `webapp CI` and `Fintech-WebApp-CD` (or `webapp CD`) are missing.

### AFTER:
1. Click **Pipeline permissions**.
2. Click **+** (Add Pipeline).
3. Select **`webapp CI`** AND **`Fintech-WebApp-CD`** (or `webapp CD`) and click **Add / Save**.
