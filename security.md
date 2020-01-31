# Security

* Encryption (Big)
* KMS (sm)
* CloudHSM (sm)
* STS (sm)
* Identity federation / Cognito (sm)
* CloudTrail (sm)
* VPC Endpoints (sm)
* Policies, IAM (sm)

## Security 101

### Encryption in flight (SSL)

Data is encrypted before sending, and decrypted after recived.
SSL certificates help with encryption (HTTPS).
Encryption in flight helps against man-in-the-middle attack

### Encryption at rest, server side

Data is encrypted after being received by the server.
Data is decrypted before being sent.
Encryption keys must be managed somewhere (like KMS) and the server must have access to the keys.

### Client side encryption

Data is encrypted before being sent.
Server should not be able to decrypted the data.
Date is returned encrypted and only decrypted at the client, after the client has recevied the data.

## S3 Encryption, four methods: SSE-S3, SSE-KMS, SSE-C, and Client Side

* SSE-S3: Encryption keys are managed by S3 (AES-256) - Header: `x-amz-server-side-encryption: AES256`
* SSE-KMS: Encryption keys are managed by KMS - Header: `x-amz-server-side-encryption: aws:kms`
  * KMS gives you more control, user control and an audit trail
* SSE-C: Encryption keys are sent with the PUT/GET requests, must use HTTPS – Header: key is supplied in the header parameters
  * S3 throws away key after it has been used
* Client Side Encryption: Client encrypts data before it is sent to S3. Customer manages keys and encryption

Encryption in transit (SSL) as available with HTTPS endspoints for S3 (SSL/TLS).

## KMS – Amazon Key Management Service

* Encrypts volumes in EBS
* Encrypts objects in S3
* Encrypts data in Redshift
* Encrypts data in RDS database tables
* Integrates with Amazon SSM Parameter Store
* Plus many more

Use KMS you share sensitive information, like database passwords, credentials to external services, or private keys of SSL certificates.

The Customer Master Key (CMK) used to encrypt the data can never be retrieved by the user. KMS handles the creation and management of the Customer Master Key. KMS can rotate the CMK for extra security.

Never store you secrets in plaintext – especially not in code. Encrypted secrets can be stored in code or environment variables. Use KMS to decrypt secrets.

* KMS can only help in encrypting secrets up to 4KB of data per call
  * If data is larger than 4KB, you should use envelope encryption

To give KMS access to someone:

* Make sure the **Key Policy* allows the user
* Make sure the **IAM Policy* allows api calls to KMS

### Manage Keys and Policies

* Manage keys and policies
* Audit key usage with CloudTrails
* Pay for api calls to KMS ($0.03/10,000 calls)

* Three types of Customer Master Keys
  * Aws managed service default CMK: free
  * User Keys created in KMS: $1/month
  * User Keys imported (must be 256-bit *symmetric* key): $1/month

### How KMS works

* Client calls the KMS Encrypt API with the `secret` it wants to encrypt
* KMS checks if the client has the correct **IAM permissions** & **Key Policy**
* KMS then performs the encryption with the KMS managed Customer Master Key (CMK)
* KMS returns the encrypted secret to the client

### Most AWS services require migration (snapshot/backup) to encrypt

* EBS volumes require a migration to encrypt
* RDS databases require a migration to encrypt
* ElastiCache require a migration to encrypt
* EFS network file system require a migration to encrypt

However, S3 allows in-place encryption.

## Cloud HSM

Some companies will require more control over the creation and management of CMKs

* CloudHSM provisions encryption **hardware**
* AWS guarentees tamper resistent
* FIPS 140-2 Level 3 compliance
* Multi AZ
* Supports both symmetric and assymetric encryption (SSL/TLS)
* Must use CLoudHSM client software

* The user manages the keys
* CloudHSM client connects with a SSL connection to AWS CloudHSM dedicated hardware

* CloudHSM is single tenant
* Keys are managed by the user
* Uses cryptographic acceleration (Oracle TDE & SSL/TLS)
* Deployed and managed from a customer VPC.
* Can be shared across VPCs using VPC peering

## Security in Kinesis

### Kinesis Data Streams

* Kinesis Data Strems has SSL endpoints using HTTPS to do encryption in flight
* KMS integrates with Kinesis Data Streams to provide server-side encryption, when data is a rest in the shards
* For client side encryption with Kinesis Data Streams, you must use your own encryption librarys
* Kinesis Data Streams supports interface with VPC Endpoints / Private Link, to avoid sending data over public internet
* KCL must have read/write access to DynamoDB table used for checkpointing

