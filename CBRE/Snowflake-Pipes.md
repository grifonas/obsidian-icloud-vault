# Issues with Ingesting multiple Files from S3 to Multiple Tables Using the Same Stage
#cbre 

## Our Goal
- We’re ingesting multiple files from S3.
- We’ve got multiple S3 paths in the same bucket that we want to ingest from.
- Every path contains files that have to be I tested into their own dedicated tables.

## Our Setup
- We've set up a pipe per S3 path.
- All the pipes use the same stage.
- Every pipe’s COPY INTO statement has its own suitable FROM parameter. The differ from each other by the `path` that comes after the stage name (which is the same for all the pipes) For example:
### Pipe #1:
```sql
COPY INTO MAIN_ACTIVITY_CHANGE_FEED(
[.....]

)

FROM @OUR_STAGE/main_activity (file_format => [...]) t
)

ON_ERROR = CONTINUE;
```


### Pipe #2:
```sql
COPY INTO USERS_CHANGE_FEED(
[.....]

)

FROM @OUR_STAGE/users (file_format => [...]) t
)

ON_ERROR = CONTINUE;
```

- Note that we have dozens of prefixes in S3 and therefore dozens od destination tables and pipes.

## Our Issue

When we create the pipes and enable DMS (which is what populates the bucket) only one single pipe out of the dozens we have defined picks up the files and inserts them into its respective destination tables.


## The Question
How can we fix that?

Do we need to create a stage per S3 path?
If so, what is the limit of the amount of stages that can be created in a single Snowflake account?

Note that we are eventually planning to ingest from and into more than a 12000 tables.

Also note that this ticket closely resembles our query: https://community.snowflake.com/s/question/0D50Z00009VvsW8SAJ/if-a-database-has-2-or-more-pipes-will-all-of-them-have-the-same-notificationchannel

In the last massage a Snowflake colleague mentions that `"The stage reference has to be different for each pipe otherwise they can load same set of files into one or more target tables."`
In our case, however, our stage reference is different as every pipe defines a different S3 path.


Thank you!


# Tables