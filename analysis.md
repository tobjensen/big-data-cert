# Analysis

* Kinesis Analytics (Sm)
* Elasticsearch Service (Med)
* Athena (Sm)
* Redshift (Big)
* RDS (Sm)

## Kinesis Analytics

Kinesis Analytics works with Kinesis Data Streams and Kinesis Firehose.
Helps you to do real-time analytics, by writing either SQL or Flink on your stream.
Kinesis Analytics outputs data to either Kinesis Data Stream or Firehose (or a Lambda function).

* The Kinesis Analytics application contains the SQL that queries, filters and creates streams
* Kinesis Analytics applications continuosly reads data in real-time
* Inputs from KDS or Firehose populates **in-application input streams**
* You can use the **Schema Discovery Feature** to infer schema on either static or streaming data
* You can also pre-process the data with a Lambda function
* Optionally, you can add a **reference table** from S3, fx. to help looking up ids to values
* Kinesis Anlytics supports multiple input streams
* You can increase thoughput by parallalizing input streams, with the `InputParallelism` parameter
* The **application code** continually reads from the in-application data streams
* The **application code** can then create and output data into **in-application output streams**
* An **error stream** is automatically created, records that produced an error are logged there
  * Error: a record read does not conform to the schema
  * Error: application code is dividing by zero
  * Error: rows are out of order (a record is late, or user modified the ROWTIME value)
  * Error: Coercion error: JSON in input could not be converted/read by the SQL application code
* Kineses Analytics can then be configured to send the records in an **in-application output stream** to a destination, either Kinesis Data Streams, Firehose or Lambda
* Uses **at least once** processing and delivery model for records
* Kinesis Analytics automatically scales, up to 8 KPU's (32 GB of memory), can be expanded

### Timestamps and ROWTIME

In application streams has a special field called ROWTIME.
ROWTIME is automatically populated by Kinesis Analytics.
ROWTIME is the exact time when a record is first inserted into a in-appliation stream.

* **Event time** typically the timestamp on a record put there by a server when the record was created. It is not guaranteed to be accurate (the clock on the server might be off!).
* **Ingest time** if the input is a Kinesis Data Stream, KDS includes a field called APPOXIMATE_ARRIVAL_TIME, which is the time when the record hits KDS. It is often pretty close to the right time, but in case of a rare delay (KDS is a distributed service), records and times may be out of order.
* **Processing time** This is ROWTIME, the time Kinesis Analytics assigns a record when it is first inserted into an in-application stream. This is always increasing. However, if the application falls behind, this time will be further from when the event actually occurred.

## Queries and Windows

There are two types of queries: **Continuous Queries** and **Windowed Queries**.

* **Continuous Queries** run all the time on streaming data. It runs on every new record in the stream
* **Windowed Queries** are run over all records in a specific window, it could either be a time interval or a number of rows
  * **Tumbling Windows** run at a fixed interval, like every 60 seconds are 20 rows. They are very simple. There is no overlap. Would use `GROUP BY` and `BY INTERVAL`. Tumbling Windows are simple, fixed intervals. Tumbling windows are the simplest.
  * **Sliding Windows** slide continously with time. It looks back continously, fx 5 seconds. Whenever a new records comes in, it emits a result with all the records in the last 5 seconds. Sliding windows only emit results when a new record comes in. Sliding windows are a bit like the sliding controller on stock graphs. When a new record comes in, it emits all the records that are within the look-back window.
  * **Stagger Windows** are good for analysing out-of-order events or late arriving events or events that arrive at inconsistent times. Windows only open when the first event arrives, and closes at a set interval after opening. If the first event is supposed to arrive exactly on the hour, but it staggers in 10 minutes late, the window opens for events to come in within the next 60 minutes. If we open the staggered window a little late, we also close it a little late.

### Encryption on Kinesis Analytics

Kinesis Analytics is a managed service, where encryption of data in transit is enabled by default.

