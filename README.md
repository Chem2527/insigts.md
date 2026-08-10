# Zenus Telemetry & Azure DevOps Deployment Guide (Final Checklist)

**Organization:** `https://dev.azure.com/ZenusBankInternational`  
**Project:** `ZB-CS - PP Digital Portal Solution`  

---

## 🛠️ Step 1: Azure DevOps Pipeline Permissions Fix (`webapp CI`)

To unblock **`webapp CI`** from waiting in queue:

1. Go to **Pipelines** ➔ **Library** in Azure DevOps.
2. Click on the variable group **`PlatformDetails`**.
3. At the top right, click **Pipeline permissions**.
4. Click the **`+`** icon at the top right of the popup modal.
5. Search for **`webapp CI`**, select it, and click **Add / Save**.

---

## 🛠️ Step 2: Statement Generator Variable Group Setup (`ZB-FintechStatementGenerator-QA`)

In Azure DevOps Library, open **`ZB-FintechStatementGenerator-QA`** and add the following 4 variables:

| Variable Name | Value | Description |
|---|---|---|
| **`APP_ENV`** | `qa` | Environment target |
| **`NODE_ENV`** | `production` | Node.js production server optimization mode |
| **`NEXT_RUNTIME`** | `nodejs` | Next.js server runtime setting |
| **`TELEMETRY_COMPONENT`** | `fintech_statement_generator` | Component name for App Insights telemetry logs |

---

## 🛠️ Step 3: `Pipelines/pipeline_CD.yml` Secret Mapping (`fintech_statement_generator`)

In `fintech_statement_generator` repository, ensure `Pipelines/pipeline_CD.yml` maps `APPLICATIONINSIGHTS_CONNECTION_STRING` under `secretEnv:`:

```yaml
    secretEnv:
      JWT_PASSPHRASE: $(JWT_PASSPHRASE)
      RESPONSE_ENCRYPTION_KEY: $(RESPONSE_ENCRYPTION_KEY)
      APPLICATIONINSIGHTS_CONNECTION_STRING: $(APPLICATIONINSIGHTS_CONNECTION_STRING)
```

---

## 📁 Summary Checklist for README Update

### Frontend Webapps (`fintech_webapp` & `fintech_admin_webapp`)
- [x] `.env.required` updated with `REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__`.
- [x] `Dockerfile` updated with `ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING`.
- [x] `package-lock.json` un-ignored in `.gitignore` and committed to Git.

### Backend Microservices (`fintech_statement_generator` & 6 others)
- [x] `.env.required` updated with `APPLICATIONINSIGHTS_CONNECTION_STRING=`.
- [x] `instrumentation.ts` added in `src/`.
- [x] `package-lock.json` un-ignored in `.gitignore` and committed to Git.
- [x] Variable group `ZB-FintechStatementGenerator-QA` updated with runtime variables.
