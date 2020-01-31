# Collection

* Kinesis (Big)
  * Kinesis Data Streams
  * Firehose
* SQS (Sm)
* IoT (Sm)
* Database Migration Service (Sm)
* Direct Connect (+ site-to-site VPN) (Sm)
* Snowball (x-Sm)

## Kinesis Data Streams (KDS)

* A managed alternative to Apache Kafka

* Stream of shards (partitions)
* Retention is 24h by default (can be increased to 7 days)
  * The Maximum retention period for Kinesis Data Streams is 7 days
* Able to Replay/Reprocess data within retention period
* Multiple application can consume data from one stream
* Data in shards are immutable, shards append only
* Billing by shard

### Kinesis Streams Record

A record consists of three parts:

* Data blob (up to 1mb)
* Record key (user provided, should be well distributed to avoid hot shards)
* Sequence number (added by Kinesis)

### KDS Limits

* Producers are limited to 1Mb/s or 1000 messages/s, per shard
* Consumer classic is limited to 2Mb/s or 5 API calls per second, per shard, across all consumers
  * With one consumer application, this creates 200 ms latency (1 app * 1s/5 calls)
  * With two consumer applications, 400 ms latency (2 apps * 1s/5 calls)
  * With three consumers applications, 600 ms latency (3 apps * 1s/5 calls)
* Consumer Enhanced Fan-Out, 2Mb/s per shard (push model)
  * Up to 20 consumer applications (default max of 5)
  * With one shard: 2Mb/s throughput
  * With ten shards: 20Mb/s throughput (in aggregate)

### Scaling KDS

* Scale by splitting or merging shards
* Scaling is a manual process
* Can implement via a lambda

* Takes time, resharding cannot be done in parallel
* Takes a few seconds (~30) to spolit
* For 1000 shards, it takes 30K seconds (8.3 hours) to double to 2000 shards
* Limits to how quick/much you can scale up and down

#### Split Shards

* Increase stream capacity
* Split to divide a 'hot shard'
* Old shard is closed after splitting, and deleted after retention ends

#### Merge Shards

* Decrease cost
* Group two shards with low traffic
* Old shards are closed after merging and deleted after retention period

### Kinesis Security

* Access Contal via IAM policies
* Encryption in flight with HTTPS endpooints
* Encryption at rest using KMS (you cannot use an internal key service, you must use KMS with Kinesis)
* Client side encryption must be manually implemented
* VPC Endpoints for access to Kinesis within VPC

### Kinesis Agent

* Use the Kinesis Agent to collect server logs
  * Don't use the Kinesis agent for anything else

## Kinesis Firehose

* Get data with the Kinesis agent, or use `PutRecord` or `PutRecordBatch` in your own application
  * Native integration with Kinesis Data Streams, AWS IoT, CloudWatch Logs and CloudWatch Events
  * Not all services can integrate with Kinesis Firehose. Firehose only supports native integration with KDS, IoT and CloudWach. For anything else you will have to write custom code (like a Lambda)
  
* Load data into 4 destinations:
  * Redshift
  * S3
  * ElasticSearc
  * Splunk

* Records are text strings
* The maximum size of a record (before Base64-encoding) is 1000 KB.

* Auto scaling, no server
* Supports many data formats
* Can convert JSON to Parquet or ORC if destination is S3
* Transformations with Lambda (ex: CSV => JSON)
* Supports compression with GZIP, ZIP and SNAPPY if destination is S3
* Supports compression with GZIP if destination is Redshift (because `COPY` command supports only GZIP, lzop or bzip2)
* Doesn't support reading with Kinesis Client Library or Spark!!

* Can save source records automatically to S3
* Can save transformation failures and/or delivery failures to S3

### Buffer Size Limits

* Buffer time is 60 to 900 seconds (15 min)
  * The max buffer time for Firehose is 15 min
* Buffer size varies by destination:
  * S3: 1MB to 128MB
  * ElasticSearch: 1MB to 100MB

## SQS

SQS is a queue service to help de-aggreagate your applications

