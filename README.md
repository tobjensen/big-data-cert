# Notes on Big Data Certification

TL;DR: Go through the [Udemy course](https://www.udemy.com/course/aws-big-data/) while writing notes, and take detailed notes on the [Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html) and [EMR](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-what-is-emr.html) AWS documentation. Finally, take the [AWS video course](https://www.aws.training/Details/eLearning?id=35364) and do the AWS provided [sample questions](https://d1.awsstatic.com/training-and-certification/docs-bigdata-spec/BD-S%20Sample%20Questions%20for%20Web.pdf). You'll be all ready to nail the 65-question, 3-hour, scenario-heavy exam.

I've added my notes here for reference, but I do encurage you to take your own notes as you are going through the courses and documentation. Taking notes makes things stick better in your brain (or at least it does for me).

Good luck!

## *Big Data* to be replaced by *Data Analytics* in April 2020

If you want to do the *Big Data* certification, be aware that it will go away in April 2020, and be replaced by a new the new *Data Analytics* certification. The [content outline](https://d1.awsstatic.com/training-and-certification/docs-data-analytics-specialty/AWS%20Certified%20Data%20Analytics%20-%20Specialty%20Exam%20Guide_v1.0_08-23-2019_FINAL.pdf) for the *Data Analytics* exam suggests a larger focus on storage and processing, and less on visualisation, analysis and security.

## Structure of the Big Data exam

The exam is a 65-question, 3-hour, scenario-heavy exam. It is tough – real tough. In order to correctly answer the questions on the exam, you need a lot of depth on a variety of AWS services for data ingestion, transformation, storage and analysis.

The testing center smells like nerd. But the personel are friendly, the A/C works and there are lockers to store your things. You will be given a pen and a single peice of paper, which is all you will be allowed to bring with you to the testing booth.

When you submit the exam, you will immidiatly be notifed if you passed or failed. A couple of hours later you will recive a detailed break down of your percentage of correct answers in total and for each of the 6 domains. Pass rates are not published, but people online have reported to have passed with a score in the low 60s.

## Exam content

Exam questions are most often scenario based. Given a scenario, you will be expected to choose between a lists of soltions that combine multiple AWS products. You will be able to eliminate solutions that are not feasible, don't exist or simple wont work. Often, two or three solutions might be technically feasible, and you will need to select a solution based on the highlighted keywords in the question like *cheapest* or *simplest to implement* or *with least operational overhead*.

In terms of subjects covered, the exam is heavy on management and security of Redshift and EMR. DynamoDB, S3, Kinesis are heavily featured as well. Beyond those core subjects, you can expect to see solutions touch on KMS, CloudHSM, STS, CloudTrail, CloudWatch, Cognito, Identity Federation, SQS, IoT Core, Database Migration Service, Direct Connect, VPN, Snowball, Lambda, Glue, AWS ML (deprecated, but still on the exam), Data Pipeline, ElasticSearch, Athena, RDS, QuickSight, BI tools (Tablau, MicroStrategy) and Visualitions libraries (D3.js, Highcharts).

Security is a big part of the exam. Three common subjects are:

* Encryption of data at-rest and in-flight
* Limiting individual user access to only some part of a dataset (buckets, partitions, tables, columns, rows)
* Cross account and cross region access

Here is the break down from the [exam guide](https://d1.awsstatic.com/training-and-certification/docs-bigdata-spec/AWS%20Certified%20Big%20Data%20-%20Specialty_Exam%20Guide_v1.4_FINAL.pdf). Notice that security is the most highly weighted domain.

| Domain                     | % of Examination |
| -------------------------- | ---------------: |
| Domain 1: Collection       |              17% |
| Domain 2: Storage          |              17% |
| Domain 3: Processing       |              17% |
| Domain 4: Analysis         |              17% |
| Domain 5: Visualization    |              12% |
| Domain 6: Data Security    |              20% |
| TOTAL                      |             100% |

Questions often do not fit neatly into any one of the domains. Rather, the questions will span many domains and ask you to evaluate potential solutions that span multiple AWS services and multiple domains.

## Resources

The best ressource out to help you prepare for the Big Data exam is the [Udemy course](https://www.udemy.com/course/aws-big-data/) by Frank Kane and Stephane Maarek. The course covers nearly everything you need in the exam. The course is to the point and doesn't linger long on AWS services where you don't need a lot of depth to be successful on the exam.

However, the only section covered in too little detail in the course are in the intricacies of managing EMR and Redshift clusters. For Redshift, the course is too light on security and cluster management, including encryption, fine-grained user access and sharing encrypted snapshots across regions. For EMR, the course is too light on security, cluster management, when to use transient vs persitent clusters, Hive (incl. using external storesfor Hive metadata) and transferring data into and out-of the cluster.

Do go through the [Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html) and [EMR](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-what-is-emr.html) AWS documentation and take notes. You don't need to look at any other AWS documentation to pass the exam.

The free [AWS video course](https://www.aws.training/Details/eLearning?id=35364) and [sample questions](https://d1.awsstatic.com/training-and-certification/docs-bigdata-spec/BD-S%20Sample%20Questions%20for%20Web.pdf) are great to finish off your studying. After finishing the Udemy course and going through the Redshift and EMR documention, do the [AWS video course](https://www.aws.training/Details/eLearning?id=35364) and take the AWS provided [sample questions](https://d1.awsstatic.com/training-and-certification/docs-bigdata-spec/BD-S%20Sample%20Questions%20for%20Web.pdf).

Some folks swear to watching re:invent videos to get a better sense of the services in action. The only worthwhile video I found was a [2018 re:invent talk on EMR](https://www.youtube.com/watch?v=ISl9sTzxoSo). Your mileage may vary.
