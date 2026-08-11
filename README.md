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

### Item 4: React Webapps (`fintech_admin_webapp` & `fintech_webapp`) CI & Dockerfile Updates — REQUIRED

> [!IMPORTANT]
> **Why keeping values in Azure DevOps Variable Groups alone will NOT work for React Webapps:**
> 1. **React compiles at Build Time (CI):** React is a Single Page Application (SPA). During `npm run build` inside Docker, Webpack scans for `process.env.REACT_APP_*` and **permanently bakes the values into static `.js` files**.
> 2. **Azure DevOps variables aren't auto-passed into Docker:** Storing a variable in Azure DevOps does not automatically pass it to `docker build` unless explicitly declared as a `--build-arg` in `pipeline_CI.yml`.
> 3. **CD Runtime variables (`secretEnv`) do NOT affect React:** `pipeline_CD.yml` sets container environment variables at runtime for Nginx. Since Nginx only serves pre-compiled static JS files, runtime CD variables have **zero effect** on compiled React code.

#### Required Changes for `fintech_admin_webapp` & `fintech_webapp`:

##### 1. `Dockerfile` — COMPLETED 
`Dockerfile` already includes `ARG`, `ENV`, and writes `REACT_APP_APPINSIGHTS_CONNECTION_STRING` to `.env.prod`.

##### 2. `Pipelines/pipeline_CI.yml` — PENDING 
Add `arguments:` parameter to pass `--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)` and link `ZB-FintechAdminWebApp-QA`:

```yaml
variables:
  - group: PlatformDetails
  - ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
    - group: ZB-FintechAdminWebApp-PROD
  - ${{ if ne(variables['Build.SourceBranch'], 'refs/heads/qa') }}:
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