* It is not possible to bring your own keys for this. Kinesis Analytics does it for you.
* Kinesis Analytics is a streaming applicaiton, where there is never any data at rest.
* Application code and optional reference data at is encrypted by default by the serice.

* Input and output data is handeled by Kinesis Data Streams or Kinesis Firehose.
* Kinesis Data Streams supports encryption at rest with KMS.
* Kinesis Data Streams also supports client-side encryption through an add-on library
* Kinesis Firehose supports encryption at rest from Direct PUT with symmetric CMKs from KMS

## ElasticSearch Service

A managed ElasticSearch cluster, run by AWS

* Great for processing, analysing, reporting and visualising log data
* Scales to terabytes of data
* For some types of queries, it is even faster than Apache Spark!
* Kibana –> A tool for queriynig, analysing and visualising data within ElasticSearch
* Kibana can make great dashboards
* Kibana can also be used to type queries interactivly
* Ingest data with LogStash (a part of Beats), similar to Kinesis or Kafka
* You can also use Kinesis, native with Firehose

### ElasticSearch Documents, Types & Indices

Data in ElasticSearch are stored as **Documents**.
A Document is a JSON object that has a unique id and a type (deperected, up until ElasticSearch 6)

**Types** are going away in ElasticSearch.
For each Document Type, you would have a shared schema and mapping.
A type could be a log entry, or a wiki article.

**Indices** power the search into Documents.
They also contain inverted indecies so you can search across everything at once.

An index is split into **shards**, where documents are then hashed to a particular shard.
Each shard may be a seperated *Node* in a cluster.

ElasticSearch has **redundency** through replicating shards.
A primary shard may be replicated twice, with the replications store on two seperated nodes in the cluster.
A *write* request is sent to the primary shard, and then replicated the the two replicas.
A *read* request is sent to a primary shard or any replica shard.

### AWS ElasticSearch Service

Fully managed, at scale, zero downtime.
No pain, no management of EC2 instances.

You still have to think about how many and which EC2 instances. Like EMR.
You only pay for EC2 instances used.
Idle clusters should be shut down!

You can encrypt data at rest at in-transit.
**Node to Node Encryption** for data in flight between nodes in cluster.
Put everything behind a VPC for network isolation.

Integrates well with IoT, S3, Kinesis, Dynamo, CloudWatch/CloudTrail.

Has Zone Awareness.
You can allocated instances to be across to Avaliablity Zones for greater resilliency. (Though higher latency).

* How many dedicated Master Nodes
* Domain is a collection of all configurations for the cluster
* Snapshots in S3 possible

### Security supports many things

* Ressource base policies
* Identify based policies
* IP based policies
* Request signing is mandatory
* VPC (you must decided this at creating time, cannot change later)
* Cognito (for talking to Kabana)

### Securing Kabana with Cognito

* Supports Microsoft Active Directory
* Supports other SAML (Google, Facebook, etc.)

* Getting inside the VPC with a **Reverse Proxy**
* **SSH Tunnel** on port 5601
* Use **VPC Direct Connect**
* Use **VPN**

### Anti-Patterns for AWS ElasticSearch

* Don't do OLTP
* Ad hoc-data queriying (use Athena)
* ElasticSearch Service is primarily for search & analytics

## AWS Athena

* Query data in S3 with SQL
* Can query anything with schema in Glue Data Catalogue
* Uses Presto under the hood

Formats:

* CSV
* JSON
* ORC (columnar)
* Parquet (columnar)
* Avro

* Athena works well with Columnar data (ORC or Parquet)

* Can connect to S3 buckets in different accounts

* Encryption of results:
  * SSE-S3 (AES 256 in S3)
  * SSE-KMS (KMS key in S3)
  * CSE-KMS (KMS key in Athena)
  * Encryption in transit with TLS

### Athena Anti patterns

