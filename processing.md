# Processing

* Lambda (Sm)
* Glue (Sm)
* EMR (Big)
* ML (Sm)
* Data Pipeline (Sm)

## Lambda

* Lots of different triggers
* For Kinesis Data Streams trigger: Lambda actually polls stream periodically

* Default 1000 concurrent sessions (can be increased with AWS)
* Max timeout is 900 seconds (15 min)
* Auto retry 3 times

### Loading into Redshift: use COPY command

* Better: have a DynamoDB to checkpoint, and help with batching
* Dynamo keeps track of state
* COPY a bigger batch for more Redshift efficiency

### Getting data from Kinesis

* Max 10,000 records
* Lambda max payload is 6MB
* Too large a batch can cause timeouts! And stall the shard!!
* You can specify a batch size for Kinesis, and adjust lambda timeout
* Lambda process shard data synchronously

## AWS Glue

Define table definitions and perform ETL

* Discovery and definations of tables
  * S3 data lakes
  * RDS (relational)
  * Redshift
  * Most other SQL (MySQL, etc)
* ETL jobs (Apache SPARK under the hood)
  * On triggers
  * On schedule
  * On demand

Billed by minute for Crawler and ETL jobs

Development endpoints for developing ETL are charge by the minute

Should not be used with streaming data. Use for batched data.

### Glue Crawler / Data Catalog

Automatically go through new data in S3, and discover schema

The Glue Crawler will populate the Glue Data Catalog.
The table definitions are stored in the Glue Data Catalog.
These table definitions can be sent to other services:

* Redshift Spectrum
* Athena (and the QuickSight)
* EMR

Glue crawler will extract partitions based on how S3 is organised.

* Query much by time?: Do yyyy/mm/dd/device in S3
* Query much by device?: Do device/yyyy/mm/dd

### Glue Security

* Glue Data Catalogue can be encrypted with KMS
* You can configure the Glue Crawler to only access SQL databases with SSL through JDBC
* Connection passwords for SQL databases can be encrypted with KMS

### Glue + Hive (SQL like queries)

Glue integrates with Hive.
You can use Glue as your Hive metastore.
Or you can import your Hive store into Glue.

Glue can provide metadate to Hive on EMR.

#### Glue ETL (Apache Spark under the hood)

* Automatic code generation (mmmmhhh)
* Either Scala or Python
* Encryption, Server-side using KMS at rest * SSL in flight
* Can provision additional Data Processing Units (DPU) to increase performance of the underlying Spark job

Generate code and build upon that.
Or Provide existing Spark or PySpark scripts

* S3
* JDBC
* Redshift
* RDS
* Other SQL databases
* Or Glue Data Catalog

Glue Scheduler schedules jobs
Glue Triggers automate based on events

#### Transformations

Built in, bundled transformations:

* DropFields, DropNullFields, remove null fields
* Filter, filter records
* Join, to enrich data
* Map, add fields, delete fields, external lookups

Machine Learning Transformations:

* FindMatches ML, identify duplicate or matching records in your dataset, even when fields don't match exactly

Format conversions:

* CSV, JSON, Avro, Parquet, ORC, XML

Apache Spark transformations (example: K-Means clustering)

#### Glue Development Endpoints

* Develop ETL scripts using a notebook
* Create ETL job that uses that scripts
* Endpoint in a VPC controlled by security groups
  * Use Apache Zeppelin on your local machine or on EC2
  * SageMaker notebook
  * Terminal window
  * PyCharm professional edition
  * Elastic Ip's to access a private endpoint address

#### Running Glue Jobs

Running on time based schedule (like CRON)

* Job bookmarks, lets you pick up where you left of
  * Preserve state
  * Only pick up new data
  * Works well with S3 data lake
  * Can also work with JDBC, relation databases (IF PRIMARY KEYS ARE SEQUENTIAL)

* CloudWatch Events
  * When ETL succeeds or fails: fire off Lambda, SNS, EC2, data pipeline

## Elastic Map Reduce

Elastic Map Reduce is a managed cluster platform.

* Runs Apache Hadoop
* Use related frameworks like Apache Spark, Apache Hive and Apache Pig

A cluster is comprised of:

* One **Master Node** (Up to two more on standby with Amazon EMR 5.23.0)
  * Runs YARN ResourecManager service
  * Runs HDFS NameNode service
  * Monitor the progress, see logs, of the cluster by SSH into master node
* One or more **Core Nodes**
  * Runs Task Tracker daemon, Data Node daemon to coordinate HDFS storage
  * Holds storage and does computation
  * Failover system with HDFS, to recover data if core node fails
  * You can *resize* core nodes while cluster is running
  * Removing or adding core nodes while running is generally a bad idea
* Optional **Task Nodes**
  * Just computation (no long-term storage), runs tasks
  * Great with spot instances, or use when compute needs scaling
  * Spot: AWS EMR default functionality protects jobs against spot instance termination. Application master process controlling jobs runs on core node, and stays alive

* Create EMR clusters on AWS Outposts (from EMR v. 5.28.0)

