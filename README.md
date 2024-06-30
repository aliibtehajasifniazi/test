# Client Data Archiving Runbook

## Table of Contents
1. [Introduction](#introduction)
2. [Pre-requisites](#pre-requisites)
3. [Archiving S3 Bucket Data](#archiving-s3-bucket-data)
4. [Mongo to Postrgres to Stack to Client to Subdomain ID Information](#mongo-to-postrgres-to-stack-to-client-to-subdomain-id-information)
5. [Archiving MongoDB Data](#archiving-mongodb-data)
6. [Archiving PostgreSQL Data](#archiving-postgresql-data)
7. [Deleting Mongo and Postgres Data](#deleting-mongo-and-postgres-data)
8. [Conclusion](#conclusion)

## Introduction
This runbook outlines the process for archiving old client data. It details steps to securely move existing data from Amazon S3 to Amazon Glacier, ensuring long-term and cost-effective storage. Additionally, it covers exporting and compressing data from MongoDB and PostgreSQL databases, uploading the archives to Glacier, and then safely deleting the original data from the servers.

## Pre-requisites
Before starting the archiving process, ensure you have the following:

1. **AWS Access**:
   - Access to the `RAM-AWS-StratifydTasteAnalytics-Admin` role for managing S3 bucket data.
   - Access to the `RAM-AWS-StratifydPostgres-Admin` role for handling PostgreSQL data.

2. **Database Access**:
   - Obtain `mongo_uri` and `postgre_uri` for connecting to MongoDB and PostgreSQL databases, respectively.
   - OpenVPN connection for secure access to MongoDB data.
   - Access to MongoDB Atlas to create a user with `dropDatabase` permission.

Ensure these prerequisites are in place for a smooth execution of the archiving tasks.

## Archiving S3 Bucket Data

### Moving Data to Glacier

1. **Login to AWS**:
   - Use the role `RAM-AWS-StratifydTasteAnalytics-Admin` to access AWS.

2. **Access S3 Buckets**:
   - Navigate to the S3 service and search for "stratifyd-client-data".
   - You should see the following buckets:
     - `stratifyd-client-data`
     - `stratifyd-client-data-local-test`
     - `stratifyd-client-data-test-env`

3. **Select the Client Data**:
   - Choose the first bucket, `stratifyd-client-data`.
   - Locate the client name you want to archive. For example, look for the object named `prudential/` if the client you want to archive is prudential.

4. **Create a Lifecycle Rule**:
   - If the object `prudential/` is present, navigate to the "Management" tab within the bucket.
   - Under "Lifecycle rules", click on "Create lifecycle rule".
   - Name the rule as "Move \<client-name\> to Glacier". For example, "Move prudential to Glacier".

5. **Configure the Rule**:
   - **Rule Scope**: Limit the scope using filters; in the prefix section, enter `<client-name>/` e.g. `prudential/`.
   - **Lifecycle Rule Actions**: Choose "Move current versions of objects between storage classes".
   - **Storage Class Transitions**: Select "Glacier Flexible Retrieval".
   - **Days after Object Creation**: Enter `0`.
   - Acknowledge the one-time lifecycle request cost and create the rule.

6.**Verify**:
   - You should a lifecyle rule looking like this:
     ![image](https://github.com/aliibtehajasifniazi/test/assets/171139951/83ec623e-6793-4bc1-ae4d-5ea243a7f34e)


7. **Repeat for Other Buckets**:
   - Follow the same steps for the other two buckets: 
     - `stratifyd-client-data-local-test`
     - `stratifyd-client-data-test-env`
   - Ensure the object `<client_name>/` exists in each bucket before proceeding.

## Mongo to Postrgres to Stack to Client to Subdomain ID Information
This is a complete layout of all the Mongo Clusters, their corresponding Postgres Clusters, the stack they are being used in, the clients that have their data in theses clusters and the client's respective sub domain IDs. Last updated `30-06-2024`.


| Mongo              | Postgres  | Stack       | Client               | Subdomain ID            |
|--------------------|-----------|-------------|----------------------|-------------------------|
| carpo-raspberry    | Herse     | raspberry   | emerson              | 5c5a511e78dba4123c8e131c |
|                    |           | get (vanilla)| prudential          | 5bad2ed7f55cabb466547a38 |
|                    |           |             | kcc                  | 5b8e97d6e5bb198aff7a5eba |
|                    |           |             | Thomson Reuters      | 6011be6fa148e48da399b6c9 |
|                    |           |             | lilly                | 5b685c204b81391ec3f94cca |
|                    |           |             | internal: get        | 5a823c4eb43983965dd1e156 |
|                    |           |             | internal: demo       | 60b690441e491cbf65df8f02 |
|                    |           |             | internal: uncc       | 60ec80a63787957109e51d9c |
|                    |           |             | lilly-testsub        | 5ff4966499f2b3151eb63538 |
|                    |           |             | stfysso              | 65a06445baac9e76dcb88867 |
|                    |           |             | deltek               | 65c10e18b8507e077ac9321e |
|                    |           |             | prc                  | 65c545c309c06cdf97cac147 |
|                    |           |             |                      |                         |
| himalia            | arche-db  | uat         | uat                  | 5dd46437b93aa380cd490e4d |
|                    |           | dev         | dev                  | 5dc2f5d1f8ab61e3cdbc5c3b |
|                    |           |             | lilly, lilly.beta    | 5b685c204b81391ec3f94cca |
|                    |           |             | qualityassurance, rabbit | 5c4a39749a796580dbbcf89f |
|                    |           |             | ally-testsub, testsub | 60300e095c0dcbafe5b3f42c |
|                    |           |             | ptest                | 653adb73bb3f015b44ac8997 |
|                    |           |             |                      |                         |
| sponde-strawberry  | sponde-db | strawberry  | priorityhealth       | 6063549710bd95a8cd1672c9 |
|                    |           |             | gilead               | 6233999ed8b5a7fc974790cc |
|                    |           |             | aetna                | 62619df6c660530674893e73 |
|                    |           |             | mach9                | 61a518a1d0bbd3177e41597a |
|                    |           |             | virtrutrax, virtutrax| 62016114f0177c58b40e890e |
|                    |           |             | dsi                  | 6227bc832b3bde9371ffbec6 |
|                    |           |             | otsuka               | 62b5f34515b866d578916d6d |
|                    |           |             | heapanalytics        | 5df9308525d84d1a5657db5c |
|                    |           |             | pmi-testsub, testsub | 5f08b5cbf6ddf0933e8fc4fb |
|                    |           |             |                      |                         |
| ersa-chocolate     | ersa-db   | chocolate   | schwab               | 5be0840de62ffd04f24ecd35 |
|                    |           |             | schwab-testsub       | 6033edd2bf756403406a5032 |
|                    |           |             | intuit               | 5b6c60d4e0fa4705fe069ea0 |
|                    |           |             | viivhealthcare, gsk  | 5be07dcd648de4a5cd2cefd8 |
|                    |           |             | cardinalhealth       | 5d03f3b43f113182b98863d1 |
|                    |           |             | heapanalytics        | 5df9308525d84d1a5657db5c |



## Archiving MongoDB Data

1. **Identify MongoDB URI**:
   - Locate the mongo cluster and production stack of the client to obtain the Mongo URI from the ECS of the stack in AWS as role `RAM-AWS-StratifydTasteAnalytics-Admin`. You can use the table above.
   - If you can not found it, access the Mongo URIs of all stacks using either 'mongosh' or 'mongo compass' (Connect to OpenVPN) and navigate to the `apps` database in all of them.
   - In the `apps` database, search the collection named `sub` and look for the client name in it.
   - Once located, note the Object ID of the document, which serves as the subdomain ID of the client.

2. **Check Database Presence**:
   - Within the same Mongo cluster, verify that a database exists with the name matching the client's subdomain ID.

3. **Download Data**:
   - If the database is present, use the following command to download and compress the data:

     ```bash
     mongodump --uri "<mongo_uri>" --db "<sub-domain ID>" --archive=path_in_local_machine\<sub-domain-ID>.gz --gzip
     ```

   - This command will download the database as a compressed file named `<sub-domain-ID>.gz` to the path in your local machine you specified.

4. **Upload to S3**:
   - Sign in to AWS with the `RAM-AWS-StratifydPostgres-Admin` role.
   - Navigate to S3 and select the bucket `archived-clients-db-data`.
   - Create a folder named `Client-<client-name>`, for example, `Client-prudential`.
   - Upload the `.gz` file to this folder.

   - Note: The S3 bucket is already configured with Glacier storage, so no additional actions are required.

## Archiving PostgreSQL Data


1. **Identify PostgreSQL URI and Cluster**:
   - Locate the PostgreSQL URI and cluster name for your client. For example, Prudential is in the Herse cluster (refer to the table above).

2. **Connect to AWS**:
   - Sign in to AWS with the role `RAM-AWS-StratifydPostgres-Admin`.
   - Navigate to EC2 instances and search for the desired PostgreSQL cluster, such as Herse.
   - Select `herse-pg15-node1` and click on **Connect** using the Session Manager.

3. **Verify the Client Database**:
   - In the Session Manager, connect to PostgreSQL:
     ```bash
     psql <postgres_uri>
     ```
   - List all databases:
     ```sql
     \l
     ```
   - Look for `db_<subdomain-ID>` corresponding to your client. If present, exit PostgreSQL:
     ```sql
     \q
     ```

4. **Dump the Database**:
   - Use the following command to dump the database data to the `/tmp` folder:
     ```bash
     pg_dump -h <host-ip> -p <port> -U <username> -d <database-name> -F c -b -v -f /tmp/db_<subdomain-ID>.dump
     ```
     You can find all this information in the Postgres uri.

5. **Compress the Dump File**:
   - Compress the dump file into a `.gz` format:
     ```bash
     gzip /tmp/db_<subdomain-ID>.dump
     ```

6. **Verify File Creation**:
   - Ensure the compressed file is created:
     ```bash
     ls -lh /tmp/db_<subdomain-ID>.dump.gz
     ```

7. **Upload to S3**:
   - Send the file to the designated S3 bucket:
     ```bash
     aws s3 cp /tmp/db_<subdomain-ID>.dump.gz s3://archived-clients-db-data/Client-<client-name>/
     ```
   - Example:
     ```bash
     aws s3 cp /tmp/db_5bad2ed7f55cabb466547a38.dump.gz s3://archived-clients-db-data/Client-prudential/
     ```

8. **Clean Up**:
   - After verifying the file upload, delete the dump file from the PostgreSQL server:
     ```bash
     rm /tmp/db_<subdomain-ID>.dump.gz
     ```
9. **Verify**:
   - After archiving Mongo and Postgres data successfully, your S3 Bucket should look like this:
     ![image](https://github.com/aliibtehajasifniazi/test/assets/171139951/f383f5a1-0972-46f2-af4d-9a831f0ce9aa)
     This is an example for prudential where `5bad2ed7f55cabb466547a38.gz` represents the Mongo data and `db_5bad2ed7f55cabb466547a38.dump.gz` represents the postgres data.

     

## Deleting Mongo and Postgres Data

### Creating a Job Definition

1. Sign in to AWS as the role `RAM-AWS-StratifydTasteAnalytics-Admin`.
2. Navigate to AWS Batch.
3. Go to **Job Definitions** and create a new job definition with these settings:
   - **Orchestration type:** EC2
   - **Name:** `<stack>-delete-<client-name>-subdomain`
     - Example: `raspberry-delete-prudential-subdomain`
4. Click **Next**.
5. Configure the image and command:
   - **Image:** `818160864477.dkr.ecr.us-west-2.amazonaws.com/ci/maintenance-utilities:0.6.26`
   - **Command:** 
     ```json
     ["-m","maintenance_utilities.utility.delete_subdomain","--subname","<subdomain-name (not ID)>","--user","<Your Name>","--soft-delete"]
     ```
     You can find the subdomain name in Mongo `app` database in the `sub` collection.
     example
      ```json
     ["-m","maintenance_utilities.utility.delete_subdomain","--subname","prudential","--user","Ali Ibtehaj Asif Niazi","--soft-delete"]
     ```
6. Set roles:
   - **Execution role:** `<stack>-prod-task-execution-role`
   - **Job role configuration:** `<stack>-prod-task-execution-role`
     - Example: `raspberry-prod-task-execution-role`

5. Configure resources:
   - **vCPUs:** 1
   - **Memory:** 256

6. Add Secrets:
   - `MONGO_URI`: `:mongo_uri_drop_database::`
   - `POSTGRES_URI`: `:postgres_uri::`

7. Add Environment Variables:
      - `POSTGRES_MAX_CONNECTIONS`: `100`
      - `STFY_PSQL_REMOVE_MUTUAL_TLS`: `true`
      - `INTERACTION_OFF`: `true`
      - `PY_BIN`: `python3.7`

8. Click **Next**.

9. In Configure logging:
    For Log driver select `awslogs`
11. Click **Next**, review, and create the job definition.

### Submitting First Job for Soft Delete Dry Run
1. Go to the new job definition and click Action > Submit a new job.
2. Use these settings to create a new job:
    - **Name**: `<stack>-delete-<client-name>-subdomain-dry-run`
    - **Job definition**: `<stack>-delete-<client-name>-subdomain`
    - **Job queue**: `<stack>-maintenance-queue`
3. Click **Next**.
4. In Container overrides, select `Load from job definition`.
5. Click **Next** and then Create Job.
6. Monitor the job status. If it fails, review the logs. If it succeeds, verify the logs indicate:
    - Dry Run
    - Would Rename postgres database `db_<subdomain-id>` to `deleted_db_<subdomain-id>`
    - Would Change subdomain doc subdomain name from `[<subdomain-name>]` to `[deleted_<subdomain-name>]`, and moved to collection: %s"

### Submitting Second Job for Soft Delete with teeth
1. Go to the new job definition and click Action > Submit a new job.
2. Use these settings to create a new job:
    - **Name**: `<stack>-delete-<client-name>-subdomain-soft-delete`
    - **Job definition**: `<stack>-delete-<client-name>-subdomain`
    - **Job queue**: `<stack>-maintenance-queue`
3. Click **Next**.
4. In Container overrides, select `Load from job definition` and add `--teeth` at the end so that the command looks like this:
```json
     ["-m","maintenance_utilities.utility.delete_subdomain","--subname","<client-name>","--user","<Your Name>","--soft-delete","--teeth"]
```
5. Click **Next** and then Create Job.
6. Monitor the job status. If it succeeds, verify the logs indicate:
    - We are going to soft delete subdomain <subdomain-name>
    - Delete action id <action-id>
    - Renamed postgres database `db_<subdomain-id>` to `deleted_db_<subdomain-id>`
    - Changed subdomain doc subdomain name from `[<subdomain-name>]` to `[deleted_<subdomain-name>]`, and moved to collection: %s"
Keep note the `Delete action id` for the next step.
To verify further go to the postgres server in the session manager and look in the list of databases you should see deleted_db_<subdomain>.
Then go to mongo cluster, in the app database and in the deleted sub collection you should see the name of the client in one of the documents in that cluster as deleted_<client_name>

### Submitting Third Job for Hard Delete
1. Go to the new job definition and click Action > Submit a new job.
2. Use these settings to create a new job:
    - **Name**: `<stack>-delete-<client-name>-subdomain-hard-delete`
    - **Job definition**: `<stack>-delete-<client-name>-subdomain`
    - **Job queue**: `<stack>-maintenance-queue`
3. Click **Next**.
4. In Container overrides, write this command for hard delete:
    ```json
     ["-m","maintenance_utilities.utility.delete_subdomain","--subname","<subdomain name>","--user","<Your Name>","--hard-delete","--action-id","<action-ID>","--teeth","--force"]
   ```
5. Click **Next** and then Create Job.
6. Monitor the job status. If it succeeds, verify the logs indicate:
    - We are performing HARD DELETE on subdomain <subdomain-name>.
    - Dropped postgres database: deleted_db_5bad2ed7f55cabb466547a38
    - Dropped mongo database 5bad2ed7f55cabb466547a38
      
Verify further by going to postgres database list and there should be no db_<subdomain-ID> or deleted_db_<subdomain-ID> present. Then go to the Mongo Cluster and there should be do database named <subdomain-ID>.
This means everything has been hard deleted successfully.

## Conclusion
Summarize the process and any final notes.
