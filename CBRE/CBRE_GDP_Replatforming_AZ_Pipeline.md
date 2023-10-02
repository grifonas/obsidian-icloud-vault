#cbre
# AZ Workflow:
## Description
- Connect to MS SQL
- Run the Generator
- Run the generated DBT model. (a folder `warehouse/models`)
- Both generated DBT pipelines AND *manual* pipelines will need to be run

## Required Connections:
- Snowflake
- MSSQL

## What to run
### Workflow 1
- Generator: `Update YAML`
- Generator: `Update pipeline models (DBT)`
### Workflow 2
- `DBT run`
