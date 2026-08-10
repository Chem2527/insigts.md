# Zenus App Insights — Action Plan & Sync Guide (9 Target Repositories)

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

## Executive Summary & Current Audit Status

- **Azure DevOps Library Variable Groups (QA)**: ✅ **COMPLETED**. Rayhan has added secret connection strings (`APPLICATIONINSIGHTS_CONNECTION_STRING` for backends, `REACT_APP_APPINSIGHTS_CONNECTION_STRING` for frontends) to all 9 QA groups (`ZB-Fintech*-QA`).
- **7 Backend Repositories**: ✅ **COMPLETED IN CODE**. Backends read telemetry connection strings dynamically at runtime. No Dockerfile or code changes are required for backends.
- **2 Frontend Webapps**: ⚠️ **ACTION REQUIRED**. Docker image builds in Azure DevOps use standard `Dockerfile` (not `Dockerfile.dev`). The `ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING` lines present in `Dockerfile.dev` must be copied to `Dockerfile`.

---

## PART 1 — GitHub Side (Pre-Sync Code Updates)

Before syncing GitHub `staging` branches to Azure DevOps `qa` / `main`, make the following file updates:

### A. Corporate Webapp (`fintech_webapp`)

#### 1. Update `Dockerfile`
Replace/update `Dockerfile` to accept the build argument:

```dockerfile
# Step 1: Use Node.js as the base image
FROM node:22-alpine AS build

# Set working directory in the container
WORKDIR /app

# Copy package manifest files (lockfile optional)
COPY package*.json ./

# Use npm ci when lockfile exists; fallback to npm install otherwise
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi

# Copy the rest of the application code
COPY . .

# Application Insights connection string (build-time)
ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING
ENV REACT_APP_APPINSIGHTS_CONNECTION_STRING=$REACT_APP_APPINSIGHTS_CONNECTION_STRING
RUN echo "REACT_APP_APPINSIGHTS_CONNECTION_STRING=${REACT_APP_APPINSIGHTS_CONNECTION_STRING}" >> .env.prod

# Build the application
RUN npm run build

# Step 2: Use Nginx to serve the app
FROM nginx:1.28.0-alpine-slim

# Set default environment to "prod", can be overridden at build time
ARG ENVIRONMENT=prod
ENV ENVIRONMENT=${ENVIRONMENT}

# Copy the build files to Nginx's HTML folder
COPY --from=build /app/build /usr/share/nginx/html

# Copy the Nginx configuration based on the environment
COPY nginx.${ENVIRONMENT}.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

#### 2. Update `.env.required`
Append 1 line to `.env.required`:

```env
REACT_APP_APPINSIGHTS_CONNECTION_STRING=__REACT_APP_APPINSIGHTS_CONNECTION_STRING__
```

---

### B. Admin Webapp (`fintech_admin_webapp`)

#### 1. Update `Dockerfile`
Replace/update `Dockerfile` to accept the build argument:

```dockerfile
# Step 1: Use Node.js as the base image
FROM node:22-alpine AS build

# Set working directory in the container
WORKDIR /app

# Copy package manifest(s)
COPY package*.json ./

# Use npm ci when lockfile exists; fall back to npm install otherwise.
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi

# Copy the rest of the application code
COPY . .

# Application Insights connection string (build-time)
ARG REACT_APP_APPINSIGHTS_CONNECTION_STRING
ENV REACT_APP_APPINSIGHTS_CONNECTION_STRING=$REACT_APP_APPINSIGHTS_CONNECTION_STRING
RUN echo "REACT_APP_APPINSIGHTS_CONNECTION_STRING=${REACT_APP_APPINSIGHTS_CONNECTION_STRING}" >> .env.prod

# Build the application
RUN npm run build

# Step 2: Use Nginx to serve the app
FROM nginx:1.28.0-alpine-slim

# For Docker HEALTHCHECK (slim image is minimal)
RUN apk add --no-cache wget