* Max 120,000 inflight messages at a time
* Messages are text
* Batch is max 256kb or max 10 messages

* Message body is a string, max 256kb
* Each message can have key/value metadata (attributes) attached
* Messages can be batched when reading and writing (pool) for greater throughput. Up to 10 messages at a time

* Consumers delete messages after reading (ie. only 1 consumer per message)

* Default retention is 4 days, up to 14 days, down to 1 minute

* Low latency, <10 ms

* On delivery you get a message identifyier and a MD5 hash of message body

### SQS security

* Encryption in flight with HTTPS endpoints
* SSE, Encryption at rest using KMS (note only body is encrypted)

* IAM policies to grant access
* Also: SQS Queue access policy, fine grain control of access / ip

### SQS FIFO

* Guarentee order of messages, first-in-first-out
* Up to 3,000/300 messages per seconds (with/without batching)

### SQS Extended Client

* Use S3 bucket as a companion to send files larger than 256kb
* AWS built java library

## IoT Core

* IoT Thing (each device)
* Thing Registry (a registry for all the things)
  * Each device has a unique Id, and optional metadata
  * Create X.509 certificates for auth
* Device Gateway (autheticates and routes messages to/from things)
  * Supports MQTT, Websockets and HTTP 1.1
  * Serverless, scales to billions of devices
  * Authenticates with X.509 (or AWS Sigv4 or custom tokens)
  * Mobile apps: Cognito
  * Web: Federated identities or IAM
* Message Broker (recives messages and routes them, like SNS topic)
  * Could send message to IoT Rules Engine
  * Could also send message to Device Shadow
  * Pub/sub, low latency
  * Published into topics, like SNS
  * Can have multiple subscribers to a topic
* IoT Rules Engine
  * Rules are defined on the MQTT topics
  * Rules are when things are triggered
  * Actions are what it does
  * Can do actions triggering many things: Lambda, Dynamo, Kinesis, SNS...
* Device Shadow (holds state for an IoT thing)

* Devices are ruled by IoT Policies
  * JSON document attahced to groups or things
  * Only for devices!
  * Attached to X.509 certificates

* IoT Topics may have diffrent Rules in the Rules Engine
  * These IoT Rules Actions can interface with many AWS services

## Database Migration Service

Move database from on-prem/other clouds

Move between same databases (oracle-oracle) or convert

* Continous Data Replication using CDC
* Replicate data with DMS software running on an EC2 instance

* Sources: Oracle, MS SQL, MonoDB, SAP, Aurora, Azure, S3, more...
* Targets: On Prem, EC2 instance DBs, RDS, Aurora, Dynamo, Redshift, Kinesis Data Streams, more...

* Use Schema Conversion Tool (SCT) to convert schema from one to anther
  * Example: OLTP to MySQL
  * Example: OLAP to Redshift

* Create EC2 replication instance, up to r4.8xlarge
* Allocate storage, set up in VPC
* Can be public accessible (for connecting to on-prem, outside VPC)

* Create source endpoints and target endpoints
  * Use KMS master key for encryption

* Create database migration tasks (replication jobs)
  * Migrate existing, replicate ongoing, only changes

* If you have an on-premse database that powers an application for OLTP purposes, but you would like to do analysis on that table, you can use the Database Migration Service to replicate that database into RDS.
  * On RDS you can then do all of your OLAP queries
  * With Database Migration Service you can configure it to migratie the existing data in the database and also replicate ongoing changes

## Direct Connect

* Sets up networking between on-prem and VPC
* Dedicated 1 Gbps or 10 Gbps _dedicated_ private network

* Need Virtual Private Gateway on you VPC

* Failover, either two Direct Connect, or site-to-site VPN, or both

* Takes some time to set up
* If multi region: use direct connect gateway to connect to diff. regions

## Snowball

* 50TB, 80TB
* Uses KMS 256bit encryption

* Optionally comes with compute, EC2 AMI or Lambda function

* Takes standard shipping back and forth
* If data transfer would take >1 week, you can use Snowball
