#cbre
## CBRE Questions for Naming Meeting


### 1. Bucket names.
- `gdp-replatforming-dms-dev`
- `gdp-replatforming-dms-qa`
- `gdp-replatforming-dms-prod`


### 2. Snowflake:

**Storage Integration names**:
-  `GDP_REPLATFORMING_S3_INTEGRATION_DEV`
-  `GDP_REPLATFORMING_S3_INTEGRATION_QA`
-  `GDP_REPLATFORMING_S3_INTEGRATION_PROD`

**Storage Int. CREATE statement:**
```sql
CREATE OR REPLACE STORAGE INTEGRATION GDP_REPLATFORMING_S3_INTEGRATION_DEV
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::067659249286:role/snowflake-dev-s3-reader'
  STORAGE_ALLOWED_LOCATIONS = ('s3://gdp-replatforming-dms-dev/')
  COMMENT = 'Created manually by CBRE'  		
```

Kindly provide the following details from the `DESC INTEGRATION GDP_REPLATFORMING_S3_INTEGRATION_DEV` command:
- STORAGE_AWS_EXTERNAL_ID
- STORAGE_AWS_IAM_USER_ARN

### 3. Will we use the same AWS account and VPC for all the envs?



# DMS Module Notes
pass to module:
`create_repl_subnet_group = false`
`repl_instance_subnet_group_id` of pre-created


Put this in the subnet group we create manually:
```# Subnet group

repl_subnet_group_name = "${var.dms_subnet_group_name}-${var.environment}"

repl_subnet_group_description = "${var.project_name} ${var.environment} DMS Subnet group"

repl_subnet_group_subnet_ids = var.dms_subnet_ids
```

### An update on the above:
AWS assumes that the VPC IAM role for DMS is called `dms-vpc-role`. There's no way to change that.
Hence:
- Trying to omit it from the module and see whether it gets created. In this scenario we still let the module create the subnet group. **FAILED: the role isn't created.
- Trying to take the role out of the template and create it in PREREQs - **WORKED**



# Terraform

## The One Repo Question

- US: infra and app are the same thing. It is building pipeline. Changes like schema > snowpipe. It's  easier to keep in sync.
	- **They**: Snowflake is not infra. The DevOps team: Do we need to make changes in DMS?
	- US: No.