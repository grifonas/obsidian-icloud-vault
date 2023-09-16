# Terraform Tips

#tech-tip 
## Migrating state from S3 to TF Cloud

Taken from https://developer.hashicorp.com/terraform/tutorials/cloud/migrate-remote-s3-backend-tfc

1. Replace backend config: `backend s3` > `cloud`:
```
```
```hcl
terraform {
  cloud {
    hostname = "app.terraform.io"
    organization = "contiamo"
    workspaces {
      tags = ["jupyter-hub"]
    }
  }
}
```

	1. Run `terraform init`