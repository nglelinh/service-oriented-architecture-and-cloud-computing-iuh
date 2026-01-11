# Lecture_8_Cloud_Providers


## Slide 1

### Cloud Providers


## Slide 3

### Amazon Web Services (AWS)


The largest by far of the public clouds
You use it every day and don’t even know it
Netflix, Reddit, Spotify, and millions others

When it goes down, the half of the internet goes down
Example: The infamous S3 outage in February 2017


## Slide 4

### AWS Offerings


## Slide 5

### Azure Services


## Slide 6

### Google Cloud Platform


## Slide 7

### Feature Parity


All clouds try to compete on features so they all end up having extremely  similar feature sets


## Slide 8

### Top Benefits of Cloud Computing


## Slide 9

### Top Benefits of Cloud Computing


## Slide 10

### Azure Regions


60+ Azure Regions


https://datacenters.microsoft.com/globe/explore/


## Slide 11

### Cost
You will face egress fees if you move data from one region to another. So, it's cheaper for you to have everything nearby.
Some services are cheaper in certain regions over others. Using the Azure calculator can help you estimate the cost differences.


Available Resources
Certain services are only available in certain regions. It might not be possible to have everything in the same place. (See Products by Region.)
Security and Compliance
Depending on the scenario, a particular data center might need to be selected for security and compliance reasons. (Such as government or China)


Speed
Pick a region that is the closest to you will help increase the speed of moving data in and out from your resources.


Reasons to Select a Region


## Slide 12

### Availability Zones


Physically separate locations within each Azure region that are tolerant to local failures (earthquakes, floods, fires, etc.)

Tolerance to failures is achieved because of redundancy and logical isolation of Azure services. 

Azure availability zones are connected by a high-performance network with a round-trip latency of less than 2ms.


Availability Zones


## Slide 13

### Available Service Categories


https://azure.microsoft.com/en-us/services/


Available Service Categories


## Slide 14

### Virtual Machines


## Slide 15

### AWS Elastic Compute Cloud (EC2)


The basic one which all of these clouds provide are Virtual Machines

AWS has everything from the tiny to gigantic
T2.Nano: 1 VCPU 512 MB Ram
X1.32xlarge: 128 VCPU 2000 GB Ram

They have GPUS!
Useful for deep learning

Priced per-second; Options for On-Demand and “Spot Instances”
Spot instance: Auction for unused EC2 capacity; generally much cheaper than On-Demand
Caveat: Your VM may be given a notice to shut down at any point


## Slide 16

### Azure Virtual Machines


Similar to AWS
GPUs
Not as many CPUs (Max is 32 currently)
Not as much ram (Max 800 GB currently)
But you probably will not hit these limits


## Slide 17

### Google Compute Engine


Provides VMs
Largest server is 96 VCPU, 624 GB Ram
Provides custom sized machines
Cost is per second


## Slide 18

### Storage


## Slide 19

### Storage


AWS Simple Storage Service (AWS S3)
Massive storage, a ton of the internet stores all their content here.
For example: Imgur
Google Cloud Storage
Azure Storage


## Slide 20

### Hosted Data Processing


Hosted Hadoop, Spark, HBase, Presto, Hive clusters
Performs all necessary cluster scaling / provisioning automatically


Amazon Elastic Map Reduce
Microsoft HDinsight
Google Dataproc


## Slide 21

### Databases


Let the clouds manage your database hosting
Does create tables and stuff for you, just the stuff below it
AWS
DyanamoDB
Relational Database Server (RDS)
GCP
BigTable
BigQuery
CloudSQL
Spanner
Azure
MSSQL
DocumentDB


## Slide 22

### Unique Features


GCP
CloudSpanner
A planet distributed database
CP System
Tensor Processing Unit
Do deep learning in hardware
AWS
Absurdly large feature set
FPGAs
Azure


## Slide 23

### Cloud Security


## Slide 24

### Cloud Security


Data Storage
Regulatory Standards for confidential data.
Compliance
Data Migration
How to move sensitive data across data centers?
Cloud Permissions
Easier permission setup within organizations
Students don’t get sudo access!
DDoS Mitigation
Fleet of cluster, network security, etc.
High Scalability
Scale with security setting