* Don't use it for highly formatted reports, use QuickSight
* Don't us it for ETL, that is what Glue is for

## Redshift

Petabyte scale data warehouse

* Designed for OLAP
* Interfaces with SQL
* Also interfaces with ODBC and JDBC
* Cost effective (but not cheap!)
* Built-in replication and backups
* Scales up and down on demand (not automatic, you have to do it!)
* Integrates with CloudWatch with default performance metrics (+ optionally custom metrics)
* You can purchase Reserved Nodes to use with your Redshift Cluster

### Redshift Architecture

Redshift is run as a cluster of Nodes

* **Leader Node** manages communication with clients and compute nodes – plans execution
* 1-128 **Cumpute Nodes** process queriess, and sends intermediate results back to the Leader Node

* Two Types of Compute Nodes:
  * Dense Storage, with HDD
  * Dense Compute, with SSD

* Compute nodes have several *Node Slices*
  * Node slices process chunks of data given to it

### Redshfit Spectrum

* Query EXABYTES of unstructured data in S3 without loading
* Like Athena
* Redshift Spectrum does the compute, S3 does the storage
* Reads tons of formats
* Supports either Gzip or Snappy compressions
* Really quick and concurrent

* Spectrum needs to first `CREATE EXTERNAL SCHEMA` from either Athena Data Catalog, Glue Data Catalog, or a Hive Metastore (like on EMR)
* After creating an external schema, you can `CREATE EXTERNAL TABLE` that has the column names, file type information and S3 location:

```sql
create external table spectrum.sales(
salesid integer,
listid integer,
sellerid integer,
buyerid integer,
eventid integer,
dateid smallint,
qtysold smallint,
pricepaid decimal(8,2),
commission decimal(8,2),
saletime timestamp)
row format delimited
fields terminated by '\t'
stored as textfile
location 's3://awssampledbuswest2/tickit/spectrum/sales/'
table properties ('numRows'='172000');
```

* Access Cross-Account S3 buckets by adding a policy to the Cross-Account S3 Bucket `"AWS": "arn:aws:iam::<redshift-account>:<role/spectrumrole>"`
* You may also add conditions to ensure the user agent is actually Redshift Spectrum `"Condition": {"StringEquals": {"aws:UserAgent": "AWS Redshift/Spectrum"}}`

* Redshift Spectrum supports many File Formats in S3, like Json, Avro, Parquet, Text, Grok, OpenCSV, ORC, Rcfile
* Columnar storage file formats like Parquet or ORC is reccomended
* You should compress your data files to improve performance with either `gzip`, `Snappy` or `bzip2`
* Redshift Spectrum dectrups data files that are encrypted with either `SS3-S3` or `SSE-KMS` (but it must have access to the KMS keys)
* Large files should be broken down into Many Smaller Files of Even Size, larger than 64MB

### Redshift Snapshots

Redshift Automated Snapshots are by default taken every 8 hours, and stored in S3 for 24 hours. A Redshift Automated Snapshot can be copied over as a Manual Snapshot if you would like to keep it long-term. Redshift Manual Snapshots can be taken at any time, and stored for any period of time. Redshift snapshots allows you to restore a cluster from a snapshot, or to restore a single table from a snapshot.

* You can exclude particular tables from Snapshots (such as a staging table). To create a No-Backup Table, include the parameter `BACKUP NO` when you create the table in Redshift

#### Automated Redshift Snapshots

* By default, Redshift takes a snapshot every 8 hours or after 5GB data changes per node
* Automated Snapshots have a default retention period of 24 hours
* To Disable Automated Snapshots, set the retention period to Zero
* You cannot manually delete an Automated Snapshot – only Redshift can do this (either Redshift deletes the Automated Snapshot at the end of the retetion period, or Redshift deletes the Automated Snapshot when you Disable Automated Snapshots or Redshift deletes the Automated Snapshot when you Delete the Cluster)
* Amazon Redshift always retains the latest Automated Snapshot, unless you Delete the Cluster or Disable Automated Snapshots
* If you want to keep and Automated Snapshot beyond the retention period, you can create a copy of the Automated Snapshot as a Manual Snapshot
* You can control when Automated Snapshots are taken with Automated Snapshot Schedules
* You can only create Snapshot Schedules with a frequency greater than once an hour or less than once every 24 hours

