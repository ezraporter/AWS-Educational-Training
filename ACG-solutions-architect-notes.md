# Ch 3 AWS Fundamentals

## Building Blocks of AWS

Region = geographical area
Availability zone = data center, building(s) filled with servers

Geographical location consists of 2+ avilability zones

Edge location = AWS endpoint for caching content. Many more ELs than AZs

Region > AZ > EL

Massive amount of services available in a number of categories but we only need to know a few

## Who Owns What?

*Owns* means who is responsible, you or AWS. Governed by "shared responsibility model"

AWS: security **of** the cloud
- Physical hardware, data centers, etc.

Customer: security **in** the cloud
- Customer data, traffic in your AWS instance, etc.

Tip: If you can do it in AWS console, you are probably responsible

Encryption is a shared responsibility

## Compute, Storage, Databases, Networking

Layout of what's to come. Format is:

Service Category
    - Service 1
    - Service 2
    - ...

Compute: actual power to process data
    - EC2 (virtual machines)
    - Lambda (serverless)
    - Elastic Beanstalk

Storage: place to put data
    - S3
    - EBS
    - EFS
    - FSx
    - Storage Gateway

Databases: reliable way to retrieve data
    - RDS (relational)
    - DynamoDB (non-relational)
    - Redshift (warehousing)

Networking: way to let compute, storage, DB services talk to eachother
    - VPSCs
    - Direct Connect
    - Route 53 (DNS)
    - API Gateway
    - AWS Global Accelerator

## What is a Well-Architected Framework?

Read AWS WA Framework whitepaper at the end of course

5 Pillars:
1. Operational excellence
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization

# Ch 4 Identity and Access Management (IAM)

## Securing the Root Account

IAM allows us to manage users and access to content in AWS

Root account is the email address used to sign up for AWS. Has full admin access
- At CHOP is there a single root account?

IAM is a service under Security, Identity, & Compliance

IAM > Rotating credentials OR username in menu bar > My Security Credentials

Most important step: Turn on MFA

## Controlling Actions w/ IAM Policy Docs

Assign permissions with policy documents
- Stored as JSON

Ex:

{
    "Version": "2020-1-1"
    "Statement": [
        {
            "Effect": "Allow",  # What to do
            "Action": "*",      # How?
            "Resource": "*"     # On what?
        }
    ]
}

* = everything

