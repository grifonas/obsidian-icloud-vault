# CBRE Access

## MS SQL
- Browsing to  esmdrargt002.emea.net forwards to GoDaddy's domain purchasing page.
- We tried testing the access using the Azure pipeline but the pipeline no longer gets triggered when we commit to DEV. We tried creating a new pipeline but got tis error:```
TF215106: Access denied. Konradt, Greg needs Queue builds permissions for build pipeline 17494:GDP-Replatform (1) in team project Advisory Services Continental Europe to perform the action. For more information, contact the Azure DevOps administrator.```



## AWS
- We can confirm that we can access the account 067659249286.
- S3 Buckets:
	- `gdp-landing-zone-dev` **We found an issue** : `Insufficient permissions to list objects`.
	- We will need to be able to view the object in the bucket. Thank you.
	- We would like to **recommend that our Terraform template creates the buckets** instead of you creating them manually. It will make it much easier to manage as we will need to be able to create and modify the bucket notification policies.
- EC2  `i-0fef0f6a5fcd3f17a`. We don't require an instance as we're using Azure Pipelines for running the automation. You can delete it.
- All the DMS resources will be created by our Terraform template. You may delete the resources you created manually. Thank you.

# Snowflake
-The **DB DEV_DEDM_DB**: The DB exists. **Issues**:
	- We did not see the **DEV_GDP_REPLATFORMING_DMS_S3** . `SHOW INTEGRATIONS` only shows these integrations:
		- `CUSTOM_OAUTH_KIRAN`
		- `CUSTOM_OAUTH_POSTMAN`
		- `DEV_SNAPLOGIC_OAUTH`
- We did not find Snowflake access secrets inAzure Devops Library