#### Redshift Manual Snapshots

* Manual Snapshot are store indefinetly by default
* Manual Snapshot created through the Console are stored for 365 days by default
* You can always edit the retnetiion period of an existing Manual Snapshot
* Redshift has a quota for how many Manual Snapshots you can create

#### Redshift Copying Snapshots to Another AWS Region

* You can configure Redshift to Automatically Copy Snapshots from a Source Region to a Destination Region
* You can Restore a Redshift Cluster in another Region from a Copied Snapshot
* You can copy either Manual Snapshots or Automated Snapshot to another Region
* A Redshift Cluster can only copy snapshots to **One** Destination Region at a time
* For Automated Snapshots, you can configure the Retention Period in the Destination Region – which will be separate from the Retionen Period in the Source Region
* The Default Retention Period for copied Automated snapshots is Seven Days

* If the Redshift Cluster is **KMS encrypted**, you must allow Redshift to use the KMS customer master key (CMK) in the Destination Region.
  * KMS Keys are specific to an AWS Region
  * For Redshift to Encrypt Snapshots in the Destination Region, you must configure a Grant for Redshift to use a master key in the Destination Region
  * You cannot Rename your Cluster, as the Cluster Name is part of the Encryption Context
  * In the Destination Region, create a Snapshot Copy Grant with KMS
  * In the Source Region, enable copying of snapshots and specify the name of the Snapshot Copy Grant
  * Before the Snapshot is copied to the Destination Region, Redshift Decrypts the snapshot using the master key in the Source Region and Re-Encrypts it temporarily using a randomly generated RSA key that Redshift manages internally. Redshift the copies the snapshot over a secure channel to the Destination Region. Redshift Decrypts the snapshot with the RSA key and the Re-Encrypts the snapshot using the KMS master key in the Destination Region

#### Restoring a Cluster from a Snapshot

* A Snapshot contains both Database Data and Information about the Cluster, including Nodes and Master User Name
* For the new Cluster created from the snapshot, you can choose configuration of nodes, AZ, maintenance track
* Monitor the restore progress with the `DescribeClusters` api call
* You cannot use a snapshot to restore an active cluster to a previous state restoring from a snapshot creates a New Cluster
* Restoring from a Snapshot allows you to Change the Node Types and Number of Nodes

#### Restoring a Table from a Snapshot

* You can restore a single Table from a snapshot to a Running Cluster
* Restore only One Table at a time
* Restoring a Table from a Snapshot is only possible if the cluster has not been resized since the Snapshot was taken

#### Sharing Snapshots across AWS Accounts

* You can share Snapshots across AWS Accounts
* If the snapshot is Encrypted with KMS, you must also share the KMS Key with the other AWS Account by creating a Grant with thier ARN

### Redshift Security

Access to Redshift is controlled at four levels

* Cluster Management, Creating, Deleting and Modifying Clusters is controlled through **IAM**
  * You can give access for `COPY` and `UNLOAD` commands by attaching a permissions policy to the Redshift Role you create in IAM
  * For Redshift Spectrum, you can also add policies to allow access to the Glue Data Catalogue or the Data Catalogue in Athena with `AWSGlueConsoleFullAccess` and `AmazonAthenaFullAccess`
  * To Restrict Access to IAM Roles for specific Redshift Database Users, you can add a Trust Relationship in the IAM Role Policy, here an example for `user1`: `arn:aws:redshift:us-west-2:123456789012:dbuser:my-cluster/user1`
