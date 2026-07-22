# Zenus App Insights — exact setup checklist


# PART A — Frontends

## A1. Corporate Webapp (`fintech_webapp`) — exact

### Step 1 — Variable groups
Pipelines → Library → open each group → **+ Add** → Save

| Group | Name | Value | Secret |
|-------|------|-------|--------|
| `ZB-FintechWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | your Insights string | Yes |
| `ZB-FintechWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | your Insights string | Yes |

### Step 2 — Repo file
File: `fintech_webapp/.env.required`  
Add this line (same style as existing lines):

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

Commit + push to the branch CD builds (`develop` / `main`).

### Step 3 — Run pipeline
Run **Fintech-WebApp-CD**

No `--build-arg`. CD already does `replacetokens` on `.env.required` → `.env.prod` → Docker build.

---

## A2. Retail Webapp (`fintech_retail_webapp`) — exact

### Step 1 — Variable groups

| Group | Name | Secret |
|-------|------|--------|
| `ZB-FintechRetailWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechRetailWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

### Step 2 — Repo file
File: `fintech_retail_webapp/.env.required`

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

Commit + push.

### Step 3 — Run pipeline
Run **Fintech-Retail-Webapp-CD**

No `--build-arg`.

---

## A3. Admin Webapp (`fintech_admin_webapp`) — exact

### Step 1 — Variable groups (Library only)

| Group | Name | Secret |
|-------|------|--------|
| `ZB-FintechAdminWebApp-QA` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |
| `ZB-FintechAdminWebApp-PROD` | `REACT_APP_APPINSIGHTS_CONNECTION_STRING` | Yes |

(CD already references these two groups. You are only **adding the new variable** into them.)

### Step 2 — Repo file `.env.required`
File: `fintech_admin_webapp/.env.required`

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

Commit + push to `qa` / `main` as needed.  
(This matches Webapp format; Admin CI does **not** currently run token-replace, so this alone does **not** bake Insights.)

### Step 3 — CI must pass Docker build-arg (required)

**Verified:** current Admin CI `docker build` has **zero** `--build-arg`.  
**Verified:** Dockerfile expects `ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING`.  
**Blocked for us:** cannot open `pipelines` / `build-acr.yml` to name the exact parameter.

#### What you must do (exact)

1. Open the shared repo used by Admin CI:  
   `ZB-CS - PP Digital Portal Solution/pipelines`  
   (ask DevOps if you cannot see it — it is referenced by CI but not in the normal repo list for this PAT).

2. Open: `pipeline-ci/flow/build-acr.yml`  
   Find the **Docker@2** task named like **Build image** (same name as in CI logs).

3. On that Docker Build task, set **arguments** so the real command becomes:
   ```text
   docker build ... --build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=$(REACT_APP_APPINSIGHTS_CONNECTION_STRING) ...
   ```
   In YAML this is the Docker@2 input:
   ```yaml
   arguments: --build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)
   ```
   (Keep any existing arguments if the template already sets some; **append** this build-arg.)

4. In `fintech_admin_webapp/Pipelines/pipeline_CI.yml`, make the Admin variable group available during CI (so `$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)` resolves).

   Replace:
   ```yaml
   variables:
   - group: PlatformDetails
   ```
   With:
   ```yaml
   variables:
   - group: PlatformDetails
   - ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
     - group: ZB-FintechAdminWebApp-PROD
   - ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/qa') }}:
     - group: ZB-FintechAdminWebApp-QA
   ```

   **Do not** put QA and PROD groups both always-on.

5. How to confirm it worked: rerun **admin-webapp CI**, open log **Build image**, search for:
   ```text
   --build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=
   ```
   If that string is missing, Insights is still empty in the image.

### Step 4 — Pipelines to run
1. **admin-webapp CI** (must succeed with build-arg in log)  
2. **admin-webapp CD**

---

# PART B — Backends / scripts — exact

For each row:

1. Library → QA group → Add `APPLICATIONINSIGHTS_CONNECTION_STRING` (Secret) → Save  
2. Same for PROD group  
3. Run the CD pipeline  

No `.env.required`. No build-arg. No Dockerfile change.

| Repo | QA group | PROD group | Run this CD |
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
- `PlatformDetails`  

Those are ACR / service connections only.

---

# Order

1. Add Library variables (all QA groups first).  
2. Update Webapp + Retail + Admin `.env.required`.  
3. Fix Admin shared `pipelines` Docker@2 build-arg + Admin CI variable groups.  
4. Run Webapp CD, Retail CD, Admin CI → CD.  
5. Run all backend CDs.  
6. Click apps → App Insights.  
7. Repeat PROD.

---

# Verify

| App type | Pass criteria |
|----------|----------------|
| Webapp / Retail | After CD, Insights shows traffic after UI clicks |
| Admin | CI **Build image** log contains `--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=`; then Insights after CD + UI |
| Backend | After CD, API usage shows structured logs / traces |

Retention: set in Azure Portal App Insights / Log Analytics (not in these repos).
