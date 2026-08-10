# Zenus App Insights — Action Plan & Options (9 Target Repositories)

**Organization:** `https://dev.azure.com/ZenusBankInternational`  
**Project:** `ZB-CS - PP Digital Portal Solution`  
**Target Repositories:**
1. `fintech_user_management`
2. `fintech_super_admin`
3. `fintech_business_management`
4. `fintech_business_settings`
5. `fintech_notifications_management`
6. `fintech_statement_generator`
7. `fintech_management_migrations`
8. `fintech_webapp`
9. `fintech_admin_webapp`

---

## 📌 Branch Mapping Strategy

| Platform | Source Branch | Target Branch | Description |
|---|---|---|---|
| **GitHub** | `telemetry-05` (or feature branch) | **`staging`** | Commit file changes on `staging` branch (or PR into `staging`). |
| **Azure DevOps** | GitHub **`staging`** | **`qa`** | Merge / Push GitHub `staging` into Azure DevOps **`qa`** branch (where CD/CI builds run for QA). |

---

## 💡 Two Implementation Options for Admin Webapp

Rayhan's update note states:
> *"The pipeline assumes webapps read environment variables from the container at runtime. If a webapp still requires build-time injection, you can add a Docker --build-arg as a short-term workaround like before. But that falls outside the standard and should be corrected later."*

### 🌟 OPTION 1 (RECOMMENDED — No `pipeline_CI.yml` Edit Needed)
**Standard Runtime Token Replacement for BOTH Webapps (`fintech_webapp` & `fintech_admin_webapp`)**

Under Option 1, you **DO NOT** edit `pipeline_CI.yml` at all! Both webapps use `.env.required` for runtime token replacement during CD.

#### File Changes for Option 1:
1. **`fintech_webapp`**: Add 1 line to `.env.required`:
   ```env
   REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
   ```
2. **`fintech_admin_webapp`**: Add `.env.required` with runtime token replacement:
   ```env
   REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
   ```
3. **`Dockerfile` (Both Webapps)**: Update `Dockerfile` in both frontend repos.
4. **`pipeline_CI.yml`**: **NO EDIT NEEDED**.

---

### OPTION 2 (Short-Term Workaround — Requires Editing `pipeline_CI.yml`)
**Build-Time Docker Argument Injection for Admin Webapp**

Under Option 2, `fintech_admin_webapp` uses Docker `--build-arg` at image compile time.

#### File Changes for Option 2:
1. **`fintech_webapp`**: Add 1 line to `.env.required`.
2. **`fintech_admin_webapp/Pipelines/pipeline_CI.yml`**: Add `--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=$(REACT_APP_APPINSIGHTS_CONNECTION_STRING)` to `arguments:`.
3. **`fintech_admin_webapp/Dockerfile`**: Add `ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING` block.

---

## 📋 Summary Table Across All 9 Repositories (Option 1 vs Option 2)

| # | Repository | Option 1 (Standard - No CI YAML Edit) | Option 2 (Short-Term CI Build-Arg) |
|---|---|---|---|
| **1** | `fintech_user_management` | No file changes (Run CD) | No file changes (Run CD) |
| **2** | `fintech_super_admin` | No file changes (Run CD) | No file changes (Run CD) |
| **3** | `fintech_business_management` | No file changes (Run CD) | No file changes (Run CD) |
| **4** | `fintech_business_settings` | No file changes (Run CD) | No file changes (Run CD) |
| **5** | `fintech_notifications_management` | No file changes (Run CD) | No file changes (Run CD) |
| **6** | `fintech_statement_generator` | No file changes (Run CD) | No file changes (Run CD) |
| **7** | `fintech_management_migrations` | No file changes (Run CD) | No file changes (Run CD) |
| **8** | `fintech_webapp` | Add `.env.required` line + update `Dockerfile` | Add `.env.required` line + update `Dockerfile` |
| **9** | `fintech_admin_webapp` | Add `.env.required` line + update `Dockerfile` (**No CI YAML Edit**) | Update `Dockerfile` + Edit `pipeline_CI.yml` |

---

## 🚀 Execution Steps in Azure DevOps (After Syncing `staging` ➔ `qa`)

1. **Frontends**: Run **Fintech-WebApp-CD** and **admin-webapp CD** (or CI ➔ CD).
2. **Backends**: Run the 7 backend CD pipelines (`user-management CD`, `super-admin CD`, `business-management CD`, `business-settings CD`, `notifications-management CD`, `statement-generator CD`, `migrations CD`).