* A **Cluster Security Group** controls access to the cluster. You can modify inbound access rules, to allow other AWS services, or external services access. Typically you would want to deploy your Redshift Cluster in a VPC
* Database access within your Redshift Cluster is controlled by **User Accounts** in the Redshift Database. You create users and superusers and manage permissions with `CREATE USER`, `CREATE GROUP`, `GRANT`, and `REVOKE` SQL statements
* Temporary Database Credentials and Single Sign-On through Custom Redshift JDBC and ODBC drivers. The JDBC or ODBC Drivers manages creation of database users, based on IAM authentication. You can use a SAML 2.0 Identity Provider (IdP) to manage access to Redshift ressources fir Federated Users.
  * To use Redshift with JDBC or ODBC driver, provide the database user name

### Loading Data into Redshift

* To verify that a load operation is complete, query the `STL_LOAD_COMMITS` system table
* To validate the data in S3 input files or in DynamoDB table, before they are actually loaded into a Redshift table, use the `NOLOAD` option in the `COPY` command. `NOLOAD` will display any errors that would have occured.
* You can apply Automatic Compression to data that you `COPY` into an empty table. You need to load at least 100,000 rows for the algo to see a pattern. Use the option `COMPUPDATE ON` with your `COPY` command to use Automatic Compression
* Manual Compression you can specify when you Initially Create a table (but Automatic Compression is recommended)

#### Loading Data from S3

* Use the COPY command `copy <table_name> from 's3://<bucket_name>/<object_prefix>' <authorization>;`
* With S3, optionally use a manifest file: `copy customer from 's3://mybucket/cust.manifest' iam_role 'arn:aws:iam::0123456789012:role/MyRedshiftRole' manifest;`
* The autorisation can be either an IAM role or (ACCESS_KEY_ID, SECRET_ACCESS_KEY)
* Specify the Compression Type when uploading compressed files `gzip`, `lzop`, or `bzip2`
* To COPY encrypted files into Redshift, Redshift supports the following types of Encrypion in S3: `SSE-S3` & `SSE-KMS` (here, Redshift automatically recognizes and loads files). Redshift also supports `CSE-C` using a Symmetric Master Key, but here you have to pass the Symmetric Customer Master Key used to encrypt the files into the `COPY` command like so: `copy customer from 's3://mybucket/encrypted/customer' iam_role 'arn:aws:iam::0123456789012:role/MyRedshiftRole' master_symmetric_key '<master_key>' encrypted;`

#### Loading Data from EMR

* You need to add the Redshift Clusters Public Key to each Node in your EMR Cluster, so that the EMR Nodes will accept the SSH connection from Redshift, when Redshift runs the COPY command
* You must also edit the EMR Security Groups for Master Nodes and Task/Core Nodes to Allow inbound traffic through SSH with TCP on Port 22. In Source, add the Redshift Cluster Nodes IP Addresses.
* Finally, run the `COPY` command:

```sql
copy sales
from 'emr://myemrclusterid/myoutput/part*' credentials
iam_role 'arn:aws:iam::0123456789012:role/MyRedshiftRole';
```

* You can also use the `COPY` command to load data from other remote hosts (like EC2 instances), by similarly opining them to SSH access, and also creating a Manifest File that you upload to S3

#### Loading Data from DynamoDB

* Make sure to set `READRATIO` to a low percentage, to avoid using all the RCU of the table and getting throttled
* Note that Redshift only supports String and Number scalar data from DynamoDB

```sql
copy <redshift_tablename> from 'dynamodb://<dynamodb_table_name>'
authorization
readratio '<integer>';
```

### Unloading Data from Redshift into S3

* Use the `UNLOAD` command to unload data into S3 `UNLOAD ('select * from venue') to 's3://<path> iam_role '<arn>;`
* `UNLOAD` automatically uses S3 Server Side Encryption `SSE-S3` when writing to S3
* You can also specify using `SSE-KMS`:

