# Azure DevOps Variable Group Audit Matrix

| Variable Name | Present in Azure DevOps? | Azure DevOps Variable Groups Found In | Value in DevOps (QA) |
|---|---|---|---|
| TUUM_ACCOUNT_URL | YES | ZB-FintechUserManagment-QA<br>ZB-FintechSupeAdmin-QA<br>ZB-FintechProcessingScripts-QA | https://account-api.zenus-test.tuumplatform.com |
| TUUM_PAYMENT_ROUTER_HOST | YES | ZB-FintechBusinessManagment-QA<br>(also as TUUM_PAYMENT_ROUTER_URL in ZB-FintechUserManagment-QA) | https://payment-router-api.zenus-test.tuumplatform.com |
| AZURE_TENANT | YES | ZB-FintechUserManagment-QA<br>ZB-FintechSupeAdmin-QA<br>ZB-FintechNotificationsManagement-QA<br>ZB-FintechBusinessSettings-QA | zenusqa |
| AZURE_TENANT_ID | YES | ZB-FintechUserManagment-QA<br>ZB-FintechSupeAdmin-QA<br>ZB-FintechNotificationsManagement-QA<br>ZB-FintechBusinessSettings-QA | d4abffbd-1917-4272-af30-5539f8b8debd |
| AZURE_CLIENT_ID | YES | ZB-FintechUserManagment-QA<br>ZB-FintechSupeAdmin-QA<br>ZB-FintechNotificationsManagement-QA<br>ZB-FintechBusinessSettings-QA | 55bd3efa-94be-4f48-a9cd-11f34b3e4d7d |
| AZURE_CLIENT_SECRET | YES | ZB-FintechUserManagment-QA<br>ZB-FintechSupeAdmin-QA<br>ZB-FintechNotificationsManagement-QA<br>ZB-FintechBusinessSettings-QA<br>ZB-FintechBusinessManagment-QA | Secret (Locked) |
| AZURE_APP_FLOW | YES | ZB-FintechUserManagment-QA<br>ZB-FintechSupeAdmin-QA<br>ZB-FintechNotificationsManagement-QA<br>ZB-FintechBusinessSettings-QA | B2C_1_ROPC_PURPLEPLUM |
| AZURE_API_SCOPE | YES | ZB-FintechUserManagment-QA<br>ZB-FintechSupeAdmin-QA<br>ZB-FintechNotificationsManagement-QA<br>ZB-FintechBusinessSettings-QA | User.Read |
| CIRCLE_API_HOST | YES | ZB-FintechUserManagment-QA<br>ZB-FintechBusinessManagment-QA | https://func-mw-circle-qa-eastus-002.azurewebsites.net |
| CIRCLE_FUNCTION_KEY | YES | ZB-FintechUserManagment-QA<br>ZB-FintechBusinessManagment-QA | Secret (Locked) |
| CIRCLE_IDEMPOTENCY_KEY | NO | Not in Variable Groups | Generated dynamically per-request in application code |
