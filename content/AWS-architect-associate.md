---
title: "AWS Architect Associate"
date: 2018-01-16
draft: false
menu:
  main: {}
---

# AWS Associate Architect Exam

## General concepts

*Read the FAQs for all services!!*

### Region

- Geographical location in the world that contains availability zones (AZs)
- Services available across regions can vary

### Availability zone

- Physical data center
- Provide resilience and fault tolerance for applications


## EC2

### Pricing models

| Type | Use case |
| ---- | -------- |
| On demand | Unpredictable loads |
| Spot | Lowest cost, cpu intensive calculations, stateless, dev environments |
| Reserved | Known long term capacity |
| Dedicated host | Require dedicated hardware (regulatory usually) |


*Spot costs*

 - _User terminated_ - pay for the hour
 - _AWS terminated_ - get the hour for free

### Instance types

| Family | Speciality        | Use case       |
| ------ | ----------------- | -------------- |
| F1    | Field programmable gate array | Genomics research, financial analysis, big data  |
| I3    | High speed storage | NoSQL DBs, Data warehousing |
| H1    | High disk throughput | MapReduce workloads, distributed file systems |
| D2    | Dense storage     | File servers, data warehousing, Hadoop |
| R5    | Memory optimised  | Memory intensive apps/DB |
| M5    | General purpose   | Application servers |
| C5    | Compute optimised | CPU intensive apps/DB |
| G3    | Graphics intensive | Video encoding/3D application streaming |
| I2    | High speed storage | NoSQL DBs, Data warehousing |
| F1    | Field programmable gate array | Hardware acceleration for code |
| T3    | Low cost general purpose | Web servers, small DBs |
| P3    | Graphics/general purpose GPU | Machine learning, Bitcoin mining |
| X1    | Memory optimised  | Apache spark, SAP HANA |
| Z1D   | High compute and memory | Electronic design automation and DBs with per core licensing |
| A1    | ARM based workloads | Scale out workloads such as webservers |
| U-6tb1 | Bare metal       | Eliminate virtualisation overhead |

### Termination protection

- Turned off by default
- Prevents instance from being terminated through:
  - CLI
  - Console
  - API
- Can still issue a shutdown command from within the host OS

### Placement groups

- Cluster placement groups
  - Used to obtain lowest network latency possible between instances
  - Recommended to provision all instances in group at the same time
  - Recommended that all instances are of same type
  - All instance must be in same AZ


- Spread placement groups
  - Use for placing instances on separate hardware
  - Can be spread across AZs
  - Max 7 instances per AZ per placement group  

- No extra charge for using placement groups

### EBS Storage

*Type summary*

| Type | Name | Use |
| ---- | ---- | --- |
| SSD  | General purpose - GP2 | Up to 10,000 IOPS |
| SSD  | Provisioned IOPS- IO1 | Over 10,0000 IOPS |
| HDD  | Throughput optimised - ST1 | frequently accessed workloads |
| HDD  | Cold - ST1 | less frequently accessed data |
| HDD  | Magnetic - standard | cheap, infrequently accessed storage |

<br>

*Usage guidelines*

- _SSD_ - transactional, IOPS intensive tasks (boot volumes, some databases)
- _HDD_ - throughput intensive workloads (big data, large I/O sizes)

*Other notes*

- EBS volumes can only be mounted to _one_ EC2 instance.
- Can be encrypted

### Volumes

- EBS storage attached to an EC2
- Volumes can be encrypted
- Volume restored from encrypted snapshot will be encrypted

### Root volumes

- Deleted when terminated by default
- Cannot be encrypted by default (must use a 3rd party tool)

### Snapshots

- Point in time capture of a volume
- Stored on S3
- Incremental (only blocks changed since last snapshot are uploaded)
  - First snapshot will be slower

- When taking snapshot of root device volume:
  - Instance should be stopped
  - Will still work if instance is running

_Encryption_

- Snapshot of encrypted volume will be encrypted automatically
- Encrypted snapshots cannot be:
  - Shared with other accounts
  - Made public

