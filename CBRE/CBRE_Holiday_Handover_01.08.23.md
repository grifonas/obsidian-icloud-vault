
# CBRE Handover


## Communication with CBRE

For a quick response use Teams under you lucas.roesler@cbre.com user.


## Current State

### **TERRAFORM**

- **GLOBAL** - done and dusted. Contains the Azure runner only.

- **CORE** - Setup with automatic plan and apply on pushes to `dev`. [TFE](https://tfe.cloudeng.cbre.com/app/cbre/workspaces/GDP-Replatform-core-dev).

- **PIPELINES** - Setup with auto plan and apply n pushes to `dev`. [TFE](https://tfe.cloudeng.cbre.com/app/cbre/workspaces/GDP-Replatform-pipelines-dev)

### Making Changes in TF
- **Code**: Change the code and push to dev
- **Variables**: We *DO NOT HAVE WRITE ACCESS* to any of the TFE workspaces so we have to ask Sumalatha (Ande.Sumalatha@cbre.com) to change and then trigger a plan and apply.


## Issues
- MSSQL user lacks permissions. In touch with a million people on their side. Added you to the thread. The person responsible for the DB seems to be Leon.
- Sent them a suggestion that they should have a look at the PROD DB CDC user. That one has a working DMS task (called `prd-argoweb-de-sql-server`. [AWS Link](https://eu-west-2.console.aws.amazon.com/dms/v2/home?region=eu-west-2#endpointDetails/prd-argoweb-de-sql-server)). DMS MSSQL settings that we know are working

```json
{
	"Port": 1433,
	"DatabaseName": "prd_argoweb_de",
	"ServerName": "SQLPRDDNS_ArgoWeb_DE.emea.cbre.net",
	"Username": "sql_PRD_argoweb_de_aws_replica"
}
```

## Nest Steps

**Checking DMS**
- DMS task: [AWS DMS task link](https://eu-west-2.console.aws.amazon.com/dms/v2/home?region=eu-west-2#taskDetails/gdp-replatforming-replication-task-dev)
- Can be restarted manually every time they say they made changes.