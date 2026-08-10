# Zenus App Insights — Fix `RUN npm ci` CI Build Failure (3 Target Repositories)

**Organization:** `https://dev.azure.com/ZenusBankInternational`  
**Project:** `ZB-CS - PP Digital Portal Solution`  

---

## 🎯 Verified Error & Fix

Docker Line 28: `RUN npm ci` failed with `exit code 1` because `package-lock.json` was missing the newly added telemetry libraries.

---

## 🛠️ Exact Commands to Run (Copy-Paste)

### 1. `fintech_webapp`
In your local terminal inside `fintech_webapp`:
```bash
npm install @microsoft/applicationinsights-web
git add package.json package-lock.json
git commit -m "fix: sync package-lock.json for applicationinsights"
git push origin staging
```

---

### 2. `fintech_admin_webapp`
In your local terminal inside `fintech_admin_webapp`:
```bash
npm install @microsoft/applicationinsights-web
git add package.json package-lock.json
git commit -m "fix: sync package-lock.json for applicationinsights"
git push origin staging
```

---

### 3. `fintech_statement_generator`
In your local terminal inside `fintech_statement_generator`:
```bash
npm install @azure/monitor-opentelemetry
git add package.json package-lock.json
git commit -m "fix: sync package-lock.json for telemetry"
git push origin staging
```

---

## 🚀 Post-Push Pipeline Execution

1. Sync GitHub **`staging`** ➔ Azure DevOps **`qa`** branch.
2. Trigger the CI/CD pipelines in Azure DevOps.
3. `RUN npm ci` will complete successfully, and your build will pass!