```sql
unload ('select venuename, venuecity from venue')
to 's3://mybucket/encrypted/venue_'
iam_role 'arn:aws:iam::0123456789012:role/MyRedshiftRole'
KMS_KEY_ID '1234abcd-12ab-34cd-56ef-1234567890ab'
encrypted;
```

* You can also use Client Side Encryption with you own key, that you give to your `COPY` command:

```sql
unload ('select venuename, venuecity from venue')
to 's3://mybucket/encrypted/venue_'
iam_role 'arn:aws:iam::0123456789012:role/MyRedshiftRole'
master_symmetric_key '<master_key>'
manifest
encrypted;
```

### Redshift Performance

Redshift uses Massively Parallel Processing for fast queries across the cluster.

* Data is stored as columnar storage, to support fast execution (avoids scanning)
* Block size is 1 MB
* Column Compression is enabled by default, and works automatically
  * You *dont need* index or materialised views
  * You load data into Redshift with the COPY command
  * It is not possible to change the compression after a table is created

### Redshift Durability

* Redshift uses replication to different nodes within the cluster.
* Data is also back up asynchronously to an S3 bucket in anther region.
* Automatic snapshots with a 1 day default retention period
* Redshift automatically detects and replaces failed nodes.
* Redshift will be breifly unavailable when a node is replaced.

Redshift is limited to a single availability zone!
If the AZ goes down, Redshift goes down as well.

#### Scaling Redshift provisions a new cluster

* While data is being move the the new cluster, the old cluster will be available for read
* The CNAME is flipped to the new cluster (a few minutes of downtime)
* Old cluster shuts down when all the data is copied to the new cluster

### Sort Keys in Redshift

While creating a table, you can define one or more sort keys.
Data is stored on disk in sorted order.

* Like an index
* Range queries are fast
* Min and max values are automatically stored by Redshift as metadata

Choose a sort key that matches you common queries for best performance

* Querying based on recency? Have Timestamp as sort key
* Filtering based on product category? Have category_id as sort key
* Joining table often with another? Have the join set as both sort key and distribution key
  * The distribution key will enable all of the data with that key be store together

* Single sort key: One sort key, like 'customer_id'
* Compound sort key: Two or more sort keys, keys are sorted in this order!
* Interleaved sort key: if you often filter on different sort keys. Use compression internally on map.

### Redshift Resizing

Two ways of scaling:

* **Elastic Resize**: Quick, but limited to Adding or Removing Nodes of same Type
* **Classic Resize**: Slow (down for hours to days), but can change node types. The cluster will by Unavailable during a Classic Resize.
  * Way to do it: Use snapshot command, restore it and resize, to keep cluster available
* **Snapshot, Restore & Resize**: By making a Snapshot and restoring a New Cluster from the snapshot, you can choose to Resize the New Cluster, while keeping the Old Cluster running.

New things for 2020:

* RA3 nodes: scale compute and storage independently
* Redshift Data Lake Export: Unload Redshift queiry into S3 in Parquet format

### Redshift Distribution Styles for Distribution Keys