### Kinesis Firehose

* Attach IAM roles to send data to S3 / ElasticSearch / Splunk / Redshift or Kinesis Data Analytics
* Encrypt the Firehose delivery stream with KMS (Server-Side)
* Firehose supports interface VPC Endpoints / Private Link to avoide sending data over public internet

### Kinesis Data Analytics

* Attach IAM roles to read and write Kinesis Streams and reference table in S3
* Data is never at rest in Kinesis Analytics

## Security in SQS

SQS is simple queue service.

* Encryption in flight with SQS HTTPS endpoints
* SQS has a SQS **Queue Access Policy** (similar to an S3 Bucket Policy)
* Service side encryption, for objects in the queue with KMS

* Client Side encryption for SQS must be implemented manually
* SQS integrates with VPC endpoints

## Security for AWS IoT

* IoT polices are X.509 certificates attahed to **Things** or **Groups**
* IAM policies to users, roles, etc.
* Attach roles to the **Rules Engine** so it can interface with other AWS services

## Security in S3

* Access to S3 with IAM policies (for users, groups and roles)
* S3 **Bucket Policies** for individual buckets
* **Access Control Lists** (ACL), Object Access Control List gives fine grain control over access
* Encryption in flight with SSL
* S3 has four different options for encryption at rest SSE-S3, SSE-KMS, SSE-C, CSE
* Versioning and MFA delete options, to avoid things get deleted
* CORS for protecting websites
* VPC Endpoints
* Glacier has **Vault Lock Policies** to prevent deletes

## Dynamo DB

* Data is encrypted using TLS for fligt
* Encrypt at rest is possible with KNS
  * Only for new tables
  * You have to do it when you create the tables
* DynamoDB streams did not always support encryption (it does now)
* VPC Endpoints is provided for DynamoDB through a Gateway

## Security in RDS

* VPC provides network islation to RDS databaes
* For RDS, *Security Groups* control the network access to the database instances
* You can use KMS to encrypt the data at rest in RDS databases
* SSL in flight
* IAM policies are only there to controll access to the RDS apis
* However, PostgreSQL and MySQL in RDS support IAM authentication
* You must manage user permission within the database itself
* MSSQL seriver and Oralce support Transpoarrent Data Encrypiion (TDE)

**Aurora Security** is similar to security in RDS

## Security for Lambda

IAM roles are attached to each Lambda. Integrates with KMS to encrypt secrets. You can also use the SSM parameter store for configuratios. You can deploy Lambda in a VPC to access private ressrouces

## Security in Glue

* You can configure glue to only access JDBC through SSL
* Data Cataloge, can be encrypted by KMS
* Connection passwords can be encrypted by KMS

## Security in EMR

Security in EMR is very important

When creating a cluster, you attach EC2 Key Pairs to SSH credentilas

* Attach IAM roles to EC2 instances to
  * Proper S3 acces
  * For EMRFS requests to S3
  * Dynamo scans through Hive
* EC2 Security Groups
  * One specific security group for the master node
  * Anthe security group for core nodes and task nodes
* Encryption data at rest, you have choices:
  * EBS encryption
  * Open Source HDFS Encryption
  * LUKS + EMRFS for S3
  * LUKS supported with EMR. LUKS works with EMR.
* Nodes communicating with each other, through EMRFS or TLS
* Data is ALWAYS encrypted before uploading to S3
* Kerberos authentication (Privide atuh from Microst Active Directory)
* Apache Ranger support: Centroalized Authorisation (Role Based Access Control - RBAC), set-up on an external EC2 instance to the cluster
  * Apache Ranger is not installed by default within EMR. You must set Ranger up in a seperate EC2 instance external to the cluster

## Security in ElasticSearch

* Deploy in VPC
* ElasticSerch Policy to manage sec further
* Encrypt data with KMS

* Amazon cognito allows end-user to log in to Kibana with SAML / MS Active Directory

* To log into Kibana, you can either use an IAM user or use Cognito
  * Kibana does not support simply using an email/password login

## Security in Redshift

* Vpc
* Cluster Security Groups
* Encryption in flight using JDBC driver with SSL
* Encryptio at rest
  * Using KMS
  * Using a HSM device (Hardware Security Model)
  * Redshift HSM encryption can either be set up with an External HSM or with CloudHSM

* Use IAM
* If using COPY or UNMLOAD must reference keys (alt. paste access key and secret in SQL command)

## Security in Athena

