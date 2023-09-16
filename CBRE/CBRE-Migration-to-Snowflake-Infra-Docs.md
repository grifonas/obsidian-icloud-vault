# GDP Replatforming Infrastructure  Documentation
#cbre 

## Description
The infrastruvture consist oif two major part:
1. AWS DMS (Database Migration Service)
	1. A DMS instance
	2.  DMS replication task
	3. Supporting resources: DMS instance security group, DMS subnet group.
2. Snowflake Resources
	1. Target tables
	2. Stages
	3. Snow pipes

Please find a detailed description below.

## 1. DMS

- DMS Instance - required as part of AWS implementation of the service. I compute instance that will run the tasks. This is where we can **manage our resource requirements**.

- DMS replication task. This is where the rules for which table changes should be replicated are defined.
	- The rules are called **Table Mappings** . See the documentation [here](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TableMapping.html).
	- As we add items to the list of tables we want to replicate we will be adding Table Mapping rules as documented [here](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TableMapping.SelectionTransformation.Selections.html).
	- This part is tightly coupled with Snowflake. The procedure for adding a new table to the list is as follows (**the order is crucial**); In this example we will add a new table called "EVENTS":
		1. Create a target table "EVENTS_CHANGE_FEED" in Snowflake.
		2. Create a new Snowflake stage with the parameter **URL** set to the S3 URL for the CDC files in the bucket: `URL = 's3://[BUCKET NAME]/dbo/EVENTS/'`
		3. Create a new Snowpipe called "EVENTS"
		4. Add anew table mapping rule in DMS:
		```json
        {
            "rule-type": "selection",
            "rule-id": "1",
            "rule-name": "1",
            "object-locator": {
                "schema-name": "PUBLIC",
                "table-name": "EVENTS"
            },
            "rule-action": "explicit"
        }
		```

	⚠️ Warning ⚠️
	Files created by DMS and placed into S3 that haven't got their corresponding **target tables**, **stages** and **pipes** defined will
	1.  NOT BE INGESTED,
	2. Marked as dealt with by Snowflake
	In practice that means data loss. This is why the order of automated actions described above is very important.



## 2. Snowflake

As described above, we are creating the following resources in Snowflake:
- **Target tables** will hold the CDC data from the source MSSQL tables. All the tables will have th same schema that will contain common DMS metadata:
	- `FILENAME`
	- `FILE_ROW_NUMBER`
	- `FILE_CONTENT_KEY`
	- `FILE_LAST_MODIFIED`
	- `START_SCAN_TIME`
	and
	- `DATA`. Data will contain the actual data from the source table. Its schema will be defined in the related pipe.
- **Stages**. They are responsible for setting the S3 path that contains the CDC file for the target tables.
- **Pipes**. Define the flow from files to the target table. This include the schema of the `VARIANT` field in the target table.