Can assign policy docs to:
- Groups
- Users (Hard to manage to don't do this. Assign user to group)
- Roles

In console: IAM > Access Management > Policies

AWS logo -> AWS manages the "default" policy

Can create custom policies but usually won't need them

Study tip: Check out random policy docs in AWS

## Permanent IAM Credentials

Building blocks:
- User: a physical person
- Groups: collection of users, corresponding to job functions
- Roles: Internal usage within AWS, easier to understand with examples

Best practice:
1. Let users inherit permissions from group
2. No shared users. One user = one person
3. Principle of Least Privilege: Assign the minimum access needed to perform job

Adding a user:
IAM > Access Management > Users > Add User

Access types:
Programatic - ability to access from command line or API. Faciliated w/ access key ID + secret access key
Console - basic console acces w/ password

Tags allow you to associate data with a user for later tracking

Password policy sets standards for passwords

IAM Federation: Mechanism for combining existing account (like computer log on) with AWS so credentials can be shared

New users are created with no permissions

IAM doesn't apply to specific regions, it's global

## Lab Notes

ARN = Amazon resource name
- Uniquely identifies a resource in AWS
- Users are resources? What else is?

Inline policy is specific for a group. Managed policies can be applied to different groups/users

Action: "Deny" = explicit deny, will always override and allow

# Ch 5 S3

Object based storage in AWS. Can upload any file type but should be static files

Unlimited storage. Objects up to 5TB

Bucket = storage location, like a directory
- Sahred namespace between all AWS accounts

key = filename, value = data

High availability + durability

Tiers:

1. Standard

Lifecycle management = way of transitioning objects to cheaper tiers or removing

Encryption

"Read after write consistency" - newest version of file can be read immediately after writing

## Securing Bucket w/ Block Public Access

Object Access Control List (ACL) controls access to an individual object in a bucket
Bucket policy sets access at the bucket level

Default is to have public access blocked to bucket and ACLs disabled (no object-level permissions)

Steps to make object public:
1. Remove block public access from bucket settings
    - Doesn't make everything public but allows capability to make specific objects public
2. Enable ACLs
    - Allows object-level permission setting
3. Make public with ACL

Now anyone can access using obect URL

Key: Need to allow public access at both bucket and object level to make objects public on an individual basis

Next section introduces bucket policies to make everything in the bucket public

## Hosting Static Website w/ S3

S3 is only good for static websites. Nothing with a server

Automatic scaling means S3 scales to meet demand by default


We'll store website content in S3 bucket
Use bucket policies to make all content in the bucket available

First, remove block public access. Then add bucket policy to make public

## Versioning Objects

Once enabled:
- All versions available
- Cannot be disabled, just suspended
- Supports MFA

Properties > Bucket Versioning

Versions get new url with -version_id appended

Public bucket policies don't make old versions public by default. Enable at version level (object actions)
    - Can this be done in batch? Maybe there's specific policy settings that do this

Once deleted an objects versions show up as a version w/ type = Delete Marker
To restore, delete the delete marker

## S3 Storage Classes

For exam: Need to know different storage classes and which is best for which situation

S3 Standard
- High availability + durability
- Redundant storage across AZs
- Good for frequent access

S3 Standard-IA (Infrequent Access)
- Good for infrequent access but needed fast when requested
- Pay per access
- Ex: Backups, recovery files
- Slightly lower availability (99.9% vs. 99.99%)
- Same durability (11 9)

S3 One-Zone-IA
- Like S-IA but data is stored in a single AZ -> less availability (99.5%)
- 20% cheaper than S-IA

S3 Intelligent-Tiering
- Moves data to most cost effective tier based on usage
- Good if you will have a mix of frequent and infrequent access
- Same availability + durability as S (99.99/11 9)

Glacier Options (for archiving):

Glacier Instant Retrieval
- Long-term archiving, instant access
- Pay per access

Glacier Flexible Retrieval
- No cost access
- Retrieval time can be up to 12 hours
- Cheaper if you can afford to wait

Glacier Deep Archive
- Cheapest storage class
- Standard retrieval is 12 hours but up to 48 for

**Summary**
Availability/Durability:
- Everything is 11 9 durability
- Availability: Std/Glacier > IT/IA > 1-Zone-IA
    - Design: 99.99 / 99.99 / 99.5
    - SLA: 99.99 / 99 / 99
- Everything stored in 3+ AZ except for 1-Zone-IA

Cost:
- Retrieval: Std/IS > Others
    - No fee/per GB fee
- Storage: Glacier > 1-zone-IA > IA > IT > Std
    - Glacier: DA > FR > Std

## Lifecycle Management

Automates moving data between tiers

Works with versioning -> have different versions at different tiers

To enable: Management>Lifecycle rules

Need to specify a rule like:
Move to [Tier] after X days, then move to [other Tier] after X days

## S3 Object Lock and Glacier Vault Lock

S3 object lock:
- write once, read many (WORM) model
- Prevent object from being deleted/modified for specific amount of time
- Governance mode: require special permissions to delete and object or alter lock settings
- Compliance mode: locked objects' protections can't be overridden by any user
- Retention period fixes amount of time object can't be deleted/overwritten for
- Legal hold allows for indefinite retention
- Can be applied at object or bucket level

Glacier vault lock:
- Apply lock policies to individual Glacier vaults ... what is a Glacier vault?

## Encryption

Encryption in transit
- When object is being transferred
- SSL/TLS; HTTPS

Encryption at rest: server-side
- SSE-S3: AWS manages it all
- SSE-KMS: AWS provides key but customer needs to work with it
- SSE-C: Customer provides keys

Encryption at rest: client-side
- Encrypt yourself before uploading to S3

Enforcing SSE can be done:
1. Through console with checkbox
2. Through bucket policy
    - Works by denying PUT requests that don't include headers telling AWS to encrypt

## Optimizing Performance

Prefix allows folders within a bucket. Ex: bucket/folder1/subfolder1/file.txt

S3 can do ~ 3500 requests per second per **prefix** -> better performance by spreading requests across prefixes

KMS encryption has additional limitations
- Region specific quota per second
- Doesn't scale w/ prefixes

Parallelizing upload/download:

Multipart upload:
- recommended for > 100MB
- required for > 5GB
- can improve performance

Download byte range fetches:
- Asks AWS to send file in parallel X byte chunks
- Failure in download only affects specific byte range
- Also useful for partial download (i.e to check headers)

## Backing up Data w/ Replication

Method of replicating objects from one bucket to another

Versioning must be on in both buckets

By default, delete markers are not replicated
- i.e. deleting in the source bucket does not delete in destination bucket

Replication only takes effect once it's turned on. Existing files are not replicated

Can be applied to entire bucket or specific objects



Properties>Management
- IAM role = Create from existing > Create new
    - Not sure what this does but we'll find out

# Ch 5 EC2

EC2 are essentially virtual machines in the cloud
Can be scaled as needed

4 Pricing Options:

1. On demand - pay by hour
    - Flexible
    - Good for short-term use and during development
2. Reserved - 1-3 year contract, pay more up front -> lower hourly rate
    - Predictable usage
    - Specific capacity reqs
    - Standard RIs give biggest saving
    - Covertible RI - more expensive but can switch to type of equal or greater value
    - Scheduled - Fluctuate capacity during specific times of day
    - Applies within region -> can't switch to a different one on same contract
3. Spot - Purchase unused capacity but price fluctuates
    - Flexible start and end times
    - Urgent need for lots of computing
4. Dedicated - Physical server, most expensive
    - Sometimes need for compliance
    - Can be purchased on demand and reserved but will be more expensive

## AWS Command Line

AWS CLI is alternative to online GUI

`aws SERVICE_NAME COMMAND`

SSH to connect to running EC2 instance from command line

```
$ chmod 400 KEY_FILE.pem # change permissions on file?
$ ssh ec2-user@PUBLIC_IP -i KEY_FILE.pem
```

Inside EC2 inst run `aws configure` to authenticate

Need to create access key for programmatic access (see IAM)

List S3 buckets:
```
aws s3 ls
```

Make a bucket:
```
aws s3 mb
```

Upload a file:
```
aws s3 cp FILE BUCKET
```

## Using IAM Roles

Role is similar to a user but meant to be assumed by anyone with access, i.e. multiple people to a role

Roles are temporary

Roles can be assumed by users or other AWS things

Permissions are managed by JSON policies like normal

Workflow:
Assign role to EC2 instance which allows it to connect to S3 bucket on your behalf

Alternative to creating keys that will give a resource persistent object. Roles are preferred for this

## Security Groups

By default an EC2 instance has no inbound access to any outside sources. Need to open up ports with a security group to allow access

Outbound access is allowed by default

Security groups are reusable so you can attach them to multiple EC2 instances

Security group access changes take effect immediately

`0.0.0.0/0` = IP address range that allows everyone in
In production you'd only do this for HTTP/S ports

Bootstrap script runs when instance starts at root level
- Adds more time to startup but automates installation of what you need
- Configure Instance Details > Advanced Details: User data
- "user data" is a synonym for bootstrap script

Note for demo `/var/www/html` is the default root dir for web server on linux


## EC2 Metadata

Data about your EC2 instance

Accessible via http within EC2 instance at IP 169.254.169.254

```
curl http://169.254.169.254/latest/meta-data
```

Ex. use it to dynamically grab instances IP

```
curl http://169.254.169.254/latest/user-data
```

to recover bootstrap script (aka user data)

"we can use user data to query metadata"

## Networking w/ EC2

3 different types of virtual networking cards:
1. Elastic Net Interface (ENI)
    - basic option
2. Enhanced Networking (EN)
    - high performance via SR-IOV
    - Elastic Network Adapter (ENA) or Intel VF
        - Basically always use ENA. VF has lower ceiling on transfers speed and is used on older machines
3. Elastic Fabric Adapter (EFA)
    - High performance computing for for machine learning
    - OS bypass allows direct communication with EFA on linux and can be used to increase speed

## Optimizing w/ Placement Groups

Placement group is a way of grouping EC2 instances together

3 types:
1. Cluster placement group
    - Group of instances within a single availability
    - good for high throughput and availability
2. Spread placement group
    - Each instance on separate hardware
    - Good for small number of critical instances. Want them to be separate to increase reliability
3. Partition placement groups
    - Each placement group has its own set of racks with separate power sources
    - Multiple EC2 instances?

## Solving Licensing Issues

Recall diff pricing options. Dedicated hosts are the solution for issues with special licensing reqs

## Timing workloads w/ spot instances

Recall: spot instances use excess compute capacity as disocunt prices

To use, you name your max spot price. Instance is provisioned as long as price is below spot max.
If spot price goes above max, you get 2 minutes to decide whether to stop or terminate instance

Spot block is a mechanisms for keeping instances up if price increases above max. Can be set for 1 to 6 hours

Pricing history is available down availability zone level

Good use cases: CI/CD, high performance computing
Bad use cases:
    - persistent workloads like webservers or dbs
    - critical jobs

2 types of spot requests:
1. one time
    - instance is stopped if interrupted
2. persistent
    - keeps provisioning as long as price is low enough and spot request is active
    - make sure you cancel request, don't just stop instances

Spot fleet = collection of spot instances + maybe on-demand isntances

Specify target capacity in fleet request and this attempts to meet capacity within constrains using a combination of spot and on-demand

Spot fleets provision instances as long as you're below price constraint and capacity target using a selected strategy

## VMware Cloud (new)

VMware already used by other orgs

Use cases:
- Hybrid cloud: connect on-prem private cloud to public cloud
- Cloud migration (VMware has good built-in tools)
- Disaster recovery
- Like VMware but want AWS capabilities

Dedicated hardware that runs a bunch of VMware instances

## AWS Outposts (new)

Bring AWS services to on-premisis data center
Use cases: hybrid cloud

2 types:
Outpost Rack vs Outpost Server
- Rack coonsists of many servers so you'd use it with a lot of space
- Server gives you just local compute
- Rack gives you basically all AWS services

# EBS and FBS

## Elastic Block Storage (EBS)

Virtual storage for EC2 instance like a hard drive

Use cases: mission critical storage
    - high availability
    - spread across multiple locations in availability zone

Can increase volume with no downtime

Types:

1. General purpose SSD
    - gp2 is older generation, gp3 is newer
    - Default storage, good for booting an OS
2. Provisioned IOPS SSD
    - io1 and io2
    - high performance, need > 16k IOPS
    - Good for things like large databases or anything where latency is an issue
3. Throughput Optimized HDD (st1)
    - Throughput is more important for large data loads, ETL, big data, etc
    - Can't be used as boot drive
4. Cold HDD (sc1)
    - baseline throughput option
    - lowest cost storage

IOPS vs. Throughput

IOPS = read/write operations per sec
Important for quick transactions, low latency
Think online store

Throughput = number of bits read/written per sec
Analytic workloads like large datasets, complex queries

## Volumes and Snapshots

At least one volume per EC2 instance. Root volume = where OS is installed

Snapshots are copy of volume at a point in time stored in S3. Incremental so that only changes from last snap are stored

Recommendation: stop instance and take snap
    - Snaps only capture data in EBS so things in memory will get missed

Snapshots of encrypted vols are automatically encrypted

Snaps can only be shared in the region they were created in

EBS vols are always in same AZ as EC2 instance they're attached to

Resize or change types on the fly

Use snaps to move EC2 instances to other regions:
1. Create snap (Volumes>Actions>Create Snapshot)
2. Copy to new region (Snapshots>Actions>Copy)
3. Make new image (Snapshots>Actions>Create Image)
4. Launch EC2 from image (AMIs>Launch)

## Encryption

Can use KMS (Amazon run) or CMK (customer run)

Encryption has minimal performance impact

Once you encrypt, everything is encrypted: data at rest, data in transit, snaps, images from snaps

Exam topic: copying snaps allows encryption during copy. Best (only?) way to encrypt an unencrypted volume

1. Create a snap of the root vol
2. Copy and select encrypt
3. Create image from encrypted snap
4. Launch

## EC2 Hibernation

What's the difference between stopping and terminating EC2 inst?

Stop = Data in storage is kept on EBS and available when started
Terminate = By default data is lost unless we elect to save it

Hibernation
- Save contents of RAM to disk and persist it in EBS
- On restart, EC2 inst is restored to previous state and processes that were active keep running
- Advantages: much faster startup when booting from hibernation
- Some limitation on size of RAM
- Max hibernation of 60 days

## EFS

File storage that can be shared across EC2 instances

Linux AMIs only

File system scales automatically as more space is needed

Performance types: General purpose or max IO

Storage tiers:
1. Standard- frequent access
2. Infrequent access

Can also set up lifecycle management rules

Uses NFS protocol (native Unix/Linux file server protocal)

## FSx

File system solution for Windows
Useful is you want to migrate Windows file system to AWS

Uses SMB protocol as opposed to NFS

FSx for Lustre for high performance computing

## AMIs: EBS vs Instance Store

Recall that an AMI is the blueprint for an EC2 instance

AMIs are backed by either EBS or image store

EBS-backed
- Root device launched from EBS volume created from snapshot
- Can be stopped and data is preserved
- By default EBS volume is deleted when instance is terminated but we can opt to have it kept
Instance Store
- Root device launched from template stored in S3
- "Ephemeral storage". Instance can't be stopped, only terminated.
- When terminated data is lost

## AWS Backup

Centralized backup system covering multiple AWS services 

# RDS

Cloud-based relational databases. Basically an EC2 instance where we only have access to the DB running on it

6 different database engines

Good for OLTP, not OLAP

OLTP vs. OLAP

OLTP (Online Transaction Processing): Proccess many small transacation in real time
OLAP (Online Analytics Processing): Proccess large, complex queries that may take time

Multi-AZ support: AWS creates a copy of database in a second AZ
- 5/6 engines can be configured as single or multi-AZ. Aurora is always multi-AZ by default
- AWS handles failures of primary db and automatically re-routes traffic to secondary AZ

## Read Replicas

Use to increase performance

Creates a read-only copy of database that can be used for read heavy operations

Don't confuse with multi-AZ. multi-AZ is for disaster recovery, read replicas are for performance

Each replica has its own DNS endpoint

Possible to promote replica to its own primary database with write permission

## Aurora

One of the 6 RDS engines, developed by Amazon
MySQL/PostgreSQL compatible

Automatic scaling of storage and compute

3 types of read replicas:
- Aurora, MySQL, PostgreSQL
- Aurora replicas have better performance and you can create more of them
- Aurora replicas have automatic failover if primary db fails

Automated backups that can be shared with other AWS accounts

Serverless option
- Good for infrequent and unpredictable workloads
- Still has same performance as traditional option

## DynamoDB

NoSQL database developed by Amazon

Read consistency types:
Eventually consistent = consistency across all copies of data within 1 second. Best performance
Strongly consistent = results refelct all writes

Default is eventually but can opt into strongly

DAX = in-memory cache within DynamoDB

Encryption at rest with KMS

Stored on SSD storage

Spread across 3 distinct data centers

## ACID Databases

ACID:
Atomic = All changes successful or not made at all
Consistent = Consistent state before and after transaction
Isolated = Nothing else can change data while it's running
Durable = Changes made persist

Dynamo DB transactions support ACID

Key: ACID is all or nothing. If one part of the transaction fails it all fails

## DynamoDB Backups

On-demand backup allows full backup at any time. Same region as source table

Point-in-time-recovery- restore to any point in time in the last 35 days
- Needs to be enabled

## Streams

Stream = ordered sequence of db changes over time
Streams are stored for 24 hours

Allows for replication of db tables across multiple regions

# VPC

VPC is like a virtual data center in the cloud

Region > VPC > Subnet

Subnets all exist within a single AZ

Example setup:
Web - outside access
Application - Private subnet that can only speak to web and database
Database - Private subnet that can only talk to application layer

Choose IP address range- typically 10.0.0.0/16 at orgs
/16 indicates the size of the range. Large # -> smaller range
- 16 is largest for VPCs

VPC components:

- Router
- Route table, 1 per subnet
- Network ACL, 1 per subnet
- Subnets
- Internet gateway
- Optionally, Virtual Private Gateway

AWS comes with a default VPC but configuring your own VPC allows for more customization

## VPC Demo

1. Create a VPC
    - automatically creates new route table, ACL, and security group
2. Create subnets
    - Only one AZ per subnet
    - Need to enable autoassign of public IP addresses if we want it. Ex. for our public subnet
3. Create internet gateway
    - Attach to VPC
4. Edit route table to create route out to internet
    - By default new subnets get associated with main route table. If we only want one subnet to access out we need to create a new route table

AWS reserves some IP addresses for each subnet

Difference between route tables and security groups?

EC2 -> security group
    -> subnet
        -> route table
            -> internet gateway

For an EC2 instance to connect to the internet it needs to be:
1. In a subnet connected to an IG (via route table)
2. Attached to a security group with the right inbound permissions

What would happen if we have only 1 of those?

## NAT Gateways

Secure way of letting instances in private subnets connect to internet, ex. for updates/patches

Basically allows connections but stops external sources from initiating a connection

NAT gateway goes in public subset so that private instances can connect there and then use public route to talk to internet

Key facts:
1. Associated with one AZ (because associated with one subnet)
2. Scalable transfer speeds
3. AWS handles upkeep
4. Not associated with security group we need to worry about

Connect NAT to private subnet using route table of the private subnet:

NAT is associated with public subnet. Connection to private subnet EC2s configured through route table for private subnet

EC2 -> subnet (private)
        -> route table
            -> NAT
                -> subnet (public)

## Security Groups (Review)

Virtual firewall for an EC2 instance

Security groups are stateful:
- If you make an outbound request, response traffic will be allowed to flow in regardless of inbound rules
- And vice-versa: Responses to allowed inbound reqs are allowed to flow out

## Network ACLs

ACL = Access Control List

Extra layer of security that controls access in and out of one or more subnets

ACL sits between route table and security group

Default VPC comes with ACL that allows everything
Custom VPC comes with ACLs that deny everything by default

Every subnet is associated with an ACL (default if not explicit) and only 1
    - Multiple subnets can have same ACL

Block IP addresses with ACLs not SGs

ACLs are stateless. Traffic is blocked or allowed based on where it came from not why it came
    - Contrast with SGs that have different behavior depending on whether traffic is response to a request

ACL consists of rules associated with a rule number:
100
200
*

They get evaluated in ascending order with * last. Once a rule applies we stop looking so any conflicting rules with higher numbers are ignored

See AWS docs about ephemeral ports

## VPC Endpoints

Private connection from VPC to AWS services without leaving the AWS network

Useful because they don't put burdens on your network traffic (as connecting via NAT would)

2 types:
1. Interface endpoint - private IP address that serves as entry point to AWS service
2. Gateway endpoint - endpoint needs to be provisioned
    - Supports S3 and DynamoDB

Similar to NAT, you associate endpoint with route table after provisioning

## Peering

Method for connecting VPCs so they can act as if in the same network

Possible to peer between regions and between AWS accounts

VPC peering only supports direct connections. Ex:

A <-> B <-> C

A and C can't talk unless a connection is established between them

## PrivateLink

Another method for sharing information across VPCs

Consider alternatives:
1. Open VPC to internet
    - Okay but everything in public subnet is public and you have to manage a bunch of stuff
2. VPC Peering
    - Need to manage many peering relationships
    - Entire VPC is public to peers

PrivateLink is good if you only want to share a particular piece of VPC and want to do it with many other VPCs

Needs: 1) Network load balancer on service VPC 2) ENI on customer VPC

