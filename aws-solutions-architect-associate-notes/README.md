# **NOTES**

## Exam Logistics
* 130 minutes long
* 60 Questions
* All multiple choice (sometimes choose one answer, sometimes choose multiple)
* Results between 100 - 1000. Passing score is 720+
* Qualificatoin is valid for 2 years
* The questions are scenario based. Not concept or textbook
* $150 registration fee

## Identity and Access Management

IAM allows you to manage users and their level of access to the AWS console and offer features like:
* Centralized control of AWS account
* Shared access to AWS acocunt
* Granular permissions
* Identity Federation
* Multi-Factor Authentication
* Provides temporary access for users/devices and services
* Allows yo set up own password rotation policy
* Supports PCI DSS Compliance (to accept credit card details)

### Key Terminology
* Users
  * End users such as people / employees of an organization
* Groups
  * A collection of users. Each user in a group will inherit the permissions of the group
* Policies
  * Policies made up of documents called policy documents which are formatted in JSON and grant permissions as to what Users/Group/Role is able to do
* Roles
  * Create roles and then assign them to AWS resources

### IAM Lab Notes

Under the IAM service's landing page you have an IAM Users sign-in link like the below:
https://XXXXXXXXXXX.signin.aws.amazon.com/console

This link can be provided to users, and can also be customized. It is a publicly accessible url (inside universal namespace) to sign into that account.

Everything inside of the IAM service is on a global basis. Not locked down to a specific region.

### S3 101

* S3 Storage Classes
  * **S3 Standard**
    99.99% availability. Stored redundantly across multiple devices in multiple facilities. Designed to sustain the loss of 2 facilities concurrently.
  * **S3 IA (Infrequently Accessed)**
    For data that is accessed less frequently but requires rapid access when needed. Lower fee than S3 but charged a retrieval fee
  * **S3 One Zone - IA**
    For when you want a lower-cost optoin for infrequently accessed data, but do not require the multiple Availability Zone data resilience.
  * **S3 Intelligent Tiering**
  Designed to optimize the costs by automatically moving data to the most cost effective access tier without performance impact or operation overhead.
  * **S3 Glacier**
  Secure durable, and low cost storage class for data archiving. Retrieval times configurable from minutes to hours.
  * **S3 Glacier Deep Dive**
  Amazon S3's lowest cost storage class where a retrieval time of 12 hours is acceptable
  
|    | S3 Standard| S3 Intelligent Tiering | S3 Standard IA  | S3 One Zone IA  | S3 Glacier  | S3 Glacier Deep Archive  
|---:|------------- |:-------------:| -----:|------------- |:-------------:| -----:|
|  Durability  | (11 9's) | (11 9's) | (11 9's) | (11 9's) | (11 9's) | (11 9's) |
|  Availability  | 99.99% | 99.99% | 99.99% | 99.5% | N/A | N/A |
|  Availability SLA  | 99.99% | 99% | 99% | 99% | N/A | N/A |
|  Availability Zones   | >=3 | >=3 | >=3 | 1 | >=3 | >=3 |
|  Min Capacity Chrg/obj  | N/A | N/A | 128Kb | 129Kb | 40Kb | 40Kb |
|  Min Strg Duration chrg  | N/A | 30 days | 30 days | 30 days | 90 days | 180 days |
|  Retrieval Fee  | N/A | N/A | per Gb | per Gb | per Gb | per Gb |
|  1st Byte Latency  | ms | ms | ms | ms | mins or hr | hr |

Charged for S3 in following ways:
  * Storage
  * Requests
  * Storage Management Pricing
  * Data Transfer Pricing
  * Transfer Acceleration
  * Cross Region Replication Pricing
  
S3 is object based (allows to upload files)
  * Files can be from 0 to 5 Tb
  * Unlimited storage
  * Files are stored in buckets
  * S3 is a universal namespace (names must be globally unique)
  * Format: https://s3-eu-west-1.amazonaws.com/klaus-testing

### S3 Create Bucket Lab Notes

* There are three layers of access controls. Listed in order of decreasing priority:
  * Account Settings
  * Bucket
  * Objects in the bucket

- When you first upload an object to a bucket it is not publicly accessible.
- To make it public you must enable access at the Account Settings layer, then the Bucket layer, and finally the object layer.
- If you try to enable Public Access on a Bucket before the Account Settings layer, the setting will not stick and simply reset.
- Disabling public access at the Account Settings level will propogate down to Bucket and Object level, disabling them too, regardless of their individual settings.

Scenario:

  Once you enable public access at the Account and Bucket level, you can then make an object in the bucket public using the Actions dropdown menu. Say you make an object publicly available and then want to make it unavailable without disabling the Account and Bucket access settings, you have to:
1. Browse to the object in the S3 bucket
2. Select it 
3. Choose Permissions from the window that appears to the right
4. Then select Everyone under the Public Access section
5. Uncheck Read Object

### S3 Security and Encryption

* Buckets are private by default.
* Set up access control to buckets using:
  * Bucket policies - Work at a bucket level
  * Access Control Lists - Go all the way down to individual objects
* S3 buckets can be configured to create acccess logs which:
  * Log all requests made to the bucket
  * Can be saved to another buckets
  * Can be3 saved to another bucket in another account
* Encryption in transit is achieved by:
  * SSL/TLS
  * HTTP/S
* Encryption at Rest is achieved by:
  * Server side - Amazon encrypts the objects
    * S3 Managed Keys - SSE-S3
    * AWS Key MAnagement Service, Managed Keys - SSE-KMS
    * Server Side Encryption w/ Customer Provided Keys - SSE-C
  * Client side - Client encrypts is and uploads to AWS
    * User encrypts data using own key and uploads to AWS

### S3 Version Control

* Stores all versions of an object (including all writes and even if you delete an object)
* Great backup tool
* Once versioning enabled on bucket, cannot be disabled, only suspended. Have to delete and re-create to completely turn it off
* Integrates with Lifecycle rules
* Has MFA Delete capability which uses multi-factor authenticatoin and is used to provide an addeditional layer of security

* When you enable versioning and re-upload a file, the file goes back to being private. 
  * You must re-enable its public access.
  * The older versions' permissions however remain the same. If you made them public, they will continue being public. Only the latest version reverts back to being private.
* When you enable versioning, the size of the S3 bucket is the **sum** of all the files' sizes. Each version is stored, so from an architectural perspective, keep in mind that if you have versioning enabled and are editing large files, the size of the bucket will increase much much faster.
* If you must have versioning and are editing large files, consider enforcing lifecycle policies.
* If you hide the Versions in a bucket then click delete on an object, the bucket will appear empty (assuming that was the only object in the bucket).
  * But if you enable Versions, you will see all versions of the file still present, except at the top you will now have a new "object" with (Delete Marker) next to it.
  * The actual latest version of the object will be listed immediately below
  * Then if you go in and actually delete the (Delete Marker) "object", the regular, **actual** object will be restored
  * To actually delete that version of the object, you must first show Versions and then delete the object with (Latest Version) next to it.