Note: You can only attach EBS volumes before starting the cluster.
If you remove an EBS volume from an underlying EC2 instance, while the cluster is running,
EMR will intepret that as a failure and replace both the EC2 instance and storage.

When you are working with data in S3, because you want to ensure high durability of the data, in case the AZ or cluster goes down, it is best to store the data in S3, but hold a `warm copy` of the data in HDFS. You will incur fewer networks costs. You could do this with a priliminary S3DistCp, to copy data from S3 to EMR before you run your job.

### Creating a Cluster on AWS EMR on console

* Open EMR from the console
* Give cluster a name
* Optional: Select an S3 bucket to store clusters logs
* Choose launch mode:
  * Cluster (defualt): manually terminate cluster when done, or leave it on!
  * Step execuion: add a series of steps (streaming, Hive, Pig, Spark or custom JAR), terminate when done

* Choose EMR version (latest is 5.28.0)
* Choose application package: Core Hadoop / HBase / Presto / Spark
* Optional: Use AWS Glue Data Catalogue as the external Hive metastore for Hive

* Choose type of instance (from m1.medium to p3.16xlarge)
* Select number of instances (1 or more)

* Select EC2 key pair for SSH access to master node
* Choose IAM roles for EMR and EC2 machines or create new automatically (default)

#### Advanced Settings

* Use multiple master nodes (3) for improved cluster availability (reqires external database for Hue and Hive)
* Edit configuration files for software / load JSON from S3
* For Steps: Enable concurrent executing of steps

* Set uniform instances or instance fleets (different types of instances in cluster)
* Select VPC (network) and subnet
* Set root device EBS storage per machine (10GB - 100GB)
* Set number of instances across Master/ Core Nodes / Task Nodes
* Choose On-demand or Spot

* Enable Logging and/or Debugging to be sent to S3 bucket
* Enable cluster termination protection
* Add tags
* Enable EMRFS consistent view
  * Name DynamoDB table
  * Select number for retries & retry period on S3 if object in S3 is inconsistent
* Set bootstrap actions (start-up scripts to run on each node)

* If you are seeing consistency problems with writes and reads to S3 (like from Hive), you need to use EMRFS in your cluster to fix this

* Select Security Configuration
* Select EC2 security groups for Master Node and Core/Task nodes

* Optionally: (Default) Block Public Access – except port 22 for SSH

#### Create Security Configuration

* Enable **S3** Encryption-at-rest for EMRFS data in S3
  * SSE-S3 – S3 creates and manages keys. Files are encrypted and decrypted in S3
  * SSE-KMS - KMS creates and manages keys. Files are encrypted and decrypted in S3
  * CSE-KMS - KMS creates and manages keys. Files are encrypted and decrypted on EMR cluster
  * CSE-Custom – Custom JAR with key provider. Files are encrypted and decrypted on EMR cluster

* Enable **Local Disk** Encryption-at-rest
  * KMS provides key, choice of EBS encryption or LUKS encryption
  * Custom JAR with key provider. LUKS encryption only

* Enable in-transit (in-flight) encryption with TLS
  * Reqires PEM certificates or custom JAR certificate provider

* Enable Kerberos authentication
  * Choice of Kerberos installed on master node of cluster OR external Kerberos KDS server

* Use IAM roles for EMRFS

### EMRFS

Uses S3 as the storage for EMR cluster – **Consistent View in Amazon EMR release version 3.2.1**

* Consistent View uses a DynamoDB table to track consistency on S3
* Dynamo tracks file metadata
* EMRFS will retry 5 times by default, when verifying consistency
* You can enable consistency notifications to CloudWatch Metrics & SQS

### EMR Encryption

AWS EMR versions 4.8.0 or later, you can use *security configurations* to more easily apply encryption settings.

* EMRFS (S3) supports both server side (S3, using S3 or KMS) and client side (EMR cluster, using KMS or custom) encryption
  * Server Side: Either SSE-S3 or SSE-KMS – objects are encrypted by S3 using either S3 managed keys or KMS managed keys. Custom keys are not supported.
  * Client Side: EMR encrypts data on cluster. Use either KMS or your own custom key. Objects are encrypted before uploaded and decrypted after download.

### Spark

### EMR Applications

#### Hadoop

Hadoop is the core of AWS EMR.
It is the framework that enables jobs to be split out across a cluster of machines.

#### Flink

Flink is a streaming data engine.
You can use Flink to write Java (with DataStream API) or Python on streaming data.

* Process Unbounded (no end) or Bounded (start and end) data
* Works with Kinesis Analytics (choose either SQL or Flink runtime)
* Flink has a connector to consume from an AWS Kinesis Data Stream
* Flink supports event time for out-of-order events
* Exactly once processing of events
* Flink API for writing both streaming and batch

#### Ganglia

Ganglia is a monitoring tool for clusters.

* You can use Ganglia to see memory usage, CPU, and network traffic
* Visualies both Hadoop and Spark metrics

#### HBase

HBase is a massivly scalable noSQL datastore - a non-relational database.

