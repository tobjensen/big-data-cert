# Storage

* S3 (Md)
* Dynamo (Md)

## S3

* Objects must have a key, (full path)
  * <my_bucket>/my_folder/my_file.txt
* Max size is 5TB
* if uploading more than 5GB, must use multi-part upload

### S3 Consistincy model

S3 has read after write consistency. After deletes or updates, S3 is eventually consistent.

* Read after write consistency aft PUTS of new object
* However! If you just did a GET before PUT, you will get 404 (eventually consistant)
* Eventual consistency for DELETE and PUTS (updates) on existing objects (you might get a 200 on an object you just deleted)

### S3 Storage Tiers

* Standard (multi Az)
* IA, infrequent access (ie. analytics once per month), multi az
* One-Zone IA, infrequent access, lower availability, store data you can recreate
* Intelligent tier, auto-tier'd between standard and IA

* Glacier, low cost archiving. (Vaults instead of Buckets)
  * Retrival expensive (1-3 min to hours)

### S3 Lifecycle Rules

* Automate moving between storage tiers
* Transition actions (move to glacier after 60 days)
* Expiration actions (delete .log files after 30 days)

### S3 Versioning

* Enabled at bucket level
* Protects against deletes
* Easy roll back to prev. version

* Note: objects created before versioning has version id null

### Cross region replication (or same region replication)

Replicate bucket across regions (ex: us-east-1 to eu-west-1)

* Must enable versioning on both buckets
* Can be in different accounts
* Copying is async
* Must have IAM permissions
* Located in Managment -> Replication
* Can filter on prefix or tags, change storage tier

* Replicates objects within 15 min SLA (but usually much faster: 1Gbps)

### S3 ETag (MD5 hash of file content)

* <5GB: it is MD5
* \>5GB: MD5 of the concatenated MD5 hashs of the multiple files

### S3 Performance

*Previously*, in order to have good partition allocation, have random prefix.
*Now*, you can get 3500 RPS for put and 5500 RPS for GET on EACH PREFIX

* Use Multipart object for parralilised upload

* CloudFront to cache S3 objects around world

* S3 Transfer Acceleration, using edge locations,
  * Write to an S3 bucket close to you, then AWS replicates internally

* If using SSE-KMS you may be limited by default KMS throttle limits

### S3 Security & Encryption

* SSE-S3, S3 makes and manages keys (AES-256)
  * header: `x-amz-server-side-encryption: AES256`
* SSE-KMS, KMS makes and manages keys (more control & logging)
  * header: `x-amz-server-side-encryption: aws:kms`
* SSE-C, customer provided keys, you send key with the put/get request in header
  * header: data key, must be HTTPS
* CSE, you do you.
  * Optionally with AWS Java library: Amazon S3 Encryption Client, or Encryption SDK

Encryption in flight using SSL/TLS (HTTPS)

#### S3 CORS (Cross origin resource sharing)

Allows you to limit the number of websites that can request your files in S3.
Can be a problem when testing website on *localhost*

#### S3 Access Logs

Log all access to S3 Buckets. Data can be analysed in tools, incl. Athena.
All log request can be saved to a logging bucket

#### S3 Access and Security

User based: IAM Policies

Resource based:

* Bucket Policies (allows cross account)
  * JSON based, allow deny list
* Object Access Control List (ACL) - fine grain
* Bucket Access Control List (ACL) - less common

Default encryption can be enabled as a bucket wide setting

Networking supports VPC endpoints (avoid public internet)

Can use MFA

Can use Signed URLs, time-limited links that expire

### Glacier

Archives are stored in Vaults. Vaults are a collection of archives.

Vault Access policy is similar to bucket policy

Vault Lock Policy is IMMUTABLE, (ex forbid deleting archive within 1 year)

1-3 min to 5-12 hours retrival

If you are using Glacier for complienace purposes, to store data for years and not allow anyone to delete it:

* Immidiatly replicate all source data into Glacier from day one
* Implement a Vault Lock Policy to forbid deleting from the archive

## Dynamo DB

Manged database across 3 AZ, scaled massivly, 100s TB, low latency

* Each Table has a PRIMARY KEY (decided at creation time)
* Max item size is 400 KB
* Supports many data types

### Primary Keys

* Option 1: Partition key ONLY
* Option 2: Partition key + Sort Key

Choose a good partition key that is distributed well, to avoid hot keys

### Calculate RCU & WCU

Throughput can be temporarily using burst credit.
Backoff using exponetial backoff if hitting `ProvisinedThroughputExceededException`

Avoid hot partitions:

* Exponetial backoff
* Distribute partition keys
* Use DAX to solve the issue with caching

#### WCU

One WCU equals 1 KB of write per second