Essentially, if you delete an object it will place a Delete Marker over that object but the versions that existed before it will still exist.

### S3 Lifecycle Management and Glacier Lab Notes

* To add lifecycle rule go to > Bucket > Management > Lifcycle
* Automate moving objects between the different storage tiers
* Can be used in conjunction with versioning
* Can be applied to current versions and previous versions
* Transitioning small objects to Glacier or Glacier Deep Archive will increase costs
* Must use >=61 days from point of object creation to expire an object
* Must use >=61 days from point of becoming a previous version to permanently delete object

### S3 Cross Region Replication

* Enabling cross region replication requires that versioning is enabled
* Can base replication of objects and buckets off of tags (i.e. only replicate objects with the tag 'Finance')
* The target bucket to which you are replicating cannot exist in the same region as the source bucket 
* Will need to create a new IAM role (if one doesn't already exist) allowing to do replication
* Cross region replication will not replicate objects already in the bucket. Those must be uploaded to the target bucket manually
* If you delete (place a delete marker) an object in the source bucket, the delete marker will **not** replicate over to the target bucket. This is done by Amazon to prevent accidental deletions across buckets.
* If you delete the actual (latest version) of the object in the source bucket, it will **still not** replicate that delete.
* Basically, deletions of any kind are not replicated across buckets.
* When you upload an object to a bucket with replication enabled, it will be private in both the source and target bucket. If you make it public in the source bucket, it will also become public in the target bucket.

EXAM TIPS
  * Versioning must be enabled on both the source and target bucket
  * Regions must be unique
  * FIles in an existing bucket are not replicated automatically
  * All subsequent updated files will be replicated automatically
  * Delete markers are not replicated 
  * Deleting individual versions or delete markers will not be replicated
  
### S3 Transfer Acceleration

*What is:* Utilizes the cloudfront edge network to accelerate uploads to S3. Instead of uploading directly to S3 bucket, 
can use a distinct URL to upload directly to an edge locatoin which will then transfer that file to S3. You will get a distcint URL to upload to: *your-alias*.s3-accelerate.amazonaws.com

* Users upload to edge location first and that will then upload the file across Amazon's backbone network directly to an S3 bucket
* Amazon provides a web tool to compare speeds between direct S3 uploads, and transfer acceleration uploads here:
https://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html

### Cloudfront

*What is:* Is a content delivery network (CDN). A system of distributed servers (a network) that delivers webpages and other web content to a user based on the geographic locations of the user, the origin of the webpgage, and a content delivery server. 

KEY TERMINOLOGY:
  *  **Edge Location:** This is the location where content will be cached. This is separate to an AWS Region/AZ
  *  **Origin:** This is the origin of all the files that the CDN will distribute. This can be an S3 bucket, an EC2 instance, an Elastic Load Balancer, or Route53
  *  **Distribution:** This is the name given to the CDN which consists of a collection of Edge Locations

PROCESS:
  * 1. User queries edge location for content
  * 2. Edge location checks if it has a copy of the content or not (assume not)
  * 3. Edge location will download the content to the edge location and it will be cached there for the Time To Live (TTL) in seconds. (Say 48 hrs)
  * 4. Now if any other users query that same edge location for the same content it will already be cached and quickly accessible.
  
* CloudFront can be used to deliever entire website including dynamic, static, streaming, and interactive content using a global
network of edge locations. Requests for content are automatically reouted to the neartest edge location, so content is delivered with the best possible performance. 
* Two Different Types of Distributions:
  * Web Distributions: Typically used for Websites
  * RTMP - Used for Adobe and Media Streaming

EXAM TIPS
  * **Edge Location**: This is the location where content will be cached. This is separate to an AWS region
  * **Origin**: This is the 
  * 
  * All subsequent updated files will be replicated automatically


### EC2 101

*What is*: Web service that privdes resizable compute capacity in the cloud. Reduces thetime required to obtain and boot new server instances to minutes allowing you to quickly scale capacity both up and down as your computing requirements change.

* Pricing model:
  * **On Demand:** Allows you to pay a fixed rate by the hour (or second) with no commitement
    * Useful For:
      * Users that want low-cost and flexibility without any up front payment or long term commitment
      * Apps with short term, spiky, or unpredictable workloads that cannot be interrupted
      * Apps being developed or tested on EC2 for the first time
      
  * **Reserved:** Provides you with a cpacity reservation and offers a significant discount on the hourly charge for an instance. Contract terms are 1-3 year terms.
    * Useful For:
      * Apps with steady state or predictable usage
      * Apps that require reserved capacity
      * Users who are ablew to make upfront payments to reduce total computing costs even further
    * Three Reserved Pricing Types:
      * Standard Reserves Instances: Offer up to 75% off on-demand instances. The more you pay up front and the longer the contract, the greater the discount.
      * Convertable Reserved Instances: Offer up to 54% off on-demand capability to change the attributes of the RI as long as the exchange results in the creation of Reserved Instances of equal or greater value.
      * Scheduled Reserved Instances: Available to launch within the time windows you reserve. Allows you to match your capacity reservation to a predictable recurring schedule that only requires a fractoin of a day, week, or month.
      
  * **Spot:** Enables you to bid whatever price you want for instance capacity providing even greater savings if your apps have flexbile start/stop times. Price moves around with spot instances. Goes up and down like the stck market. Basically AWS selling off excess capacity at a lower rate.
    * Useful For:
      * Apps that have flexible start/end times
      * Apps that are only feasible at very very low compute prices
      * Users with urgent computing needs for large amounts of additional capacity
    
  * **Dedicated hosts:** Physical EC2 server hosts dedicated for your use. Can help reduce costs by allowing you to use your existing server-bound softwre licenses (i.e. Oracle).
    * Useful For:
      * Regulatory requirements that may not support multi-tenant virtualization
      * Licensing which does not support multi-tenancy or cloud deployments
      * Can be purchased on-demon (by the hour)
      * Can be purchased for reservation for up to 70% off the on-Demand price

| Family | Specialty | Use Case |
|-------:|---------- |:--------:|
|  F1  | Field Programmable Gate Array | Grnomics research, financial analytics, real-time video processing, bid data | 
|   I3 | High Speed Storage | NoSQL DBs, Data Warehousing  | 
|   G3  | Graphics Intensive | Video encoding / 3D Application streaming |
|   H1   | High Disk Throughput | Map-Reduce based workloads, distributed file systems such as HDFS and MapR-FS |
|   T3 | Low Cost, General Purpose | Web servers / small DBs | 
|   D2 | Dense Storage | File Servers / Data warehousing / HADOOP |
|   R5 |  Memory Optimized | Memory intensive Apps/DBs |
|   M5 | General Purpose | Application Servers |
|   C5 | Compute Optimized | CPU Intensive Apps/DBs |
|   P3 | Graphics / General Purpose GPU | Machine learning, Bit Coin mining |
|   X1 | Memory Optimized| SAP HANA / Apache Spark |
|   Z1D | High compute capacity and a high memory footprint | Electronic Design Automation (EDA) and certain RDB workloads w/ high per-core licensing costs  |
|   A1 | ARM based workloads | Scale out workloads such as web servers |
|   U-6tb1 | Bare Metal | Bare metal capabilities that eliminate virtualization overhead |

F - FPGA
I - IOPs
G - Graphics
H - High Disk Throughput
T - Cheap general purpise (think T2 micro)
D - Density
R - RAM
M - Main choice for general purpose apps
C - Compute
P - Graphics
X - Extreme Memory
Z - Extreme Memory AND CPU
A - Arm-based workloads
U - Bare Metal

* Can't convert standard EC2 instances (i.e. change from T2 to T4). Only convertible EC2 instances allow you to change beteen instance types.

EXAM TIPS
  * Know what EC2 is
  * Know the different pricing types
  * If you have a spot instance that's being terminated by EC2 you won't be charged for a partial hour of usage. But if you terminate it yourself you will charged for any hour in which that instance ran.

### EC2 LAB PART I Notes

* Root device volume can only launch on SSD or Magnetic Standard
* Additional volumes include things like cold hard disk drive and optimized throughput hard disk drive (all magnetic)
* Root disk drive is virtual disk in the cloud where our OS will be installed

Connecting:

* To ssh to Linux EC2 from Windows CMD, be sure to specify the user to connect as (ec2-user), otherwise you will get access denied as it will try to connect in as your local workstation user (i.e. boblocal@<ec2-public-dns-name>) 
   * `ssh -i key.pem ec2-user@ec2-XX-XXX-XX-XX.compute-1.amazonaws.com`
* To generate public from pem file:
   * `ssh -keygen -y -f key.pem > key.pub`
* To rename pem file:
   * `ren key.pem key`

Installing and Launching Web Server:

* Update Linux OS
   * `sudo yum update -y`
* Install Apache
   * `sudo yum install httpd -y`
* Navigate to web root
   * `cd /var/www/html`
* Start HTTPD
   * `service httpd start`
* Configure httpd as a service
   * `chkconfig on`

### EC2 LAB PART II Notes

* System Status Check: Checking underlying Hyper-Viser, the actual physical machine
* Instance Status Check: Checking the EC2 instance itself
* Can now encrypt Root device volumes (couldn't before) - **is a popular exam topic now**
* Additionally attached EBS volumes do not have "Delete on Termination" checked by default so if you delete an EC2 instance and didn't check that setting during the initial EC2 config/deployment, only the Root device will get deleted upon the termination. All other volumes will remain. 

EXAM TIPS:

* Termination protection is turned off by default. You must turn it on.
* On an EBS-backed instance the default action is for the root EBS volume to be deleted when the instance is terminated. Additional ones won't.
* EBS Root volumes of your DEFAULT AMIs **CAN** be encrypted. You can also use a third party tool such as BitLocker to encrypt the root volume, or this can be done when creating the AMI's in the AWS console  or using the API. - **is a popular exam topic**
* Additional volumes can be encrypted as well

### EC2 Security Groups

* Rule changes on security groups is immediate - **is a popular exam topic**
* Security groups are stateful. Whenever you create an inbound rule, a corresponding outbound rule is also created. 
  * So for example, if you have an inbound rule allowing HTTP traffic, deleting the default outbound rule (for all traffic) will not affect the web page. You will still be able to load the web page because HTTP will automatically be allowed back out. Same for RDP, SSH, FTP, etc... If it's allowed in, then it's also allowed out.
* NACLs however are stateless
* Can't block or blacklist a particular port or IP in security groups. Only in NACLs.
  * But everything is blocked by default anyway when it comes to security groups
* Can attach more than one security group to a given EC2 instance:
  * From the EC2 instances page, go to Actions > Networking > Change Security Groups
  * Adding a security group to an instance takes affect immediately
  
EXAM TIPS:

* All Inbound traffic is blocked by default 
* All Outbound traffic allowed
* Security groups are stateful
  * If you enable something on the Inbound, Outbound for that same type of traffic is automatically allowed
* Changes to security groups are immediate
* Can have any number of EC2 instances withing a security group
* Can have multiple security groups attached to an EC2 instance
* Cannot block specific ports or ips using security groups. That must be done using NACLs
* Can specifiy allow rules, but not deny rules

### EBS 101

*What is:* Elastic Block Store that provides persistent block storage volunmes for use with Amazon EC2 instances in the cloud. Each EBS volume is automatical replicated within its Availability Zone to protect you from component failure offering high availability and durability.

* Just a virtual hard disk in the cloud
* 5 Different types of storage:
  * General Purpose SSD
  * Provisioned IOPS
  * Throughout optimized hard disk drive
  * Cold hard disk drive
  * Magnetic

| Family | Specialty | Use Case | Family | Specialty | Use Case |
|-------:|---------- |:--------:|-------:|---------- |:--------:|
|Volume Type|Gen Prurpose SSD|Provisioned IOPS SSD|Throughput Optimized HDD|Cold HDD|EBS Magnetic|
|Description|Balances price and perf for wide variety of transactional workloads|Highest performance SSD volume designed for mission critical apps|Low cost HDD volume designed for frequently accessed, throughput-intensive workloads|Low cost HDD volume designed for less frequently accessed workloads|Previous generation HDD|
|Use Cases|Most Work Loads|Databases|Big Data and Data Warehouses|File Servers|Workloads where data is infrequently accessed|
|API Name|gp2|io1|st1|sc1|standard|
|Volume Size|1Gb - 16Tb|4Gb - 16Tb|500Gb - 16Tb|500Gb - 16Tb|1Gb - 1Tb|

### EBS Volumes and Snapshots

* Wherever yous EC2 instance is, the volume is going to be in the same AZ
* Always want the volume and instance to be in the same AZ to minimize latency and lag
* Terminating the EC2 instance will also termiante the EBS volume automatically
* Delete on termination is NOT checked by default **unless** it's the root volume
* Can extend a volume's size even after it's been deployed with the EC2 but will need to repartition the drive in the OS as it will only see the original volume size until then.
* Maximum ratio of 50:1 is permitted between IOPS and volume size.
* Don't need to pause/stop/restart EC2 instance to make changes to volumes
* So we are changing the storage medium as well as the amount of storage available
* In order to move an EBS volume to another AZ, must first create a snapshot of it, and then create an image of that snapshot. Once that image is created you will then be able to use it to launch new EC2 instances. When you launch an EC2 of that image you will have the optin to deploy it to another (different) AZ. (Essentially migrating data from one EC2/EBS from one AZ to another). Snapshot --> AMI --> New EC2 - **is a popular exam topic**
* PV vs HVM. When you create an AMI from a EBS snapshot and want as many EC2 instance types as possible, then use HVM. That provides more EC2 instance types to launch that image from late from.
* Can copy AMIs to different regions and then launch instances of that AMI in the new region. So this offers a way of not only moving across AZs but also regions.
* When you delete an EC2 instance the root device volume will be deleted as well. But additionally added volumes will remain (Delete on termination was not checked by default)
* May have to delete the AMI before the snapshot if you based the AMI off that snapshot

EXAM TIPS:

* Volumes exist on EBS. Think of EBS as a virtual hard disk
* Snapshots exist on S3. Think of snapshots as a photograph of the disk
* Snapshots are point-in-time copies of volumes
* Snaphots are incremental. So only the blocks that have changed (the delta) will be replicated to S3
* First snapshot may take some time to create
* To createa snapshot for Amazon EBS volumes that serve as root devices, you should stop the instance before taking the snapshot
* CAN take a snappsht while an instance is still running
* Can create AMIs from both Volumes and Snapshots
* You can chang EBS volume sizes on the fly. Including size and storage type
* Volumes will ALWAYS be in the same AZ as the EC2 instance 
* To move an EC2 volume from one AZ to another, take a snapshot, create an AMI off the snapshot, then use the AMi to launch an EC2 in the new AZ
* To move an EC2 Volume from one region to another, take a snapshot of it, create an AMI off the snapshot, and then copy the AMI from one region to the other. Then use the copied AMI to launch the new EC2 instance in the new region.

### AMI Type EBS vs Instance Store

* Can select AMI based on following factors:
  * Regions
  * Operating Systems
  * Architecture (32-bit or 64-bit)
  * Launch permissions
  * Storage for the root device (Root Device Volume)
    * Instance Store (EPHEMERAL STORAGE)
    * EBS Backed Volume
* All AMIs are backed by *either* EBS Volumes or Instance Store, but not both
 
 Instance Store Volumes: *The root device (where the OS is installed) for an instance launched from the AMI is an instance store volume created from a template stored in AWS S3*
 
 EBS Volume: *The root device (where the OS is installed) for an instance launched from the AMI is an Amazon EBS Volume created from an Amazon EBS Snapshot*
 
* You can attach additional EBS volumes after launching an instance, but not instance store volumes.
* Can only see EBS volumes under the volumes service. Instance Stores won't appear.
* When you stop an EBS backed EC2 instance all the data persists
* Can only Rebot or Terminate Instance Stored backed EC2 instances
* Can resolve issues with the EC2 Hyper-Visor simply by stopping starting them. That will launch the instance on a new HV is say a System Status Check threw an error.
* If underlying HV has an issues for an Instance Store backed EC2 then you will lose all your data (i.e. Ephemeral)

EXAM TIPS:

* Instance Store volumes are sometimes called Ephemeal Storage
* Instance Store volumes cannot be stopped. If theunderlying host fails you will lose your data
* EBS backed instances can be stopped and won't lose data is the EC2 is stopped 
* Can reboot both. Will not lose data
* By default both root volumes will be deleted on termination. However with EBS volumes you can tell AWS to keep the root device volume 

### Encrypted Root Device Volumes and Snapshots

* Root device volume is hard disk with Operating System
* **Can** provision encrypted root device volumes immediately when creating an EC2 instance 
* To create an encrypted volume the *old* way (i.e. have already provisoined an unencrypted volume and want to encrypt)
  * 1. Create a Snapshot of Volume
  * 2. Create a Copy of the Snapshot 
  * 3. When creating a copy, check the option to encrypt
  * 4. Create an Image of the encrypted Snapshot (the copy)
  
EXAM TIPS:

* Snapshots of encrypted volumes are encrypted automatically
* Volumes restored from encrypted snapshots are encrypted automatically
* You can share snaphots but only if they are unencrypted
* These snapshots can be shared with other AWS accounts or made public if they are unencrypted
* Can now encrypt root device volumes upon the creation of an EC2 instance. Otherwise the steps are:
  * Create a Snap of the enencrypted root device volume 
  * Create a Copy (also a snapshot) of the Snapshot and choose the encryption option
  * Create an AMI from the encrpyted Snap
  * Use that AMI to launch new encrypted instances

### CloudWatch 101

* *What is CloudWatch:* A monitoring service to monitor AWS resources as well as the applications you run in AWS. It monitors performance. Can monitor things like:
  * Compute: 
    * EC2 Instances
    * Autscaling groups
    * Elastic Load Balancers
    * Route53 Health Checks
  * Storage and Content Delivery
    * EBS volumes
    * Storage Gateways
    * CloudFront
* CloudWatch can also monitor Host Level Metrics of an EC2 such as: (**is a popular exam topic**)
  * CPU
  * Network
  * Disk
  * Status Check
    * Underlying Hyper Visor (i.e. if it is running)
    * Underlying EC2 instance

* In the exam will likely be given a scenario and asked if you should be using CloudWatch or CloudTrail

* *What is CloudTrail:* Increases visibility into your user and resource activity by recording AWS Management Console actions and API calls. Can identify which users and accounts called AWS, the source IP from which the calls were made, and when the calls occurred. Basicaly a CCTV camera for your AWS environment. (i.e. records if/when/who created an EC2 instance or S3 bucket)

* **CloudWatch monitors performance**
* **CloudTrail monitors API calls in the AWS platform**

EXAM TIPS:

* CloudWatch can monitor most of AWS as well as your applications that run on AWS
* CloudWatch with EC2 will monitor events every 5 minutes by default
* Can have 1 minite intervals by enabling detailed monitoring
* Can create CloudWatch alarms which trigger notifications
* CloudWatch is all about **performance** whereas CloudTrail is all about **auditing**
example: Network throughput or disk io...CloudWatch. Who's provisioning EC2 and S3s...CloudTrail

### CloudWatch LAB Notes

* Enable CloudWatch Detailed Monitoring in the "Configure Instance Details" page of the wizard to deploy new EC2
  * One minute intervals
  * Not free tier. But Standard Monitoring for free
* Once deployed, the Monitoringtab of the EC2 will have a plethora more of monitoring details like:
  * CPU, Disk Read/Write, Network In/Out, etc...
  
  
* Setting up a CloudWatch Alert:
 1. Services > CloudWatch
 2. Alarms > Create Alarm
 3. Select Metric > EC2
 4. Per-Instance Metrics
 5. Check the CPUUtilization metric corresponding to the EC2 Instance's ID you want to watch
 6. Click Select Metric
 7. Provide a Metric Name 
 8. Select Greater/Equal
 9. Enter 90 for the threshold value
 10. Click Next
 11. Select Create New Topic (unless you already have one created, in which case choose that)
 12. Provide Topic details and create
 13. Enter email
 14. Click Next and provide an Alarm Name
 15. Click Next, Create Alarm, and then confirm the email the SNS topic sends (otherwise will not receive alerts until you do)

Says if CPU is greater than 90% for 1 minute, out of the minute, then send alarm. Can also edit it to say "2 minutes out of 5" or "3 minutes out of 10".

Note: Took 15 minutes for me to get my email notificatoin. Halfway through I switched my intervals from 5 minutes to 1 minute. Not sure if this helped expedite, or if it was my email provider.

* Trigger Alarm:
 1. SSH into EC2 (ssh -i "key-file.pem" ec2-user@public ip
 2. `sudo su`
 3. `while true; do echo; done`

EXAM TIPS:

* Standard Monitoring = 5 minutes
* Detailed Monitoring = 1 minute
* Dashboards - Can create dashboards for views of AWS activity
* Alarms - Allows you to receive an alarm when certai nthresholds are hit
* Events - Helps respond to state changes i naws resources 
* Logs - CloufWatch logs allow you to aggregate, store, and monitor logs
* CloudWatch - Performance
* CoudTrail - Monitors API calls in the AWS platform

### AWS Command Line (CLI) Lab

* *What is awcli:* Allows you to interact with the platform via the command line, without the console
* Need to grant the user(s) "Programmatic Access"
* Retrieve "Access Key ID" and "Secret Access Key" for user. These are the creds used to bind programmatically
* Can make the creds inactive and/or delete them from the IAM console for users (i.e. if you lose the key)
* Use the cli to provision/delete resources, manage them, configure them, etc...
* Need to run `aws configure` first to supply access key id and the key, region, and format output
* `cd .aws` takes you to hidden directory where you can view and edit **config** and **credentials**
* This can be a security risk because if that one EC2 is compromised, then the credentials can be viewed and used to compromise any and all other resources in that region. Safer to use "roles"

EXAM TIPS:

* Can interact with AWS from anywhere in the world just by using the CLI
* Will need to set up access in IAM - Create user, add to group, assign and attach policy to group
* Commands will not be in the exam but some basic ones will be useful to know in real life

### IAM - Roles LAB

* *What is Roles:* Roles allow us to interact with the AWS platform without having to pass our AWS Access Key Id and Access Key to the EC2 instance
* Roles are **NOT** just for EC2s. Can be used for other services as well.

* Create a Role:
  Note that we already have an existing Role from a previous lab (cross region replication) which was created automatically when we enabled cross-region replication: `s3crr_for_klaus-testing-versioning_to_klaus-cross-region-replication-target-bucket`
  
1. IAM Service > Roles
2. Create a new Role
3. Select EC2 as the service which will use this role
4. Attach the AdministratorAccess policy to the role
5. Supply name for the Role: AdminAccess
6. Browse to the EC2 instance
7. SSH into it
8. Delete the hidden .aws directory using `rm -rf .aws`
9. Now aws cli commands like `aws s3 ls` won't work because awscli is no longer configured as there is no longer credentials and config files to use
10. Now need to attach role to the EC2 instance
11. Go to EC2 instance > Actions > Instance Settings > Attach/Replace IAM Role
12. Select the AdminAccess Role created in steps 2-3 and click Apply
13. Click Close
14. Should now see IAM Role field populated in the EC2 Instance's Details
15. Go back to SSH session to EC2
16. Now `aws s3 ls` will work (yet no hidden .aws folder)
  
Note: Hackers would still be able to run the awscli but this is still more secure as hackers would not be able to use the keys for lateral movement or escalation. This EC2 would be the only port of entry. 

EXAM TIPS:

* Roles are more secure than storing your access key and secret access key on individual EC2 instances
* Roles are easier to manage. For instance, if you had 100 EC2 with the awscli confgured using keys, and you lost the keys, you'd have to regenerate a new key and manually update the awscli config on all those EC2s. Whereas with Roles, you only ever need to update the Role which can be done in one place, just once.
* Roles can be assigned to EC2 after they're created using both console and cli
* Roles are universal - can use them in any region

 ### Boot Strap Scripts LAB
 
* *What is Boot Strap Scripts:* Grants us a way to automate EC2 deployments. Way of running things at commandline when EC2 first boots. Can run individual commands, run updates, install software, and run it all as a script when the EC2 first boots up.

* When configuring an EC2 instance, in the "Configure Instance Details" page of the wizard, expand the "Advanced" banner
* In the text box for User Data, you can supply a boot strap script (i.e. a bash script)
*  Script runs as root when EC2 first boots up
* Example Boot Script:

```#!/bin/bash
yum update -y
yum install httpd -y
service httpd start
chkconfig httpd on
cd /var/www/html
echo "<html><h1>Header</h1></html>" > index.html
aws s3 mb s3://sdfgfdhfdjytjgfn
aws s3 cp index.html s3://sdfgfdhfdjytjgfn
```
* Be sure to attach our previously created role: *AdminAccess* so the aws cli commands can run. Otherwise the aws cli will have to be configured manually.


 ### Instance Metadata LAB
 
 * ssh into EC2 and run: `curl http://169.254.169.254/latest/user-data`
 * This will output the user-data value (*from the EC2 Instance Configuration Details Page*) which is our boot strap script:
 OUTPUT
 ```
[root@ip-10-0-0-161 ec2-user]# curl http://169.254.169.254/latest/user-data
#!/bin/bash
yum update -y
yum install httpd -y
service httpd start
chkconfig httpd on
cd /var/www/html
echo "<html><h1>Header</h1></html>" > index.html
aws s3 mb s3://sdfgfdhfdjytjgfn
aws s3 cp index.html s3://sdfgfdhfdjytjgfn[root@ip-10-0-0-161 ec2-user]# 
 ```
* Note: Substituting in localhost or 127.0.0.1 doesn't work. Only the 169 ip
* To get metadata use: `curl http://169.254.169.254/latest/meta-data/`
OUTPUT:
```
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
events/
hostname
iam/
identity-credentials/
instance-action
instance-id
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
```
* To get local ipv4 use: `curl http://169.254.169.254/latest/meta-data/local-ipv4/`
OUPUT:
```
10.0.0.161
```
* To get public ipv4 use: `curl http://169.254.169.254/latest/meta-data/public-ipv4/`
* Potentially use these API endpoints in a boot strap script to collect ip data and push them to an S3 bucket which triggers a lambda function to store the ip in a DB. Basically, write ip address to a DB directly from EC2 instance.  

EXAM TIPS:

* Meta data is used to get information about an instance such as private and public ip addresses
* Can get user data which will contain the boostrap script you provided during EC2 configuration


 ### Elastic File System
 
 *What is:* Is a file storage service used for EC2 instances. EFS provides an interface to create and configure file systems quickly and easily. Storage cpacity is elastic, growing and shrinking automatically as you add and remove files so your apps have the storage they need when they need it. 
 
 * Similar to EBS but with EBS you can only mount your virtual disk to one EC2 instance
 * Can't have 2 EC2 instances sharing an EBS volume. But can have them sharing an EFS
 * If you provision an EFS instance, it will grow automatically. Don't need to pre-provision storage like you do with EBS
 * Great for file servers and a great way to share files between two EC2 isntances
 * Can mount /var/www to an EFS mounts point which is used by N number of EC2s. That way when you update files in your website for instance you only need to update them in 1 place and all your EC2s serving that website will be all be updated at once.
 
  ### Elastic File System LAB
  
  1. Navigate to EFS service in console > Create File System
  2. Note the VPC and AZ
  3. Enable "Encryption of Data at Rest" and use the KMS is creates
  4. Click Next
  5. Leave default policy settings
  6. Create File System (~4 - 5 minutes)
  7. Update the Security Group you intend to attach to the EC2 and EFS service to allow NFS (2049) for the same Security Group. i.e. Select Custom from the drop down menu and stype "sg-" to let it autopopulate a dropdown list and choose the appropriate SG from the list. i.e. sg-06274eeebe1c946ea 
   * This allows the NFS protocol into out security group from our web servers
  8. Provision 2 t2 micro EC2
   * Attach the *AdminAccess* role
   * Enable auto-assign ip (make sure they get public ips assigned)
   * Supply the following boot strap script:
  ```
#!/bin/bash
yum update -y
yum install httpd -y
service httpd start
chkconfig httpd on
yum install amazon-efs-utils -y
```
9. Grab the 2 public ips of the running instances
10. SSH into both instances and ensure the /var/www/html dir exists (if so our boostrap script worked)
11. Navigate back to EFS service in AWS console
12. click Amazon EC2 mount instructions (from local VPC)
13. Note that we already ran `sudo yum install -y amazon-efs-utils` as part of our boot strap script
14. Run the command `sudo mount -t efs -o tls fs-2865e882:/ /var/www/html` on both EC2 instances
  * Note that we need the -o tls flag for encryption of data in transit because we selected Encryption of Data at Rest when creating the EFS
  * If the cmd hangs and/or throws the error *mount.nfs4: Connection reset by peer* it typically means the security group isn't correct. Go back to the EFS, Edit it, and ensure the correct SG is attached to it under Actions > Manage Network Access/
15. Note the contents of both EC2s html directories and then on one instance run: `echo "<html><body><h1>This is a HEADER From SERVER 1</h1></body></html>" > index.html` from inside the html folder
16. Note that the same file will now exist in the other EC2's /var/www/html folder
17. Similarly, making a change on the file from the other instance will be reflected in the opposite EC2

* Both EC2s are reading from the same file system (for the /var/www/html directory) 
* Can take a while to delete EFSs
* EFS is a way having commonn files or storage between various EC2 instances
* Don't need to worry about syncing files between EC2 instances

EXAM TIPS:

* EFS supports a network file system version 4 (NFSv4)
* Only pay for the storage that you use. No pre-provisoining
* Can scale up to petabytes
* Can support thousands of concurrent NFS connections
* Data is stored across multiple AZs within a region
* Read-after-write consistency

 ### EC2 Placement Groups
 
 *What is:* A way ofd placing Ec2 instances. 3 Types:
  * **Clustered**
    Grouping of instances withing a single AZ. Recommended for apps that require low network latency, high network throughout or both. Only certain instances can be launched into a clustered placement group. A way of putting EC2s very very close to each other. A way of grouping EC2s closely within a single AZ
  * **Spread**
    Essentially the opposite. A group of EC2s that placed on distinct underlying hardware. Recommended to applications that have a small number of critical instances that should be kept seprate from one another. Separate racks, with separate network inputs, and power requirements. So if one rack fails, it will only affect that one EC2 instance. Can have spread placement groups within differnt AZs in one region. Completely isolated.
  * **Partition**
    Similar to spread placement groups except you can multiple EC2 instances within a partition. EC2 divides each group into logical partitions. Ensures that each partition within a placement group has its own set of racks. Each rack has its own network and power source. No two partitions within a placement group share the same racks, allowing you to isolate the impact of harware failure within your application.

* Difference between Spread and Partition groups is that Spread is used for single instances, and partition is meant for multiple instances
* And cluster is just a way of putting everything as close together as possible.
* Used to only be 1 placement group. Now there are 3

EXAM TIPS:

* Clustered good for low latency / high network throughput
* Spread is good for individual critical EC2s
* Partition is good for multiple instances of HDFS, HBase, and Cassandra
* Clustered group can't span multiple AZs
* Spread and partition groups *can* span across multiple AZs
* The name you specify for a group must be unique within your AWS account
* Only certain types of instances can be launched within groups (i.e. Compute Optimized, GPU, Memory Optimized, Storage Optimized)
* AWS recommends homogenous instances withing clustered groups (i.e. same EC2 instance type)
* Can't merge groups
* Can move an existing instance into a group however it must be in the stopped state first
* Can move or remove an instance using AWS CLI or AWS SDK. Can't do it via console yet (as of 02/18/2020)

 ### Databases 101
 
 * If you have an excel file, it is saved as a .xls. You can think of that as the database itself
   * And then inside that file you have different worksheets
     * Those worksheets have different tables
       * And isnde those tables you have rows and columns (collectively referred to as fields)
* There are 6 different relational DBs (RDS) on AWS:
  * MSSQL
  * Postgresql
  * MariaDB
  * Oracle
  * MySQL
  * Aurora
* Two key features for RDS:
  * Multi-AZ (For disaster recovery)
  * Read-Replicas (For performance)

* Multi-AZ:
  * EC2s connect into DBs using a connection string. It is essentially a DNS address.
  * AWS hosts the DNS record and points it at a DB; the primary DB, which resolves to its internal ip address.
  * If we lose our primary DB, AWS would detect that and automatically update it to the secondary wherein it points the DNS to the internal ip of the secondary DB server
  * So failover is automatic with Multi-AZ. Basically it is used for Disaster Recovery

* Read-Replicas:
  * EC2s point to primary instance using conn string.
  * Every time a write is made on the primary, it is replicated to another DB
  * If you lose your primary DB, there is **NO** automatic failover
  * Would have to essentially create a new URL and update the EC2 instances to point to the new read replica
  * If you have EC2 instances running a Wordpress blog, and primary DB getting too many reads, you scale it using a read-replica and point half the read requests to the primary, and half to the replica. Basically it is used for performance
  * Can have 5 copies/replicas
  
* Non-Relational DBs:
  * Consists of a collection which is just a table
    * Inside the collectoin you have Documents
      * Documents are simply a row
        * Key/value pairs which are (fields or columns) which can be nested
        
* Difference between RDS and NRDS:
  * Can have as many different columns as you want for any given row within a non-relational database. i.e. in a NRDS you can add a new column/field for a given record without impacting all other records. Whereas with an RDS, you could feasibly end up with missing data for N number of rows when a new column/field is added because not all rows might have data for it.
  * For RDS must keep some kind of consistency across all the records
  
* Data Warehousing:
* *What is:* Used ofr business intelligence. Tools like Cognos, Jaspersoft, SQL Server Reporting Services, Oracle Hyperion, SAP NetWeaver. Used to pull in very large and complex data sets and is used by management to run queries on data such as current performance vs target etc...


* Online Transaction Process (OLTP) differs from Online Analytics Processing (OLP) in terms of the types of queries you will run.

EXAMPLE:

* OLTP:
  * Say we want to find the Order: Order Number 2120121. So we run a query to fetch the order and it will pull up a row of data such as Name, Date, Address to Deliver to, Delivery Status etc... 

* OLP
  * As a manager you want to find the net profit for Europe Middle East and Africa (EMEA) and Pacific for the Digital Radio product. This will pull in a large number of records, sum up all the radio sold in EMEA, sum up all the radios sold in the Pacific, compute the unit cost of the radio in each region, take the sales price of each radio, and calculate Sales Price - Unit Cost.

* So you will want to use different DBs for each scenario. One DB for simple queries coming in from your website, and a different one for your Data Analytics.
* Data Warehousing databases use different types of architecture both from a DB perspective and an infrstructure layer.
* Redshift is Amazon's data Warehousing solution for online analytics processing

* ElastiCache:
  * ElastiCache is a web service that make sit easy to deploy, operate, and scale an in-memory cache in the cloud. The service improves the performance of web applications by allowing you to retrieve information from fast, managed, in-memory caches instead of relying entirely on slower disk-based databases. Essentially a way of caching your most common web queries. So web servers will go to cache first instead of DB.
  * Supports 2 open-source in-memory caching engines:
    * Memcached
    * Redis
    
* Will get a lot of scenario-based questions around DBs being overloaded and asked for a solution. ElasticCache is one option. Can also use read-replicas.
  
EXAM TIPS:

* RDS consists of OLTP. Comes in 6 types:
  * MSSQL
  * Postgresql
  * MariaDB
  * Oracle
  * MySQL
  * Aurora
* DynamoDB is Amazon's NoSQL solution
* Redshift is Amazon's Data Warehousing solution which is used for OLAP 
* ElastiCache is Amazon's caching solution and has 2 engines:
   * Memcached
   * Redis
* Redshift is used for business intelligence or for data Warehousing 
* ElastiCache is used to speed up performance of existing databases (frequent identical queries)

### Creating and RDS Instance - LAB

* Exam will feature mainly RDS, Redshift, ElastiCache, and DynamoDB
1. Browse to RDS service
2. Click Create Database
3. Choose MySQL (Keep version the same)
4. Select Free Tier 
5. Provide a DB Instance Identifier (it is the name of the RDS instance, not the database.)
6. Supply a password
7. If you have more than one VPC be sure to select your desired VPC under the Connectivity section. Note that under "Additional Connectivity Configuration" you can make it publicly accessible
8. Attach desired security group. Will need to update it later with proper ports
9. Under Additional Configuration be sure to an "Initial database name" otherwise need to manually create one via SQL query
10. Change backup retention to 0 days. If you leave it at 7, it will autoamtically enable "Automatic Backups" **even if you uncheck it**. 
11. Click Create Database

I got the error: 

```"VPC must have a minimum of 2 subnets in order to create a DB Subnet Group. Go to the VPC Management Console to add subnets."```

So I had to go and create a second subnet and attach it to the same VPC. Then I got this error:

```"DB Subnet Group doesn't meet availability zone coverage requirement. Please add subnets to cover at least 2 availability zones. Current coverage: 1 (Service: AmazonRDS; Status Code: 400; Error Code: DBSubnetGroupDoesNotCoverEnoughAZs; Request ID: 24203498-350b-4d71-8749-2838dbba0188)"```

That was bcause they were both in us-west-2a. So I had to delete and recreate it, but this time select an explicit zone rather than saying "no preference.". After that my RDS deployed successfully.

12. While waiting for the RDS to deploy, provision a new t2 micro EC2 instance passing in the following bootstrap script:

```
#!/bin/bash
yum install httpd php php-mysql -y
cd /var/www/html
wget https://wordpress.org/wordpress-5.1.1.tar.gz
tar -xzf wordpress-5.1.1.tar.gz
cp -r wordpress/* /var/www/html/
rm -rf wordpress
rm -rf wordpress-5.1.1.tar.gz
chmod -R 755 wp-content
chown -R apache:apache wp-content
service httpd start
chkconfig httpd on
```

Note that our EC2 instances wil need to talk to our RDS instance using the MySQL port so we need to allow port 3306 inbound to the RDS SG from the EC2 SG. 

13. Edit the SG associated with the RDS intsance add a new rule to allow MySQL in. And for the for source simply start typing "sg-" and it will present a dropdown list of the other Security Groups available. In this, select the SG associated with the EC2 instance.

Note: If you need to modify the RDS instance (i.e. change the SG, it will apply only during the next maintenance period. If you want the changes to apply immediately, then you need to choose "Apply Immediately".

14. Back under RDS Service, if you click the name of your RDS isntance, you can find the RDS DNS connection endpoint under the Connectivity sub-tab. Looks like *rdslab.convco2ci5xi.us-west-2.rds.amazonaws.com*. Will be needed to install Wordpress

15. Copy the public URI of your Wordpress EC2 and browse to it, taking you to the Wordpress setup page. (remember that wordpress was installed per our bootstrap script.)

15. Supply the DB name (can be found under the Configuration sub-tab of the RDS instance page), user/pass, and the RDS DNS endpoint URL.
16. Click Submit

When you get this error the Wordpress couldn't write the wp-config.php file to the DB, copy the contents of the wp-config.php file, and ssh into the EC2 instance. Create the file manually in the html directory.
`cd /var/www/html`
`vi wp-config.php`
Paste the contents that are copied to your clipboard, save and exit, and then go back to the Wordpress setup page and click "Run the Installation".

17. Provide a name for the site, a username/password, and a dummy email (optional).
18. Click "Install Wordpress"
19. Once it's installed, it means the EC2 has successfully written data to the RDS instance

EXAM TIPS:

* RDS runs on Virtual Machines but you don't have access to them. So can't ssh to RDS instance (i.e. patch the OS. That's Amazon's responsibility. To patch the Database and OS)
* RDS is not serverless **(will be tested on this)** Aurora is the exception. 
* Aurora serverless is serverless.

### RDS Backups, Multi-AZ, and Read Replicas

 * 2 Different types of backups for RDS:
   * Automated backups
   * Database snapshots
   
* **Automated:** Allow you to recover your database to any point in timewithin a retention period. The retention period can be between 1 - 35 days. Automated backups will take a full daily snapshot and will also store transaction logs throughout the day. When you do a recovery, AWS will first choose the most recent daily backup and then apply transaction logs relevant to that day. This allows you to doa point in time recovery down to the second, within the retention period.
  * Auotmated backups are enabled by default
  * The backup data is stored in S3
  * Get free storage space equal to the size of your DB
    * So if you have a 10 Gb databasem you will gte 10Gb of free storage space
  * Backups are taken withing a defined window
  * During the backup window, storage I/O may be suspended while data is being backed up
  * May experience elevated latency

 
* **Snapshots:** Are performed mnually (i.e. user-initiated) and are stored even after you delete the RDS instance, unlike automated backups.
 
* Whenever you restore a snapshot or automated backup, th restored version of the DB will be a new RDS instance with a new DNS endpoint
  * *original.eu-west-q.rds.amazonaws.com* --> *restored.eu-west-q.rds.amazonaws.com*
* Encryption at rest is supported for:
  * MySQL
  * Oracle
  * SQL Server
  * Postgresql
  * MariaDB
  * Aurora
  
* Encryption is done through the AWS Key Management Service (KMS)
* Once your RDS is encrypted, the stored at rest in the underlying storage is encrypted as is automated backups, read replicas, and snapshots

* **Multi-AZ:** Allows you to have an exact copy of your productoin DB in another AZ. 
  * AWS handles the replicatoin for you so when your production DB is written to, it will automatically be synchronized to the stand-by DB
  * In the event of planned DB maintenance, DB instance failure, or an AZ failure, Amazon RDS will automatically fail over to the stand-by so that ops can resume without administrative intervention.
  * Don't need to change connectoin strings. Same endpoint. Just update the ip 
  * Multi AZ is for disater recovery only. Not for performance (Read Replicas)
  * Multi AZ is available for:
    * MySQL
    * Oracle
    * SQL Server
    * Postgresql
    * MariaDB
  * Aurora already faul tolerant

* **Read Replicas:** Improve performance through lateral scalability by creating read-only replicas of your DB.
  * Can configure EC2s to each read from their own respective DB instance (i.e. a replica of the master)
  * However all EC2s still need to write the same singular master RDS instance, and those changes are replicated
  * Can have read replicas of read replicas
  * Replication achieved using asynchronous replicatoin from primary to read replica
  * Can use read replica for very read-heavy workloads
  * Will come up in exam when asking how to improve performance for a DB. Really 2 ways:
    * Read Replicas
    * ElastiCache
  * Read Replicas are avaialble for:
    * MySQL
    * Postgresql
    * MariaDB
    * Oracle
    * Aurora
  * Used for scaling **NOT** DR
  * Must have automatic backups turned on in order to deploy a read replica
  * Can have up to 5 read replica copies of a DB
  * Can have read replicas of read replicas but watch out for latency
  * Each read replica will have its own DNS endpoint
  * Can create read replicas of Multi AZ source DBs
  * Read replicas can be promoted to their own standalone databases (this breaks replicatoin however)
  * Can have a read replica in a second region
    
### RDS Backups, Multi-AZ, and Read Replicas - LAB

* If you go to your RDS and select the Modify drop down, there is an optoin for "Create Aurora Read Replica" which is a good way of migrating a MySQL DB over to Aurora. Just create a read replica.
* Can create a snapshot, and migrate snapshot

1. Click Modify
2. Turn on Multi-AZ (yes)
Note that a production DB should already be a Multi AZ but if not, this is something that should be done during a maintenance window
3. In this lab we'll Apply Now
4. Now under the Configuration sub tab, Multi AZ will say "Yes"
  * Under Actions > Reboot we have the option to "Reboot with Failover" which forces the AZ to change
  * Can create Read Replica under the Actions menu of the RDS > Databases page. Not the RDS instance page itself. But recall that Automatic backups must be enabled for this.

5. Turn on backups from the RDS instance page: RDS > Databases > [RDS Instance Name]. Click Modify. Configure a backups schedule/window.
6. Once it's done modifying the RDS instance, create a Read Replica uder the Actions drop down menu
7. Provide a read replica instance name. Leave the rest as default, and create the replica
8. Will now see two RDS instances listed. Onc with a role of Master and one with a role of Replica

* Note that under the Actions menu of the Replica, you have to option to "Promote read replica"
* Popular exam scenario is migrating from MySQL to Aurora

EXAM TIPS:

* Two different types of backups for RDS:
  * Automated
  * Snapshots
* Read Replicas:
  * Can be multi AZ
  * Used to increase performance
  * Must have backups turned on to create read replicas
  * Can be in different regions
  * For MySQL, Postgresql, MariaDB, Oracle, and Aurora (NOT SQL Server)
  * Can be promoted to Master but this will break replicatoin 
* Multi AZ:
  * Used for Disaster Recovery
  * Can force a failover from one AZ to another by rebooting the RDS instance
* Encryption is supported for MySQL, Oracle, SQL Server. Postgresql, MariaDB, and Aurora. Encryption is done through AWS KMS. One your data is encrypted pretty much everything else is done encrypted as well (backups, replicas, and snapshots).

### DynamoDB

* *What is:* Is Amazon's NoSQL solution. Opposite of RDS. It is a fast and flexible NoSQL DB service for all applications that need consistent, single-digit millisecond latency at any scale.It is a fully managed DB and supports both document and key-value data models. Its flexibledata model and reliable performance make it a great fit for mobile, web, gamin, ad-tech, and IoT.



 ## AWS CLI Reference, 

* `aws s3api list-buckets`
* `aws s3 rb s3://elasticbeanstalk-us-west-1-xxxxxxxxxxxx`

**Head-Bucket:**
This operation is useful to determine if a bucket exists and you have permission to access it. The operation returns a 200 OK if the bucket exists and you have permission to access it. Otherwise, the operation might return responses such as 404 Not Found and 403 Forbidden .

* `aws s3api head-bucket --bucket elasticbeanstalk-us-west-1-xxxxxxxxxxxx`

If the bucket exists and you have access to it, no output is returned. Otherwise, an error message will be shown. For example:
`A client error (404) occurred when calling the HeadBucket operation: Not Found`



 
