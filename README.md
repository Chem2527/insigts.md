# Zenus App Insights — Final Confirmation & Execution Checklist (9 Target Repositories)

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

## 🎯 Final Confirmation: YOU ARE 100% RIGHT!

### 1. `.env.required` Update
- **For these 2 Frontend Webapps ONLY**: `fintech_webapp` and `fintech_admin_webapp`.
  Add/Ensure this line is present:
  ```env
  REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
  ```
- **For the 7 Backend Repos**: **No update needed** (Backend `.env.required` files already contain `APPLICATIONINSIGHTS_CONNECTION_STRING=` on `staging`).

---

### 2. `Dockerfile` Update
- Update `Dockerfile` in **`fintech_webapp`** and **`fintech_admin_webapp`** by adding the App Insights block:
  ```dockerfile
  # Application Insights connection string (build-time)
  ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING
  ENV REACT_APP_APPINSIGHTS_CONNECTION_STRING=$REACT_APP_APPINSIGHTS_CONNECTION_STRING
  RUN echo "REACT_APP_APPINSIGHTS_CONNECTION_STRING=${REACT_APP_APPINSIGHTS_CONNECTION_STRING}" >> .env.prod
  ```
- **For the 7 Backend Repos**: **No update needed** (Backend Dockerfiles run Node.js and read environment variables dynamically at container launch).

---

## 🚀 Final Simple 3-Step Execution Plan

### Step 1: GitHub (`staging` branch)
Update `Dockerfile` and `.env.required` in:
- `fintech_webapp`
- `fintech_admin_webapp`

*(No code changes needed for the 7 backend repos).*

### Step 2: Sync to Azure DevOps (`qa` branch)
Push or merge GitHub **`staging`** ➔ Azure DevOps **`qa`** branch.

### Step 3: Trigger Pipelines in Azure DevOps
Run the CI/CD pipelines in Azure DevOps:
1. **Frontend CDs**: Run `Fintech-WebApp-CD` and `admin-webapp CD`.
2. **Backend CDs**: Run the 7 backend CD pipelines (`user-management CD`, `super-admin CD`, `business-management CD`, `business-settings CD`, `notifications-management CD`, `statement-generator CD`, `migrations CD`).

---

## ✅ Summary Table

| # | Repository Name | Update Dockerfile? | Update `.env.required`? | Post-Sync Action |
|---|---|---|---|---|
| **1** | `fintech_user_management` | No | No | Run **user-management CD** |
| **2** | `fintech_super_admin` | No | No | Run **Fintech-Super-Admin-CD** |
| **3** | `fintech_business_management` | No | No | Run **business-management CD** |
| **4** | `fintech_business_settings` | No | No | Run **business-settings CD** |
| **5** | `fintech_notifications_management` | No | No | Run **Fintech-Notifications-Management-CD** |
| **6** | `fintech_statement_generator` | No | No | Run **Fintech-Statement-Generator-CD** |
| **7** | `fintech_management_migrations` | No | No | Run **Fintech-Migrations-Management-CD** |
| **8** | `fintech_webapp` | **YES** | **YES** | Sync ➔ Run **Fintech-WebApp-CD** |
| **9** | `fintech_admin_webapp` | **YES** | **YES** | Sync ➔ Run **admin-webapp CD** |