* HBase is based on Googles Bigtable (only accessible to public as a cloud service)
* It runs on top of HDFS or EMRFS with AWS EMR
* HBase provides random read/write access to very very large datasets
* Items stored in HBase can be large, much larger than 400kb Dynamo limit
* Strictly Consistent Reads and Writes
* HBase is included in AWS EMR 4.6.0 and later
* Use S3 as the underlying datastore in AWS EMR 5.2.0
* Read-replicas with hive in AWS EMR 5.7.0
* Snapshots in AWS EMR 4.0

* HBase is really fast in **getting** a row from a *Row Key*
* HBase is also pretty fast a **scanning** rows, if the *Row Keys* are next to each other (alphabetically)
* HBase is not so fast at **scanning** entire table, and then filtering out based on a condition

* HBase integrates tightly with Hive to enable SQL queries (batch style, for OLAP, slow)
  1. Log in to the cluster master node with **HUE**
  2. HBase Browser and create a new external Hive Table of the data in HBase
  3. After creating the table in the Hive Metastore, you can write SQL queries with Hive on the HBase data
  4. With mulitple external tables from S3, HBase and HDFS in the Hive Metastore, you can do joins!

* HBase integrates tightly with Phoenix to enable SQL queries (lower latency, lower throughout, fast)
  * Phoenix is an open source SQL skin for HBase
  * Phoenix converts your SQL queries to HBase API calls (scans mostly)
  * Doesn't create new tables, does performance optimisations
  * Alternative to useing the HBase API directly

* HBase integrates well with S3
  * HBase can store Snapshots in S3
  * HBase can store StoreFiles and Metadata in S3
  * HBase can store read-replicas in S3

* You can use HBase for OLTP queries from a web application
  * Another good alternative for OLTP queries is DynamoDB

#### Hive

Hive is a analytics package, that uses Hive scripts in Hive QL (like SQL) to analyse data.

Hive works with HBase, S3, and HDFS as underlying datastores.

* Higher Level Language – Avoids complexities of writing Tez based jobs, based on DAGs or MapReduce programmes
* S3: For using Hive with S3, ACID is not supported, as S3 is eventually consistent
* Hive Metastore is by default run on MySQL on the clusters Master Node
  * Optional External Metastore: **AWS Glue** – in AWS EMR v. 5.8.0
  * Optional External Metastore: **RDS or Aurora**
* Hive JDBC Driver available for connecting to BI tools like Excel, MicroStrategy, QlikView, Tableau...

If you want to use Hive to query data that is currently stored in a large Relational Database, you can use Apache Sqoop, to sqoop up the data in the relational database and transfer the data into HDFS.

#### Phoenix

Phoenix is used to translate SQL (or JBBC) to HBase API.

Phoenix is used exclusivly with HBase.

#### HUE - Hadoop User Experience

Hue is the frontend for applications running the cluster.

* Access Hive editor, Pig editor, File Browser, Job Browser
* To launch, set up SSH tunnel with port forwarding, and access through web browser
* By default, user information and query history is stored in a local MySQL database on master node
  * Optional: external MySQL database in RDS + a configuration file in S3

#### Notebooks (EMR Notebooks | Jupyter Hub)

EMR Notebooks a Jupyter notebooks on Spark clusters.

* Create and open notebooks through AWS Console
* EMR Notebooks are saved in S3 – seperate from the cluster
* New or existing notebooks can be attached to a cluster, or relocated
* Use Git with Notebooks for version control

#### Apache Mahout

Mahout is an (out-of-fashion) machine learning framework, that works with Hadoop and Spark

#### MXNet

MXNet is a machine learning framework for building neural networks. Works with AWS EMR.

* MXNet can be used to create Convolutional Neural Networks

#### Tensorflow

Tensorflow comes bundled with some EMR configurations

* Tensorflow can be used to create Convolutional Neural Networks

#### Oozie

Oozie is a workflow scheduler for Hadoop. Access through the HUE Oozie application

#### Pig

Pig allows you to write Pig latin - a high level language for querying data on EMR

Pig integrates with S3 in three ways

* Pig allows you to write directly to HCatalogue tables in S3
* You can submit work to EMR from the EMR console with Pig scripts stored in S3
* You can load custom JAR scripts with Pig, JAR scripts store in S3, with the `REGISTER` command

#### Sqoop is an application for transferring data between relational databases and Hadoop

You can use Sqoop to transfer data from a relational database into EMR for analysis with Hive

### S3DistCp or S3 Distributed Copy for HDFS & EMR

S3DistCp is a tool for copying large amounts of data from S3 into HDFS, or from HDFS into S3.
It uses MapReduce to do copying in a distributed manner.
It is great for copying a very large amount of objects.
It can copy objects across buckets and across different AWS accounts.

* A great use case for S3DistCp is if your are seeing a lot of network traffic, reading and writing files to S3 from your EMR Cluster
  * Add a preliminary S3DistCp so that you copy all the relevant data over to your EMR cluster before beginning the analysis job