## VPN CloudHub

Way of connecting multiple sites with own VPN connections

## Direct Connect

Dedicated network connection allowing connection between cloud and on-premisis data

Dedicated connection = physical ethernet connection to your data ceneter
Hosted connection = Physical connection provisioned by AWS partner

Direct Connect is faster and more reliable than VPN for getting private communication between cloud and physical servers

## Transit Gateway

Connects VPCs and on-prem data in a hub-and-spoke model

AWS transit gateway serves as a central router for connections and VPCs or Direct Connect connect to that

Enables transitive peering (which VPC peering can't do)

# Route 53

DNS = Big lookup to connect domain names to IP addresses

Top level domain = most general piece of address, ex. .com

DNS record types:
SOA (start of authority) ??
NS record
- Name of DNS server where records can be looked up
- "authoritative DNA server" for the record
'A record'
- fundamental record to translate domain to IP
CNAME (canonincal name)
- Map one domain name to another
- Can't be used for unprefixed domain name
Alias record
- Like CNAME but for mapping between AWS resources. Unique to AWS
- Can be used for unprefixed domain name

Components of DNS records:
TTL (time to live)
- amount of time DNS record can be cached
- If you update DNS record, may not take effect until after TTL is expired
- Low TTL -> changes to DNS propogate faster through internet

Routing policy
- Determine how routing is handled for a given DNS record

## Simple Routing
Can have multiple IPs
User gets all IPs returned in a random order
Only 1 record with multiple IPs allowed

## Weighted Routing
Assign weights to different IP addresses
Ex. 90% to IP1 and 10% to IP2

Health checks allow record to be disabled if not working

You create different DNS records for each IP and give it a weight

## Failover Routing
Assign a backup server if the primary fails
You set up a health check on primary and routing policy reroutes to secondary record if it fails
Different record for each IP
"Failover record type" stored whether record if primary or secondary

## Geolocation Routing
Choose where traffic is sent based on location of users
Use case: Want location-specific content on our web servers
Supports routing by continent or country

## Geoproximity Routing
Route 53 Traffic Flow allows you to define custom rule combinations to route traffic
Geoproximity routing only available in Traffic Flow
Route traffic based on proximity to an index location
"bias" allows you to expand or shrink sizes of zones

## Latency Routing
Route traffic based on response time
In the DNS record, specify the region where the associated reosurce is
    - Does R53 just use this info to determine latency?

## Multivalue Answer Routing
Return multiple IP addresses to DNS query but use health checks to ensure only healthy IPs are returned
Basically simple routing with health checks
Each IP gets its own DNS record

# ELB

Automatically distribute traffic across multiple targets such as EC2s

3 types:
1. Application balancer
    - best for HTTP traffic
2. Network balancer
    - higher performance
3. Classic balancer
    - legacy, best for testing

Health checks allow automated checking of status of instances behind balancer
When unhealthy, ELB stops routing traffic to instance

## Application Load Balancers

Layer 7 load balancing
- Review OSI model

Listeners:
- check for client connections and use config rules to route them
- assigns a listener for each port/protocol you want to support

Rules consist of:
- priority
- 1 or more action
- 1 or more condition

Parh-based routing:
- Routing rules based on the URL path users are requesting
- Ex. send example.com here and example.com/test there

Limitations: only supports HTTP/S
For HTTPS we need TLS/SSL cert on our ELB

## Network Load Balancer

No rules like with app load balancer
Use case: high traffic, non-HTTP/S protocols

## Classic Load Balancer

X-forwarded-for:
- Header allowing you see IP of requester to ELB
- Classic balancers give local IP to EC2s when passing traffic

Classic LB uses 504 for gateway timeouts -> means application isn't responding

## Sticky Sessions

Classic LB allows you to bind a user to an EC2 instance. Ex. we can make sure particular IP always gets back to same EC2 they first visited

Often problematic because if particular instance goes down the user can't connect

Sometimes useful if storing data locally

## Dregistration Delay

Allows existing connections to stay open if EC2 instances are deregistered from target group

Can be disabled so existing sessions will drop immediately

"Connection draining" for classic LB

# Cloud Watch

Tool for monitoring performance of AWS architecture

System metrics (default): get these out of the box with services we use
    - The more managed a service is the more you'll get out of the box
Application metrics (custom): need to install CW agent to get additional info about what's going on in an EC2

Remember: EC2 memory utilization and EBS capacity are custom metric

Important exam topic: Is this metric default or custom?

Basic/standard monitoring = checks every 5 min
Detailed monitoring = every 1 min

## CloudWatch Logs

Tool for monitoring, storing, and accessing log files from all different sources

Log event = record of what happened in a given log at a given time
Log stream = collection of log events from a **single source** (ex. 1 EC2 instance)
Log group = collection of streams that make sense together (ex. set of EC2 instances serving the same purpose)
- User defined?

CloudWatch Logs Insights is a service allowing you to make queries against your logs

Need to assign appropriate role to resouce to allow it to stream to CWL

Good for semi-realtime uses:
- If you just want to store, use S3
- If you want realtime use Keneses

Need to configure CWL agent, not available by default

## Amazon Managed Prometheus and Grafana

Grafana is an open source data viz tool used for monitoring operational metrics
Prometheus is a service for container monitoring

AWS manages setup, scaling, maintenance for both services

# High Availability and Scaling

Vertical scaling: increase capacity of particular resource like EC2
Horizontal scaling: add more instances of your resource types

What do we scale: EC2, database?
Where do we scale: where in the VPC, which AZs?
When do we scale: what triggers scaling

## Launch Templates and Configs

Launch template contains all settings that need to be chosen to deploy EC2
Launch configurations have similar functionality but support less stuff
- Can only be used for auto-scaling
- Immutable
- No versioning

Templates always > configs

Elements of a template:
- AMI
- instance size
- security groups, optionally
- network, optionally
    - If provided in the template we won't be able to use in autoscaling group
    - Autoscaling group needs to be able to set VPC and subnet

## Autoscaling Groups

Autoscaling group is a collection of EC2 instances that get treated as a collective

Components of an autoscaling group:
- Launch template
- Networking and purchasing info
    - Want to have at least 2 AZs for availability
    - What tier of compute?
- ELB settings
    - Can be set to respect health checks of load balancer -> if instance fails, group will terminate it and redeploy
    - By default the group will only use system level EC2 checks
- Scaling policy
    - Min, max, desired compute capacity
- Notifications with SNS

Remember: autoscaling groups are only for EC2s. Other services have other scaling systems

## Autoscaling Policies

Scaling policy allows desired capacity to move up and down according to utilization metrics like memory utilization

Step scaling allows us to set simple policies for adding and terminating instances, ex.:
- when memory utilization > 80% add 15 instances

warmup period = amount of time where instances are allowed to come online without passing health checks

while in warmup period, autoscaling won't trigger additional instances to meet demand that will be filled by those warming up

cooldown period = min time between scaling events
- when scaling occurs we always wait set time before doing anything else
- default is 5 min

Reactive scaling = measure load once it's there and apply scaling policy
Scheduled scaling = create scaling event on schedule if there's predictability to load

Scale out aggressively, scale in conservatively

The more you can put in AMI the faster warmup time is

Reserved compute for minimum capacity, scale out with spot and fall back to on-demand

## Scaling RDS

Types of scaling:
1. Vertical - resize CPU to get better performance
2. Storage - increase storage capacity, can only go up not down
3. Read replicas - sort of horizontal scaling but only good for read operations
4. Aurora serverless - let AWS manage scaling if it's unpredictable

Nonrelational DBs are often more scalable so switching to that is a good option

## Scaling Non-RDS

AWS handles more scaling with DynamoDB

Provisioned: set upper and lower bounds for usage
- Good for predictable workloads
- Cheapest option

On-demand: pay per read/write and AWS handles scaling
- Good for sporadic workloads
- Generally more expensive

Can only switch between provisioned and on-demand once every 24 hrs

# Decoupling Workflows

Tightly coupled = Layers of application depend on eachother such that if one resource goes down the entire app goes down

Loosely coupled = Layers have redundancy so a resouce can go down and the app stays up

EC2 instances should never talk with eachother directly. Always spread communication across something that can route to multiple EC2s like an ELB

## SQS

Poll-based messaging
- Producer creates message and puts it in queue
- Consumers can pick it up whenever
- Key is asynchronous processing, no need to reply immediately

General pattern:
1. Producer puts a message in the queue
2. Consumer polls queue and gets message
3. Consumer reports back to queue saying message was processed so it can get deleted

SQS settings:
- Delivery delay
    - Amount of time the queue waits after receiving message to make it available to consume
    - default is 0 but can be up to 15 min
- Message size: Up to 256 kb of text
- Encryption
    - By default in transit
    - Can opt into at rest encryption
- Message retention
    - Amount of time message will stay in queue
    - Default is 4 days but can be 1 min to 14 days
- Polling type
    - Short polling (default) = receivers disconnect if there is no message in queue
        - Downside: more CPU cycles and API calls = more $$$
    - Long polling = receivers wait for a message for set time before disconnecting
        - Usually preferred
- Visibility timeout
    - Amount of time consumers have to report back that message was processed
    - In this period the message is hidden from the queue but will reappear if consumers doesn't report back before timeout
    - Default is 30 seconds

Queue depth = number of messages in queue
Automatically montitored by cloud watch and can be a metric to kick off auto scaling

## Sideline Messages into a Dead-Letter Queue

Dead-Letter queue is a mechanism for separating messages that fail to process from the main queue so that reprocessing them isn't attempted

Need to create DLQ before primary queue because we need to select DLQ when creating primary queue

Max receives: Setting of how many times a message can be received before being sent to DLQ
- default is 10 but can be 1 to 1000

DLQ is just another SQS queue so it's subject to all the same settings/limits/defaults

## FIFO SQS Queue

Standard SQS has "best effort ordering"
- Messages may not come out of queue in the order they went in and occasionally messages are duplicated
- Nearly unlimited transactions per sec

FIFO = first in first out
- Messages are guarenteed to come out in the same order they go in
- Limited to 300 messages/sec and slightly more expensive

FIFO queues end in .fifo

## API Gateway

Front door to applications as a managed service

Can be fronted with web app firewall (WAF) to secure against malicious use

Supports versioning

## AWS Batch

Perform batches of jobs with a serverless setup
Can use EC2 (unmanaged) or Fargate (managed) compute
Has less limitationa than lambda

## Step Functions

Event driven serverless orchestration tool

## AppFlow

Managed service for transferring data to/from SaaS applications

Ex. Transfer salesforce data to AWS storage

# Security

## DDOS Attacks

Layer 4 attack = attack at the transport layer (TCP)
Basic TCP: client send SYN packet -> server sends SYN-ACK -> client sends ACK
Layer 4 attacks sends a large number of SYN packets and overload server

Amplification attack = third party server is sent a spoofed IP address with our server's IP so it responds to us and overwhelms server

Layer 7 attack = server is sent a large number of HTTP requests

## CloudTrail

Service for logging API calls to AWS account. Basically what occurred on command line or in console

Doesn't log SSH/RDP traffic

What is logged:
1. Time, source, and content of request
2. Content of response

Useful for after the fact investigation of security incidents

Logs are stored in S3

## Shield

Free DDOS protection for ELB, R53, CloudFront

Protects against layer 4 attacks

AWS Shield Advanced provides more comprehensive coverage and near real time info

## WAF

WAF = Web Application Firewall
Allows you to monitor HTTP traffic going to an app
Operates at layer 7 in contrast to Shield

Can use pretty much any element of request to create conditions that will block requests

## Inspector

Tool to automatically inspect EC2 configurations to assess security

2 types of assessment:
1. Network - checks things like access to VPC on different ports
    - No agent required
2. Host - checks configs of EC2
    - Need to install agent in EC2

## KMS and HSM

KMS = managed service to generate encryption keys for data
HSM = hardware security module, device that encrypts/decrypts data

## Secrets Manager

Store credentials and access them via an API
Allows automatic credential rotation but once enabled rotation starts immediately so any hardcoded credentials will no longer work

Alternative is Parameter Store which is free but:
1. can only store up to 10k values
2. no auto key rotation

## S3 Presigned URLs/cookies

Mechanism for granting time-limited access to an S3 bucket
Allows for sharing content from private S3 buckets to select users
Presigned cookies allow for sharing multiple private resources

## Advanced IAM

ARN uniquely identifies an AWS resource

arn:partition:service:region:account_id:...

## Cognito

Authentication and user management service within AWS

Allows users to create credentials or login with third party like google or FB

Allows users access to server-side resources you specify with a token

User Pool = directory of users for use within your application
Identity Pool = directory of users for giving access to other AWS resources

Cognito sequence:
1. User connects to user pool to authenticate + get tokens
2. Connect to identity pool and exchange tokens for AWS creds
3. Use credentials to access services

## Random Services to Remember

GuardDuty = AI based threat detection for more $$$
Firewall Manager = Interface for managing firewall rules across multiple accounts in an org
Macie = ML system for identifying personally identifiable info (PII) in S3 buckets
Certificate Manager = Service for managing SSL certs.
    - Replaces 3rd party SSL providers
    - Auto-renewals
    - Works with ELB, API Gateway, CloudFront
Audit Manager = Provides continuous reports for auditors
AWS Artifact = Single source for compliance info
Detective = Investigate root causes of security incidents
Network Firewall = **Physical firewall** across VPCs
    - Useful for filtering network traffic before it reaches internet gateway
Security Hub = Single place to view security alerts from GuardDuty, Macie, Inspector, Firewall Manager

# Automation

## Cloud Formation

AWS IaC tool where you can write JSON or YAML that defines your architecture

Templates get automatically stored in S3

Key components of a template:
1. Parameters = values user can fill in at runtime
2. Mappings = defined constants used in the template
3. Resources = AWS resources to create

Failing templates often associated with hardcoded IDs -> use mappings to fill in dynamically

When templates fail they roll back to last valid state

## Elastic Beanstalk

PaaS = single-step deployment of architecture

Need to pick a platform = runtime language

Selecting docker as a platform allows us to use any custom runtime

## Systems Manager

Free service for automated management of EC2s and other resources

Requires agent to be installed in EC2 and can also be used for on-prem

Need to give EC2 instance role to be able to communicate with Systems Manager service

Key components:
1. Automation documents run templated procedures on instances
2. Session manager allows connection to instances from console

# Caching

External = cache content close to users so they don't need to go far to reach it
Internal = cache content within AWS so requests within architecture are faster

## CloudFront

CloudFront is a Content Delivery Network (CDN) which is a system that helps distribute content to edge locations/end users

CloudFront sits between users and AWS and caches requests so they can be delivered faster next time

Can front both AWS and non-AWS endpoints

Can use CF to block content from specific region but WAF is a better option since it gives more finegrained control

## Elasticache

AWS managed Memcached and Redis. Both are internal caching tools that usually sit in front of your DB

Memcached = Simple database caching solution
Redis = Database caching and can be used as a standalone NoSQL DB

Key difference is that Redis supports additional features like multi-AZ and backups

## DAX

In-memory cache for DynamoDB

Highly customizable but only for DynamoDB. Elasticache is solution for RDS

## Global Accelerator

IP caching problem occurs when user has IP address of some piece of architecture like an ELB that goes offline and is replaced

GA can sit in front of architecture to handle caching issues

Good for masking complex architecture

# Governance

## Organizations

Tool for managing multiple AWS accounts with synchronized standards

Use cases:
- Logging account, when you need centralized logs that no one can edit
- Shared reserve instances between accounts
- Shared billing

Service control policies allow for standard limits on what accounts can do
- Uses same syntax as IAM policies
- Has precdence over IAM
- Can't actually grant permissions, can only take them away. Need IAM policy to enable permission
- Unlike IAM, allow is interpreted as "allow only"

## Resource Access Manager (RAM)

Allows for shared resources across accounts. Ex. single shared VPC and subnets for all accounts in org to use

Can be an alternative to VPC peering since you can have accounts share a VPC. Usually a better option if accounts are using same region

## Cross-account Roles

IAM role that lets one account assume access to another account

Use case:
Org has a production account that developers need to build in. Grant access to a role that lets them assume account

In general using roles reduces the amount of new credentials we need to create which is more secure

## Config

Config allows for auditing of all the AWS resources that you have

1. Query exisitng resources
2. Create rules to enforce architecture patterns
3. Query history of resources

Config integrates with CloudTrail so you can crossreference historical changes in resources

Often when investigating changes to specific resource Config will be easier to digest

Integrates with system manager automation documents to automatically fix broken rules

## Directory Service

AWS managed version of Active Directory

Managed Microsoft AD = fully managed cloud AD
AD Connector = hybrid tunnel to on-prem AD
Simple AD = Barebones directory setup powered by Linux

## Cost Explorer

Service to generate reports and visualize cloud costs

Can filter spend on many dimensions including tags
- Need to enable for specific tag and then tracking is possible going forward

## Budgets

Service for creating budget plan and alerting when users are exceeding

4 budget types:
1. Cost budget = how much are we spending?
2. Usage budget = how many resources are we using
3. Reservation budget = track efficiency with reserved instances
4. Savings plan = 

In addition to alerts can also automate actions when budget thresholds are hit, i.e. stopping EC2s

## Cost and Usage Reports

Most comprehensive reporting tool for AWS costs and utilization

Automatically publishes CSVs to S3 on a daily basis

Can be used for entire organizations rather than a single account

Use cases:
- detailed reporting
- daily reports
- spreadsheet format reports

## Savings Plans and Compute Optimizer

Compute Optimizer analyzes resource utilization and offers recommendations on how to reduce costs

Covers EC2, Autoscaling EC2s, EBS, Lambda

Works for single accounts as well as organization

Savings plans give flexible prices for a number of different resources:
1. Compute- covers EC2, Fargate, Lambda
2. EC2- just EC2
3. SageMaker

Plans can be all, partial, or no upfront and have either 1 or 3 year agreements

## Trusted Advisor

Auditing tool to recommend best practices for architecture

Looks for:
1. Cost optimization
2. Performance - are things configured correctly
3. Security
4. Fault tolerance - are you covered for failures? Ex. multi-AZ
5. Service limits - are you close to hitting any

Some security + service limit checks come free but others are paid service

Trusted Advisor alerts for issues but won't fix them. Need something like lambda to do that

## Control Tower

Service for automating multi-account environments leveraging other services (IAM, Config, Orgs)

Guardrails = rules enforcing standards for AWS environment
- Preventative rules enforce standards with SCPs
- Detective rules monitor and alert using Config
    - Not supported in all regions

Account factory = template for configuring new accounts

3 shared accounts are created:
Management- does setup of configuring new accounts
Log Archive- recieves config and cloudtrail logs from managed accounts
Audit- recieves notifications from config

## License Manager

Service to manage licenses for software that might be installed on AWS
Can also be applied to on-prem software

## AWS Health

Provides info of health of AWS resources as well as availability of AWS services

Generates events that can be used for automating tasks

Events can be account-specific or public

# Migration

Some ways of getting data in/out of cloud:
1. Internet - convenient but slow and potentially a security risk
2. Direct Connect - Fast but costly especially if we're only using it for migration
3. Physical with Snow family- Send your data to AWS. Fast and cheap

With Snow family, AWS sends hardware, you fill it up with data and physically send back to AWS

Snowcone
- Smallest and cheapest option
- 8TB storage, 4GB mem, 2 CPUs
- Physically small device

Snowball Edge
- Larger device with more storage
- More computing options- GPUs, etc

Snowmobile
- Largest option to capture data center worth of data

## Storage Gateway

Service to combine cloud and on-prem data. Can work for both migration and hybrid

3 options for local config depending on type of data we're dealing with
1. File Gateway for shared file system data
    - NFS/SMB file system that you mount locally and gets backed up in S3
2. Volume Gateway for data on application servers
    - Data stored in S3 but can also integrate with EBS
3. Tape Gateway

## DataSync

Internet based transfer option for one time migration of file system data

Agent-based solution -> need to install agent on-prem

Data can be sent to S3, EFS, or SFx

## More Services

Transfer Family = 
Server Migration Service = Convert on-prem VM to AMI to use on AWS
Database Migration Service = Convert on-prem database to Aurora RDS. Works with MySQL and Oracle
App Discovery Service = Tool for planning migrations to the cloud
MGN = Automated, expedited migration tool

# Practice Notes

Read replicas as asynchronous, multi-AZ is synchronous
ALB only routes traffic to healthy instances. It doesn't do anything to fix them if they are unhealthy
Use CloudWatch to take action on unhealthy instances
QLDB = cryptographic DB
Fargate for unpredictable workloads!
Read more about scaling policies + autoscaling groups
Ephemeral EBS data is only saved in case of a reboot when there was a failure
DAX for caching with DynamoDB
AWS Trusted Advisor for best practices with new architecture. Covers security, performance, and cost
Read more about VPN
AWS Health can't automatically trigger events
Kinesis Data Analytics
Public subnet = subnet with path to internet gateway
Kinesis Data Stream vs Firehose
EBS can be used normally during snapshot
SNS?
Shield for DDoS, GuardDuty for general purpose
NLB can have static IP, ALB would require elastic IP