Formula is `write_count_per_second * item_size_rounded_to_1kb`

* Example: If writing 10 objects of 2Kb per second => 20 WCU
* Example: If writing 6 objects of 4.5Kb (gets rounded up) per second => 6*5 = 30 WCU
* Example: If writing 120 objects per _minute_ of 2 kb => 4 WCU

#### RCU

Eventually vs. Strongly consistent: by default you get eventually consistent reads

One RCU equals 8KB of eventually consistent reads or 4KB of strongly consistent

Formula is `read_count_per_second * (item_size_rounded_to_4kb / (4kb | 8kb))`

* Example: 10 strongly consistent reads per second of 4KB (`10*(4kb/4kb)`) => 10 RCU
* Example: 16 eventually consistent reads per second of 12KB (`16*(12kb/8kb)`) => 24 RCU
* Example: 10 strongly consistent reads per second of 6 KB (`10*(8kb/4kb)`) => 20 RCU

### Dynamo Partitions

Internally, a partition has a max of 10GB data and 3000 RCU / 1000 WCU

RCU and WCU are spread evenly between partitions
This means that you can have hot partitions if keys are not well distributed

### Dynamo writing, deleting, reading

PutItem - write item to Dynamo
UpdateItem - possibly do atomic counters
Conditional writes – no performance implications

DeleteItem - delete an individual row
DeleteTable - delete everything

BatchWriteItem - up to 25 Put/Delete

* up to 16MB in total
* Writes done in parallel
* Parts can fail (ex. use exp. back-off)

GetItem - Primary Key (either HASH or HASH-RANGE)
BatchItem - up to 100 items & 16MB, in parallel

Query

* Only on _partition key_ and _sort key_
* Up to 1MB
* Filtering of results happens client-side (ex. filter on attributes)

Scan

* Returns 1 MB of data at a time
* Can read entire table
* Use pagination
* For faster performance, use parallel scans

### Local Secondary Indexes & Global Secondary Indexes

* Local Secondary Index:
  * Alternate range key for table, it is _local to the hash key_
  * Must be defined at table creating
  * If your partition key is _userId_ and sort key is _gameId_, a local secondary index could be _avatarId_
  * Attribute for LSI must be a simple scalar type (string, num, bool)
  * Helpful if you need to query on values other than sort key

* Global Secondary Index
  * Speed up queries on non-key attributers
  * Create brand new table based on existing table
  * _Redefine partition and sort key_
  * Needs own RCU and WCU

### DAX - Dynamo Accelerator, cache

* Writes go through DAX to Dynamo
* Micro second latency on cached reads
* 5 minute TTL be default
* Secure, encryption with KMS
* Sits in front of Dynamo

### Dynamo DB Streams

* If enabled, is the changelog (deltas) of your DynamoDB table
* Use Streams to send events to either KCL (Kinesis Adaptor) or Lambda
  * Example: React to events in real time: send email to new users
  * Example: Create derivative tables/views
  * Example: Insert data into ElasticSearch
  * Example: Implement Cross-Region Replication

* Stream has 24 hour retention time
* Batch size, up to 1,000 rows/ 6mb

* Use the KCL to directly consume Dynamo DB Streams
* Need to add a Kinesis Adapter
* Interface and programming is same as Kinesis Streams
* When KCL reads from Table, it uses another Dynamo DB table for checkpointing
* Alternative to using Lambda

### Dynamo TTL

* Time to live
* Does not use WCU/RCU or slow down table
* Background task run by dynamo
* Typically deletes within 48 hours of expiration
* Deleted items are also deleted in global- & local secondary indexes

* Dynamo STREAMS can help you recover expired items (within the 24 hour retention period)

* TTL is enabled by ROW

* Add expiration as a unix timestamp on row
* In Dynamo go to -> enable TTL, and select the expiration attribute

### Dynamo DB Security & Other Features

* Use Database Migration Service (DMS) to migrate from (Mongo, Oracle, MySQL, S3)

* Launch local Dynamo on your computer for development & testing

#### Security

* VPC endpoints (Gateway) to access Dynamo without internet
* IAM to regulate access
* Encryption at rest with
  * Default, AWS owned CMK (no charge, default)
  * KMS managed CMK in KMS
  * Customer managed CMK in KMS
* Encryption in flight with HTTPS/TLS/SSL

#### Backup and restore

* No performance impact
  * Backup on demand or on schedule
  * Automatic Point in time restore like RDS

#### Global tables

* Multi-region, replicated

## ElastiCache

* Managed Redis or Memcached caches

* 'RDS for caches'
* Helps reduce load off databases for read intensive loads
* Scale Write with sharding
* Scale Read with Read Replicas

* Multi-az, with failover capability
* AWS manages OS maintenance, patching, failure recovery, backups
