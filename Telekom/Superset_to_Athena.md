# Superset Athena Connector
#telekom #athena #superset
```Dockerfile
FROM apache/superset:2.1.0
# apache/superset:2.1.0 is out but it has a bug in the db migration script: https://github.com/apache/superset/issues/23483
# Switching to root to install the required packages
USER root
RUN pip install sqlalchemy-bigquery
RUN pip install psycopg2==2.9.1 redis==3.5.3 PyAthena==3.0.2 PyAthenaJDBC
RUN apt update && apt install netcat -y
# Switching back to using the `superset` user
USER superset
```

```bash
awsathena+rest://athena.eu-central-1.amazonaws.com/analytics_dwh_dev?s3_staging_dir=s3%3A%2F%2Fdev-ci-mpathic-analytics-dwh%2Fsuperset

```