* **Auto**: Figures it out based on data size (if you don't now)
* **EVEN**: Rows are split between slices in a round robin
  * Good when there are no joins. Data is not stored together.
* **KEY**: Matching values are on the sort keys are stored together (ie. records with key 4 will always be stored on the same node)
  * Great for when you do joins on large tables, and know which key you will use for joins
* **ALL**: Stores a copy of the data on every_single_node. Very resource intensive.
  * Good for small tables that a frequently used in joins, and rarely gets updated with new data.

### Importing and Exporting Data to Redshift

#### Import with COPY command

* Reads from multiple streams in parallel
* Reads from either S3, EMR, DynamoDB or remote hosts
* S3 ca use manifest file (schema and where files are in partitions)
  * Can provide prefix: COPY from S3;;//prefix
  * or with manifest: manifest file contains the location of data
  * Must also have IAM role

* Use COPY to load large amounts of data
* If your data *is already in Redshift* in another table, use INSERT INTO ... SELECT or CREATE TABLE AS

* COPY can decrypt data as it is loaded from S3

* COPY supports Gzip, Lzop and bzip2 compression

* Automatic compression option, analysis data being loaded and figures out how to store it on Redshift

* Special case: Narrow Table: should  be loaded with a single COPY transaction if possible. Dont break it up.

* Do things in one copy command if possible

##### Copy a Redshift snapshot to another region for backup, from a KMS-encrypted cluster

* In the **destination** AWS region:
  * Create a KMS key
  * Specify a unique name for you snapshot copy grant
  * Specify the KMS key ID for which you are creating the copy grant
* In the **source** AWS region:
  * Enable copying of snapshot to the copy grant you just created

##### DBLINK allows you to connect Redshift to PostgreSQL (possibly in RDS)

Good way to copy and sync data between PostgreSQL and Redshift

* Must be in the same availability zone
* `CREATE EXTENSION dblink`

#### Export with UNLOAD command

* Unloads from a table into files in S3

#### Enhanced VPC routing, forces COPY and UNLOAD through VPC

* Makes sure that data does not go through the internet!
* Needs setup

### Redshift integration with other AWS serivces

Redshift integrates with

* S3
* DynamoDB (load data from Dynamo to Redshift with the COPY command)
* EMR (import data from EMR to Redshift on SSH with COPY command)
* Data Pipeline (schedule data movement into and out of Redshift)
* Database Migration Serivce (helps you migrate into Redshift)

### Redshift Workload Management (WLM)

Creates queues of queries and prioritises them

* Can prioritise short fast queries over long slow queries
* Can be configured via console, CLI or AP

#### Concurrency Scaling

Automatically add cluster capacity to handle increase in concurrent **read** queries.
WLM queues manage which queries are sent the the concurrency scaling cluster.
Great for really spiky queries.

#### Automatic Workload Managment

By default you have five queues.

Large queries will be set to lower concurrency, while small queries are set to higher concurrency.

* You can set a priority value for queries.
* Assign a set of user group to a high (or low) priority group
* Set a query group label, a tag that determines which queue it goes to
* Set up Query monitoring rules -> if a query in the 'short-running queue' exceeds 60 seconds, then abort

#### Manual Workload Management

* Superuser queue with concurrency level 1
* Define up to 8 queues, up to concurrency level 50
  * Define concurrency scaling mode, concurrency level, memory and timeout
  * Can also enable query queue hoopping

#### Short Query Acceleration (SQA)

Prioritise short running queries over longer-running ones

Can be used in place of WLM queues for short queires

* Works with Machine Learning to predict a queries execution time
* Works with CREATE TABLE AS (CTAS), read only queries (SELECT statmeents)

#### VACUUM reclaims space from deleted rows

Can take a really long time, and is super intensive.

* VACUUM FULL
* VACUUM DELETE ONLY
* VACUUM SORT ONLY
* VACUUM REINDEX (great for interleaved sort keys)

### Redshift anit-patterns

* Small dataset. Redshift is for big datasets
* OLTP. Redshift is good for OLAP
* Unstructrued data. ETL data first with EMER
* BLOB, Redshift is good for database style data

## RDS

Small data! And OLTP.

* Aurora
* MySQL
* PostgreSQL
* MariaDB
* Oracle
* SQL Server

Migrating from RDS to Redshift with Database Migration Service.

All RDS databases are ACID (Atomicity, Consistency, Isolation, Durability) compliant.

### Aurora

MySQL (5x faster) and PostgreSQL (3x faster) compatible
64TB storage per database instance
Up to 15 read replicas
Continuous backup to S3

* VPC network isolation
* At rest encrypt with KMS (also backup, snapshot and replicase)
* In transit encryption with SSL