* Inherit Security from S3
* Encrypt data with all S3 options
* Fine grained acces using the AWS Glue Data Cataloge

## Security in Quicksight

* Standard edition is either IAM or email based
* Enterpirese Supports
  * MFA
  * Federated Login
  * Encruption at rest, and in SPICE
  * Active Directory

Quicksight also has Row Level Security to control which users can see which rows

## STS - Security Token Service

Allows granting limited an temporary access to AWS ressources.

An STS token is valid for up to one hour before it must be refreshed.

* STS tokens are used to provide Cross Account Access
  * Allows users from on AWS account to access ressources in anther account
* STS tokens also enable federation access (Active Director)
  * Token can provide a non-AWS user with temporary AWS access by linkning the users Active Directory credentials
  * It uses SAML (Security Assertion Markup Language)
  * STS also allows for Single Sign On (SSO), logging into the AWS console without asssigning IAM credentials
* STS integrates with third party provides / Cognito (login with Google/FB)

### Cross Account Access

To enable cross account access for an applcation with STS, you go through these steps:

* In the target account, you define an IAM for another account to access
* Define which other accounts are able to access this new IAM role
* Using STS, you retrive credentials and impersonate the IAM role you have access to with `AssumeRole API`
* The credentials can be available for 15 min to 1 hour

## Identity Federation

Let users outside AWS temporarily assume roles to access AWS ressources

* A user in a company, or in an app have access to a third party that manages their logins
* If the third party is trusted by AWS, it can communicate with AWS and retrieve temporary credentials for the users
* The user can then use these temporary credentials to access AWS ressources

Supported third party authentications include:

* LDAP
* Microsoft Active Directory (similar to SAML)
* Single Sign On
* Open ID
* Cognito

With Identity Federation, you don't need to create IAM users, as the user management happens outside of AWS

### SAML Federation for Enterprise

* Integrate Active Directory / ADFS with AWS (or any SAML 2.0)
* Avoid creating new users in AWS for all of your employees
* Provicedes access to AWS console or CLI with temporary credentials
* Creates SAML assertions (a kind of token), that talks with STS to create STS token
* Can also send SAML assertion directly through the AWS SSO endpoint for console access

### Custom Identity Broker Application

* Only if your indentity provider is not complient with SAML 2.0
* You have to write your own indentity broker that tolks to STS and indentity provider

### App Access with Cognito, to allow app users to call AWS services

* Credentials created with Cognito to allow temporary access to write to S3
* Alternative to Cognito: Web Idenitiy Federation (but you should use Cognito)

* Your app is connected to Identiy Provider (Goole, Facebook, Cup, SAML)
* The token from the Identity Provider is sent to Federated Identiy
* It talks to STS, and gives back temporary AWS access token
* The app can now call AWS services (like write to S3)

## Advanced Policies

* ${aws:usernamer} : to restrict users to tables or buckets
* ${aws:principaltype} : account, user, federated, assumed role
* ${aws:PrincipalTag/department} : to restrict using tags

You can create specific policies to each user, hardcoding their name.
Or **Better** you use a **policy variable**

* You could allow a user to see all queues, but only to change the queue that has thier name as prefix

IAM policies replace policy variables at runtime

Note for RDS: IAM policies don't help with in-database security!

### Policies for Federated Users

* ${aws:FederatedProvider} : gives which identity provider is used for the user (fx. Cognito, Amazon, SAML, STS)
  * For Amazon, the identity variable looks like this: `${www.amazon.com:user_id}`
  * For Cognito, the identiy variable looks like this: `${cognito-identity.amazonaws.com:sub}`
  * For SAML: `${saml:sub}`
  * For STS: `${sts:ExternalId}`

## CloudTrail

CloudTrail is tracks all API calls, so you can see API calls and audit them

* Who did what and when
* Console
* SDK
* CLI
* AWS services

If a resource is delelted, check CloudTrial

By default, CloudTrail stores the **last 90 days of activity**. After that, you may want to export the logs to somewhere else, for example CloudWatch Logs.

### CloudTrail Trails

* Get a detailed list of event you choose
* Can store to S3
* Can be region specific or globa
* Cloudtrail Automatically uses SSE-S3 encryption when storing in S3 (KMS optional)

## VPC Endpoints (Private Link)

VPC Endpoints (Private Link) allows you to access AWS services without going on the public internet.

They are redundant. AWS manages the VPC Endpoints for you

Two types:

* VPC Endpoint Gateway are for S3 and DynamoDB
* VPC Endpoint Interface are for all other AWS Services