_Snaphotting RAID arrays_
- Extra care required due to interdependencies in the array
- Take an application consistent snapshot
  - Stop the application writing to disk
  - Flush the cache


- Options:
  - Freeze the file system
  - Unmount the RAID array
  - Shutdown the EC2 instance

### AMI

- Regional
  - Can only launch an image from the regions it's stored in
  - Can copy between regions using command line, console or API

- Cannot delete the EBS snapshot that an AMI is based on.

### Instance backed storage

 - Sometimes called ephemeral storage
 - Instances with instance backed storage cannot be stopped
 - If the host fails, the data will be lost
 - Can still be rebooted

### Instance backed vs EBS summary

| Type | On shutdown | On restart | On host failure | On terminate |
| ---- | ----------- | ---------- | --------------- | ------------ |
| EBS  | Persisted   | Persisted  | Persisted       | - Default deleted <br>- Optional persisted |
| Instance backed | Deleted | Persisted | Deleted         | Deleted         |

### Cloudwatch

- Used for performance monitoring
  - Don't confuse with CloudTrail (auditing)
- Standard monitoring 5 min intervals
- Optional 1 min intervals
  - Costs extra


- Things to do with Cloudwatch
  - Fancy dashboards to see whats going on
  - Alarms when thresholds are hit
  - Events triggered on state changes
  - Logging - aggregate, monitor and store

### Roles
 - Use instead of IAM users for an EC2
 - Removes the need for storing secret access keys
 - Easier to secure & manage
 - Can be assigned to an EC2 _during_ and _after_ provisioning
 - Policies can be added to the role at any time
 - Roles are universal - can use across all regions

### Instance metadata

- Used to get information about an instance
  - Eg public IP

- curl http://169.254.169.254/latest/meta-data

### Security Groups

- By default all traffic to instances is blocked
- Instances can be assigned several security groups

## Load Balancers

- Used for spreading traffic across several EC2 instances
- Instances must be in the same region, but can be in different AZs

- Performs heath checks on the instances
- Will only send traffic to healthy instances

- Never have a public IPv4 address.
- Must use a DNS name.

Types of load balancers

| Type | Usage |
| ---- | ----- |
| Application Load Balancer | Layer 7 (application) - based on the content of packets |
| Network Load Balancer | Layer 4 (network) - content of message not inspected |
| Classic Load Balancer | Use for EC2 classic. Prefer other load balancer types |


## Auto Scaling

- Used to manage a group of EC2 instances
- Can scale the number of instances up or down depending on thresholds set
- Will start new instances when any in the group are terminated
- Typically used with a load balancer
- Evenly distributes instances across AZs configured in the load balancer

- New instances are started using a _launch template_
- Launch template contains EC2 configuration details

- Can be used for scaling up and down for meeting spiky loads

## Lambda

- AWS takes care of all the infrastructure to run code

- Event driven - responding to events raised by other AWS services
  - Eg file uploaded to S3 bucket
- Compute service
  - Responding to http requests behind an API Gateway
  - Called through AWS API

- Pay for time spent executing and amount of memory consumed
- Max execution time of 5 min

### Triggers
Services that can trigger Lambda:
- API gateway
- AWS IoT
- Alexa skills kit
- Alexa smart home
- CloudFront
- CloudWatch events
- CloudWatch logs
- Code commit
- Cognito
- Dynamo DB
- Kinesis
- S3
- SNS

## IAM

### Users

### Roles

### Policies

## S3

Read the S3 FAQ

- Object based storage
- File size
  - 0 bytes to 5 TB
- Files stored in buckets
- Buckets must have globally unique name
- Successfully uploaded files return a http 200 status code
- Buckets can be protected with MFA delete

### Object fundamentals

- Key - name of object
- Value - object data
- Version ID
- Metadata
- Subresoruces
  - access control list (ACL)
  - Torrent

### Consistency model

- read after write for new object PUTs
- eventual consistency for DELETE and overwrite PUTs

