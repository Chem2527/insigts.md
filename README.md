# Telemetry & Deployment — QA Status & Production Migration Guide


---

## SECTION 1: QA STATUS & PENDING ACTION ITEMS

### Item 1: `Pipelines/pipeline_CD.yml` (`fintech_statement_generator`) — PENDING
File Path: `Pipelines/pipeline_CD.yml`  
*(Note: This single file change automatically supports BOTH QA and Production deployments)*

#### BEFORE (Current Code):
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

#### AFTER (Replace With):
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


### Item 2: Azure DevOps Library Setup (`ZB-FintechStatementGenerator-QA`) — PENDING
Location: Azure DevOps -> Pipelines -> Library -> `ZB-FintechStatementGenerator-QA`

Add these 4 Non-Secret Variables:

| Variable Name | Value | Secret (Padlock)? |
|---|---|---|
| **APP_ENV** | `qa` | Off |
| **NODE_ENV** | `production` | Off |
| **NEXT_RUNTIME** | `nodejs` | Off |
| **TELEMETRY_COMPONENT** | `fintech_statement_generator` | Off |

---

### Item 3: Azure DevOps Pipeline Permissions (`PlatformDetails`) — PENDING
Location: Azure DevOps -> Pipelines -> Library -> `PlatformDetails` -> Pipeline permissions

Add these 2 pipelines to the allowed list:
1. `webapp CI`
2. `Fintech-WebApp-CD` (or `webapp CD`)

---

## SECTION 2: PRODUCTION 

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

*(Note: `Pipelines/pipeline_CD.yml` updated in Item 1 automatically handles PROD once merged into `main`)*

---
