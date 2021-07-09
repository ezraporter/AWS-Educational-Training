# A Cloud Guru Notes: AWS Certified Solutions Architect Associate Course

**Author:** Richard Hanna

The purpose of this document is to consolidate and highlight key notes from the lessons and labs in the [**AWS Solutions Architect Associate**](https://acloudguru.com/course/aws-certified-solutions-architect-associate-saa-c02) Course on A Cloud Guru.

This markdown is broken down by chapters in the course, with subsections based on the lectures and labs.

This is an admittedly scrawled series of notes, so please excuse any typos :sweat_smile:

**Symbol Key:**

- :bulb: = AWS exam likelihood

**Key AWS Resource and Links:**

- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [AWS Well Architected Framework (5 Pillars)](https://aws.amazon.com/blogs/apn/the-5-pillars-of-the-aws-well-architected-framework/)

## AWS Fundamentals

### The Building Blocks of AWS

**AWS Global Infrastructure:**

There are **24 regions & 77 availability zones**.

- **Regions**: A geographical region (us-east-1 i.e. N. Virginia) with 2+ availability zones
- **Availability Zones**: A Data center. Big building filled with servers. Could be multiple so an event doesnt take the whole zone offline. _There are always more AZ's than regions._
- **Edge Locations**: end points for AWS caching content. Ex: download a file from New York that can be retrieved from a cacheing server in London _There are always more edge locations than AZ's (and by proxy, regions).

### Who Owns What in the Cloud?

As in the cloud practitioner notes, know that you are repsonsible for what is _in_ the cloud while Amazon is responsible for the cloud itself. Security _in_ the cloud versus security _of_ the cloud.

:bulb: **Remember this diagram:**

![AWS Responsibility Model](img/aws_shared_responsibility.png)

Think if you're able to do something _yourself in the management console_? If so, the answer is it's likely your responsibility. If you can't control something, it's likely AWS' responsibility.

Check out AWS' page for more information about the [shared responsibility model here](https://aws.amazon.com/compliance/shared-responsibility-model/).

### Compute, Storage, Databases, & Networking

:bulb:

- **Compute** = The way we process information
  - EC2, Lambda, Elastic Beanstalk
- **Storage** = Giant disk in the cloud, a safe place to leave information
  - S3, Elastic Block Store (EBS), Elastic File Service (EFS), FSx, Storage Gateway
- **Databases** = A spreadsheet, a reliable way to store and retrieve information
  - RDS (Amazon Aurora, MySQL), DyanmoDB (noSQL), Redshift (data warehousing)
- **Networking** = Allows for communication between these resources
  - VPCs, Direct Connect, Route 53, API Gateway, AWS Global Accelerator

### What is the Well Architected Framework?

Read here to review the **[5 Pillars of the Well-Architected Framework](https://aws.amazon.com/blogs/apn/the-5-pillars-of-the-aws-well-architected-framework/)** :bulb: which are:

1) Operational Excellence
   1) Focuses on running and monitoring systems to deliver business value, and continually improving processes and procedures
2) Security
   1) Focuses on protecting information and systems
3) Reliability
   1) Focuses on ensuring a workload performs its intended function correctly and consistently when it's expected to
4) Performance Efficiency
   1) Focuses on using IT and computing resources efficiently
5) Cost Optimization
   1) Focuses on avoiding unnecessary costs

## Identity and Access Management (IAM)

### Securing the Root Account

The root account for AWS is the email address used to sign up for AWS, it has full admin access and is extremely important to keep secure.

One of the best things to do immediately is to turn on multi-factor authentication for the root account using a resource like Google's MFA service. This can be done under IAM.

There are 4 steps to securing a root account:

1) Enable MFA
2) Create an admin group for admins and assign appropriate permissions to it
3) Create user accounts for admins
4) Add those users to the admin group

### Controlling Users' Actions with IAM

Permissions are assigned to users using policy documents. Policy documents are made using JSON files. :bulb: **You will need to know how to read JSON files for the exam!** Policy documents can be assigned to groups, users, and roles though it is typically frowned upon to assign to and manage policies at the user level. It is better to assign users to groups and have users inherit permissions from those groups.

> :bulb: Note that IAM is a global service and not a regional one!

Sample JSON notation for a policy can look something like this:

```JSON
{
   "Version": "2017-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Action": "*",
         "Resource": "*"
      }
   ]
}
```

Which is essentially an admin policy allowing all actions for all resources.

