#tech-tip #oidc #aws

# Terraform Auth to AWS with OIDC

Based on [this article](https://developer.hashicorp.com/terraform/tutorials/cloud/dynamic-credentials)

1. Go over the code and the variables in https://github.com/contiamo/project-ops/tree/main/Contiamo/tf-cloud-oidc
2. Run:
```
terraform init
terraform apply
```
Note the `ARN` in the outputs.
3. Go to TFC > your project > namespace and create these `ENVIRONMENT` variables (not TF variables):
	- `TFC_AWS_PROVIDER_AUTH` = `true` - tell TF to use the OIDC connector
	- `TFC_AWS_RUN_ROLE_ARN` = `[the ARN you noted in step #2]`
4. Edit your `aws-auth` configmap in the `EKS` cluster and add this bit to the `mapRoles` section:
```
    - groups:
      - "system:masters"
      rolearn: arn:aws:iam::990343961361:role/terraform-cloud-westag-tender
      username: any-name
```