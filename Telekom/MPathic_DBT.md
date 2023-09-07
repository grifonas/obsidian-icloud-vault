
# DBT for data Platform

- DBT materialises databases.
- Should be run every 10 minutes.
## What does it do?

- Connects to Athena (in MPathic AWS)
- Updates tables
- DBT will materialise tables in different databases
## How to Run
- SQL file are in `models/*.sql`
- To Run:
```
poetry run dbt build --target prod
```
- `profiles.yaml` - check that file for access details (target db, bucket, etc)
- `dbt_project.yml` - check this as well.
-

### DBs to Connect to
- `*.prod` DBs

## Docs
- Every run generates artefacts
- Artefacts are stored in `target`
- Already deployed manually by MPathic (@Gourav Saklecha) to http://dwh-dbt-doc.s3-website.eu-central-1.amazonaws.com/#!/overview

> [! Callout test]
> Nice