### Storage types

- S3 Standard
  - 99.99% availability
  - 11 9s durability
- S3 - IA
  - charged a retrieval fee
- S3 One Zone IA
  - charged retrieval fee
  - only stored in one zone
- S3 intelligent tiering
  - moves files around tiers based on usage
- Glacier
  - archiving
  - access from minutes to hours
- Glacier deep archive
  - cheapest storage
  - 12 hour access sla

### Lifecycle

- Transition to different storage types after a set amount of time
- Can also be used to expire and delete objects

### Security

### Encryption

- In transit
  - SSL/TLS

- At rest server side
  - S3 managed keys - SSE-S3
  - AWS key management service - SSE-KMS
  - Customer managed keys - SSE-C

### Version control

- Stores all versions of an object
- Deleting places a delete marker as the most recent version
- Good backup tool
- Cannot be disabled once enabled, only suspended
- Provides MFA delete capability

### Cross region replication

- Requires versioning to be enabled
- Regions must be unique
- Doesn't replicate objects already in bucket
- Delete markers don't get replicated
- Deleting individual versions not replicated

### Transfer acceleration

- Used for speeding up file uploads
- Utilises the edge network
- User uploads file to edge which is then transferred over the amazon network to S3

## Glacier

## EFS

- Supports Network File System version 4 (NFSv4) protocol
- Pay for storage Used
- No pre-provisioning
- Scales to petabytes
- Supports thousands of concurrent NFS connections
- Stores data across multiple availability zones
- Read after write consistency

## Snowball

- Used for moving large amounts of data into or out of AWS
- import or export data to S3


## Storage gateway

- connects on prem device to AWS
- replicates data from data center to AWS
- Virtual or physical appliance

- Three different types:
  - File gateway (NFS $ SMB)
  - Volume gateway (iSCSI)
    - Stored volumes
    - Cached volumes
  - Tape gateway

## CloudFront

- Objects are cached according to their TTL
- Can expire objects to force refresh but get charged for it

### Edge location

- Location where content is cached
- Separate to a Region or Availability zone
- Can be written to as well as read from

### Origin

- The source of the files that the CDN will distribute

### Distribution

- The name given to the CDN
- Consists of a collection of edge locations

### Distribution types

- Web
- RTMP - media streaming

## API gateway

### CORS

## DNS

### DNS basics
#### Top level domains
- Managed by Internet Assigned Numbers Authority (IANA)
- Top level domains are .com .net etc

#### Domain registrars
- Manage purchasing of domains
- Registers domains with InterNIC which enforces uniqueness of domains across the internet.
- Domain information becomes available in a central database called WhoIS database.

#### SOA records
- Contains metadata about the domain

#### Name server records (NS Records)
- Used by top level domain servers to locate which Content DNS servers contain the authoritative DNS records.
  - in other words, where to go to find out the actual domain name -> IP address mapping (A record).

#### Address record (A Record)
- Translates the domain name to an ip address
  - eg http://aws-exam-notes.com to 192.168.0.0

#### Canonical name (CNAME)
- Used to resolve one domain name to another
  - eg mobile.somesite.com -> m.somesite.com
- Can't be used for naked domains (apex record) eg somesite.com

_cost_

- Charged for CNAME look ups.

#### Alias records
- AWS specific type of record.
- Very similar to CNAME
  - However alias record can map from a naked domain
- Maps resource record sets in Route 53 hosted zone to AWS resources:
  - S3 buckets, load balancers, cloud front distributions.
  - Mainly used for load balancers

_cost_

- No charge for alias record look up
  - For this reason, alway prefer over CNAME when possible

#### TTL
- Time to live for a record.
- Resolving servers or local pc will cache record for this long.
- Lower TTL means faster propagation of name changes.
- Default set to around two days

_Planning DNS migrations_

- Lower TTL to 300 sec (5 min) at least 2 days before.
  - Allows for TTL change to propagate.
- Change name records.
  - Changes will now propagate quickly.
  - Minimise possible downtime with change.
