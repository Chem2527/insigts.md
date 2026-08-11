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
