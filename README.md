# Azure DevOps Telemetry Configuration Report

1. Required Configuration for Payment Flow
 We need to create these for  Payment Flow (e.g. ZB-FintechPaymentFlow-QA and ZB-FintechPaymentFlow-PROD), they must include the following three variables for  telemetry  to work:

## Variable Name	& value
```
APPLICATIONINSIGHTS_CONNECTION_STRING	=
TELEMETRY_COMPONENT = 	fintech_payment_flow
OTEL_SERVICE_NAME	= fintech_payment_flow	
```


## Exact values we need to add for each specific group:
```
Azure DevOps Variable Group Name	       Missing Variable to Add	                           Required Value
ZB-FintechUserManagment-QA / PROD           OTEL_SERVICE_NAME	                                  fintech_user_management
ZB-FintechBusinessManagment-QA / PROD	      OTEL_SERVICE_NAME                                 	fintech_business_management
ZB-FintechBusinessSettings-QA / PROD	      OTEL_SERVICE_NAME	                                fintech_business_settings
ZB-FintechDocumentsManagement-QA / PROD 	  OTEL_SERVICE_NAME                                	fintech_documents_management
ZB-FintechManagementMigrations-QA / PROD	  OTEL_SERVICE_NAME	                                fintech_management_migrations
ZB-FintechNotificationsManagement-QA / PROD	OTEL_SERVICE_NAME                              	fintech_notifications_management
ZB-FintechProcessingScripts-QA / PROD	      OTEL_SERVICE_NAME	                              fintech_processing_scripts
ZB-FintechStatementGenerator-QA / PROD	      OTEL_SERVICE_NAME	                            fintech_statement_generator
ZB-FintechSupeAdmin-QA / PROD	              OTEL_SERVICE_NAME	                              fintech_super_admin
```
Hi