# Set default environment to "prod", can be overridden at build time
ARG ENVIRONMENT=prod
ENV ENVIRONMENT=${ENVIRONMENT}

# Copy the build files to Nginx's HTML folder
COPY --from=build /app/build /usr/share/nginx/html

# Copy the Nginx configuration based on the environment
COPY nginx.${ENVIRONMENT}.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

HEALTHCHECK --interval=15s --timeout=5s --start-period=15s --retries=5 \
  CMD wget -qO- http://127.0.0.1/ >/dev/null 2>&1 || exit 1

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

#### 2. Update `Pipelines/pipeline_CI.yml`
Update `Pipelines/pipeline_CI.yml` to pass the build argument during CI:

```yaml
trigger:
  branches:
    include:
      - main
      - qa

resources:
  repositories:
    - repository: pipelines
      type: git
      name: ZB-CS - PP Digital Portal Solution/pipelines
      ref: refs/heads/main

variables:
  - group: PlatformDetails
  - ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
    - group: ZB-FintechAdminWebApp-PROD
  - ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/qa') }}:
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

### C. 7 Backend Repositories
- **No file changes needed**. All 7 backend repos (`user_management`, `super_admin`, `business_management`, `business_settings`, `notifications_management`, `statement_generator`, `management_migrations`) are 100% ready.

---

## PART 2 — Azure DevOps Side (Sync & Deployment Steps)

### Step 1: Sync GitHub `staging` to Azure DevOps Repositories
Merge or push the updated `staging` branches from GitHub into Azure DevOps `qa` / `main` branches.

### Step 2: Execute Pipelines in Azure DevOps
Run the pipelines in the following order:

1. **Trigger Admin CI**:
   - Run **admin-webapp CI** (`fintech_admin_webapp`). Confirm build log shows `--build-arg REACT_APP_APPINSIGHTS_CONNECTION_STRING=...`.
2. **Trigger Frontend CD Pipelines**:
   - Run **Fintech-WebApp-CD** (`fintech_webapp`).
   - Run **admin-webapp CD** (`fintech_admin_webapp`).
3. **Trigger Backend CD Pipelines**:
   - Run **user-management CD** (`fintech_user_management`)
   - Run **Fintech-Super-Admin-CD** (`fintech_super_admin`)
   - Run **business-management CD** (`fintech_business_management`)
   - Run **business-settings CD** (`fintech_business_settings`)
   - Run **Fintech-Notifications-Management-CD** (`fintech_notifications_management`)
   - Run **Fintech-Statement-Generator-CD** (`fintech_statement_generator`)
   - Run **Fintech-Migrations-Management-CD** (`fintech_management_migrations`)

---

## Summary Status Table

| # | Repository Name | GitHub Action Required | Azure DevOps Action Required |
|---|---|---|---|
| **1** | `fintech_user_management` | None (Ready) | Sync branch ➔ Run CD pipeline |
| **2** | `fintech_super_admin` | None (Ready) | Sync branch ➔ Run CD pipeline |
| **3** | `fintech_business_management` | None (Ready) | Sync branch ➔ Run CD pipeline |
| **4** | `fintech_business_settings` | None (Ready) | Sync branch ➔ Run CD pipeline |
| **5** | `fintech_notifications_management` | None (Ready) | Sync branch ➔ Run CD pipeline |
| **6** | `fintech_statement_generator` | None (Ready) | Sync branch ➔ Run CD pipeline |
| **7** | `fintech_management_migrations` | None (Ready) | Sync branch ➔ Run CD pipeline |
| **8** | `fintech_webapp` | Update `Dockerfile` & `.env.required` | Sync branch ➔ Run CD pipeline |
| **9** | `fintech_admin_webapp` | Update `Dockerfile` & `pipeline_CI.yml` | Sync branch ➔ Run CI pipeline ➔ Run CD pipeline |
