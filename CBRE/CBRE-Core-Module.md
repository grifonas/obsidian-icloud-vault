#cbre
# TF Setup in CBRE Plan

## STAGE 1
- We move the EC2 stuff to `terraform/azure-pipeline-runners` - **DONE**

- Rename the existing workspace from `GDP-Replatform-core-dev` > `azure-pipeline-runners-all-envs`

- Merge the PR into `dev` - **DONE**

- Point `azure-pipeline-runners-all-envs` to  to the folder `terraform/azure-pipeline-runners`, branch **dev**.

- Set a reminder for next month to change this to `main`

- Add variables to `azure-pipeline-runners-all-envs`
	- `tags` (map)
	- `instances` (map, see the code)
	- `key_name` - ssh key name
	- `ami_id`
	- `vpc_type`
	- `subnet_type`

- Start a run. This will get rid of everything apart from the EC2.

## STAGE 2

- Create a new workspace called `core-dev`

- Configure it

- Kick off a run

## Stage 3 -25.07.2023

### Notes before we start:

- There're several deprecation notices in `core` related to the bucket module: **resource "aws_s3_bucket" "bucket" {**
	- `Use the aws_s3_bucket_lifecycle_configuration resource instead`
	- `aws_s3_bucket_versioning`
	- `aws_s3_bucket_server_side_encryption_configuration resource instead`

### Action Items:
1. Remove extra variables from **core**
	- `pub_sub_1` - remove
	- `pub_sub_2 ` - remove
	- `pub_sub_2` - remove
	- `AWS_SECRET_ACCESS_KEY` - should be an **ENV VAR**, not Terraform var.

1. Remove variables from **global**
	- `redirect_port` - remove
	- `environment` - remove
	- `instance_name_suffix` - remove
	- `instance_name_prefix` - remove
	- `port` - remove
	- `AWS_ACCESS_KEY_ID` - make ENV VAR
	- `AWS_SECRET_ACCESS_KEY` - make ENV VAR


2. Add path prefixes for triggers.
	- **global** should only run when `terraform/global/` changes
	- **core** - only when `terraform/core/` changes
	- **pipelines** -  `terraform/pipelines/`

3.  Configure `pipelines-dev` workspace.

4.  Kick off a run
5. Fix issues - **IN PROGRESS**





# Notes
## On Replication Instance Version 3.4.6 vs 3.4.7

1. There's an issue in Terraform that adds this option to the DMS S3 target endpoint: `add_trailing_padding_character=false`. This option is not supported in version 3.4.6. It is not enough to set it to false. It must not be added at all.
2. To solve this we need to upgrade to 3.4.7
3. To upgrade to 3.4.7 we must add A VPC Endpoint. See https://docs.aws.amazon.com/dms/latest/userguide/CHAP_VPC_Endpoints.html#CHAP_VPC_Endpoints.User_Mitigation.
4. We cannot create a VPC endpoint under this account (`067659249286`) as the VPC in question (`vpc-01195ee384d28beac`) is shared.
5. We reached out to AWS and they confirmed that this can be solved by requesting the owner of the source account to create a VPC endpoint for S3.
6. The endpoint settings we require are as follows:
	- Type: Gateway
	- Service: S3
	- Region of the S3 bucket: eu-west-2

## S3 Endpoint Settings:

```json
{
    "ServiceAccessRoleArn": "arn:aws:iam::067659249286:role/dms-s3-access-role-dev",
    "CsvRowDelimiter": "\\n",
    "CsvDelimiter": "|",
    "BucketName": "gdp-raw-zone-dev",
    "CompressionType": "NONE",
    "EnableStatistics": true,
    "IncludeOpForFullLoad": true,
    "CdcInsertsOnly": false,
    "ParquetTimestampInMillisecond": false,
    "CdcInsertsAndUpdates": false,
    "DatePartitionEnabled": false,
    "UseCsvNoSupValue": false,
    "PreserveTransactions": false,
    "UseTaskStartTimeForFullLoadTimestamp": false,
    "AddColumnName": false,
    "Rfc4180": true,
    "AddTrailingPaddingCharacter": false
}
```

```resource "aws_dms_s3_endpoint" "destination_s3" {
  depends_on                = [aws_iam_role.dms_role_to_access_s3]
  endpoint_id               = "gdp-replat-dst-s3-${var.environment}"
  endpoint_type             = "target"
  bucket_name               = var.dms_s3_bucket_name
  service_access_role_arn   = aws_iam_role.dms_role_to_access_s3.arn
  compression_type          = "NONE"
  csv_delimiter             = "|"
  csv_row_delimiter         = "\\n"
  date_partition_enabled    = false
  include_op_for_full_load  = true

  tags = local.combined_tags
}
```

**Ended up switching to the deprecated `s3_settings` in `aws_dms_endpoint`:
```
resource "aws_dms_endpoint" "destination_endpoint_s3" {
  depends_on                = [aws_iam_role.dms_role_to_access_s3]
  endpoint_id               = "gdp-replat-dst-s3-${var.environment}"
  endpoint_type             = "target"
  engine_name               = "s3"
  s3_settings {
    service_access_role_arn   = aws_iam_role.dms_role_to_access_s3.arn
    bucket_name               = var.dms_s3_bucket_name
    csv_delimiter             = "|"
    csv_row_delimiter         = "\\n"
    compression_type          = "NONE"
    include_op_for_full_load  = true
    timestamp_column_name     = "timestamp"
  }
  tags = local.combined_tags
}

```