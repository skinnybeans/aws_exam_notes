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

- Partitioned placement groups
  - Different groups (partitions) of EC2 instances
  - Each partition runs on a different set of hardware

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
- EBS volume will be in the same AZ as the EC2 instance
- By default the root volume is deleted when the instance is terminated
- Additional volumes are not deleted by default on termination
- The root volume can be encrypted
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

#### Encryption

- Root volumes can now be encrypted
- Snapshot of encrypted volume will be encrypted automatically
- Encrypted snapshots cannot be:
  - Shared with other accounts
  - Made public

#### Making encrypted AMI from unencrypted root volume

- Make snapshot of image
- Copy snapshot and make it encrypted
- Create AMI from encrypted snapshot

#### Snaphotting RAID arrays

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

- Use hardware assisted virtualisation for  best EC2 instance type support when creating AMI from snapshot

### Instance backed storage

- Sometimes called ephemeral storage
- Instances with instance backed storage cannot be stopped
- If the host fails, the data will be lost
- Can still be rebooted

### Instance backed vs EBS summary

- EBS backed or instance store.
- Instance store is ephemeral storage closely coupled to the compute
  - cannot  be stopped.
- EBS backed storage sits on a disk array somewhere and lifecycle is decoupled from the compute

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

- All inbound traffic blocked by default
- All outbound traffic allowed
- Instances can be assigned several security groups
- Security groups are stateful
  - Inbound requests can be responded to without an explicit rule
  - Outbound requests can be responded to without an explicit rule

- Can't block specific ports or IP addresses

## Load Balancers

Types of load balancers

| Type | Usage |
| ---- | ----- |
| Application Load Balancer | Layer 7 (application) - based on the content of packets |
| Network Load Balancer | Layer 4 (network) - content of message not inspected |
| Classic Load Balancer | Use for EC2 classic. Prefer other load balancer types |

### Application load balancers

- Smart routing
- Lots of configuration options

- Set up a target group
- Add instances to target group
- Create ALB
- Add target group


### Network load balancers

- Have a fix IP address
- High performance
- Run at the connection level
- Expensive

### Classic load balancers

- Can use some layer 7 specific features
  - X Forward For
  - Sticky sessions
- Also configurable for layer 4
- Cheapest

---

- X Forwarded For
  - Allowed instances behind the load balancer to see the client IP in the request rather than the load balancer IP
  - HTTP header

- 504 error
  - Means the application behind the load balancer has stopped responding

- Instances behind a load balancer are always reported as:
  - In Service - passing health checks and available to take traffic
  - Out Of Service - can't take traffic

- Sticky sessions
  - Ensure that all requests from a client are forwarded to the  same instance
  - Classic load balancers will route to the same instance
  - Application load balancers will route to the same target group
  - If traffic is not evenly distributed to instances behind load balancer try disabling sticky sessions

- Cross zone load balancing
  - Load balancer can only send traffic to instances in another AZ if cross zone load balancing enabled
  - This applies even if they registered as targets for the load balancer

- Path patterns
  - Send traffic to target groups based on the URL
  - Known as path based routing

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


- Object based storage
- File size
  - 0 bytes to 5 TB
- Files stored in buckets
- Buckets must have globally unique name
- Successfully uploaded files return a http 200 status code
- Buckets can be protected with MFA delete
- Supports IPv6 connections
- Default limit of 100 per account

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
  - 99.9% availability
  - 11 9s durability
- S3 One Zone IA
  - charged retrieval fee
  - only stored in one zone
  - 99.5% availability
  - 11 9s durability, however data loss if AZ goes down.
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

### Website hosting

- Can set up redirects at the bucket level

## Glacier

## EFS

- Supports Network File System version 4 (NFSv4) protocol
- Pay for storage Used
  - Grows automatically as necessary
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

## Route 53

### Top level domains

- Managed by Internet Assigned Numbers Authority (IANA)
- Top level domains are .com .net etc

### Domain registrars

- Manage purchasing of domains
- Registers domains with InterNIC which enforces uniqueness of domains across the internet.
- Domain information becomes available in a central database called WhoIS database.

### SOA records

- Contains metadata about the domain

### Name server records (NS Records)

- Used by top level domain servers to locate which Content DNS servers contain the authoritative DNS records.
  - in other words, where to go to find out the actual domain name -> IP address mapping (A record).

### Address record (A Record)

- Translates the domain name to an ip address
  - eg http://aws-exam-notes.com to 192.168.0.0

### Canonical name (CNAME)

- Used to resolve one domain name to another
  - eg mobile.somesite.com -> m.somesite.com
- Can't be used for naked domains (apex record) eg somesite.com

#### cost

- Charged for CNAME look ups.

### Alias records

- AWS specific type of record.
- Very similar to CNAME
  - However alias record can map from a naked domain
- Maps resource record sets in Route 53 hosted zone to AWS resources:
  - S3 buckets, load balancers, cloud front distributions.
  - Mainly used for load balancers

#### cost

- No charge for alias record look up
  - For this reason, alway prefer over CNAME when possible

### TTL

- Time to live for a record.
- Resolving servers or local pc will cache record for this long.
- Lower TTL means faster propagation of name changes.
- Default set to around two days

#### Planning DNS migrations

- Lower TTL to 300 sec (5 min) at least 2 days before.
  - Allows for TTL change to propagate.
- Change name records.
  - Changes will now propagate quickly.
  - Minimise possible downtime with change.

### Routing types

| Name | Description |
| ---  | ---         |
| Simple | Picks one of the associated IPs at random |
| Weighted | Set a weight to send traffic to different IP addresses |
| Latency based | Routes traffic to the destination that has the lowest latency to request source |
| Failover | Used for an active/passive fail over set up |
| Geolocation | Traffic gets sent to servers based on geolocation of users |
| Geoproximity | Uses route 53 traffic flow for doing fancy stuff |
| Multivalue answer | Like simple routing but can also support health checks |

## RDS

- Two key features
  - Multi AZ for disaster recovery
  - Read replicas for performance

- Runs on virtual machines
- Have no access to them (cannot SSH in)
- Patching is responsibility of Amazon
- RDS is not serverless

### Multi AZ

- Copy of the data stored in a different AZ
- For disaster recovery purposes only, not performance
- AWS will automatically cut over the DNS to point to the other database
- Can force a failover by rebooting the instance and selecting failover
- Replication is performed synchronously
- Available for:
  - MySQL
  - Oracle
  - SQL Server
  - PostgreSQL
  - MariaDB

### Read replicas

- Read only copy of DB master instance
- Useful for read heavy workloads
- For scaling, not DR
- Must have automatic backups turned on
- Read replica can exist in a different region
- Can be promoted to masters but breaks replication
- Available for:
  - MySQL
  - Oracle
  - PostgreSQL
  - MariaDB
  - Aurora

### Backups

- Automated backups
  - Done during maintenance window
  - Backups are stored in the same AZ

- Snapshots
  - User initiated

## DynamoDB

- On SSD storage
- Data replicated across three distinct geographical locations
- Two different consistency models
- Eventually consistent reads
  - Data can be read after 1 second of being written
  - Best read performance
  - Default setting
- Strongly consistent reads
  - Data can be read in less than one second after being written

- 400KB limit for key and value

## Redshift

- Database for data warehousing
- Online analytic processing (OLAP)
- Two options for configruarion
  - single node (160GB)
  - multi node
    - leader node receives connections and manages queries
    - compute nodes store data and process queries
    - up to 128 compute nodes

- backups
  - enables by default
  - default retention of 1 day
  - configurable up to 35 days

- data redundancy
  - maintains 3 copies of the data
    - original file
    - replica on compute nodes
    - backup on s3

- priced on compute node hours

- cannot have multi AZ turned on
- can restore snapshots into another AZ

## Aurora

- MySQL compatible database developed by AWS

### Storage scaling

- Starts at 10GB of storage
  - scaleable in 10GB increments
  - max 64TB

### Compute scaling

- Up to 32 vCpus
- 244GB Memory

### Data redundancy

- Stores 2 copies in each AZ
- Minimum of 3 AZ used

### Replicas

- Aurora replicas (up to 15)
- MySQL replicas (up to 5)

### Backups

- Automatically backed up
- Can take snapshots and share to other AWS accounts

## Elasticache

- web service for in memory caching
- used in front of web sites

- engines
  - memcached
    - simple and easy to use
  - redis
    - more features such as advanced data types and backups/restore
    - multiAZ

## VPC

- Virtual data center in the cloud
- Main components
  - Internet gateways (IGW)
  - Virtual private gateways (VPG)
  - Route tables
  - Security groups
  - Network access control lists (NACLs)
  - Subnets
- A subnet can only span one availability zone
- Security groups are stateful
- NACLs are stateless

- 5 IP addresses for every VPC are reserved

### Creating a new VPC

- Comes with
  - Route table
  - NACL
  - Security group
- Does not come with
  - Subnets
  - Internet gateway


### NAT instances/gateways

- Used for providing internet access to private subnets
- Instances can communicate out, but external traffic cannot initiate connections in
- Route table entries required to route connections from subnet through NAT

#### NAT instances

- Runs on an EC2
- Use Amazon AMI
- Launch inside a public subnet
- Must disable source and destination traffic checks
- Must be in a security group
- Not very resilient

#### NAT gateway

- Choose subnet to create in
- Require an elastic IP to be attached
- Managed by AWS
  - Don't need to patch them

### NACLS

- Rules are evaluated in order until a matching rule is found
- Need outbound and inbound rules
- Each subnet can only be associated with 1 nacl
- A nacl can have many subnets associated with them
- Default nacl assigned to all newly created subnets (through console)
- NACLS are evaluated before security groups

- The default NACL allows all traffic
- Any custom NACLs do not allow any traffic

- Each subnet must be associated with 1 NACL

### VPC flow logs

- Network traffic logs for VPCs
- Available at 3 levels
  - VPC
  - Subnet
  - ENI
- Cannot enable flow logs for a peered VPC unless it is in the same account
- Cannot tag a flow log
- Cannot update its config after creating it

### VPC endpoints

- Connect to AWS services without going over the internet
- Do not need public IPs for EC2
- Two types of endpoint:
  - Interface endpoint
  - Gateway endpoint

- Interface endpoint
  - An ENI with a private IP that can be used to access a range of AWS services
  - Attach the ENI to an EC2 to get access
- Gateway endpoint
  - Looks like a NAT gateway
  - Used for S3 and DynamoDB

## Applications

### SQS

- Distributed  message queue
- Used to decouple components of a distributed system
- Stores up to 256KB of text data
  - Can go up to 2GB if messages are stored on S3
- Retention
  - Configurable from 1 minute to 14 days
  - Default 4 days
- Visibility time out
  - Time from when a message is taken from the queue to when it must be deleted
  - Otherwise it will become visible in the queue again for processing
  - Maximum of 12 hours
- Pull based
- Can long poll a queue
  - If the queue is empty, request will not return  until it times out or a message arrives
- Two types of queue
  - Standard
    - Can take millions of transactions per second
    - Ensures each message will be delivered at least once
    - May deliver messages in a different order than they are sent
  - FIFO
    - Always retains ordering
    - No duplicate messages
    - Limited to 300 transactions per second

### Simple workflow service

- Used to coordinate  work across distributed application components
- Mainly replaced with step functions now
- Can incorporate manual steps with automated steps
- Workflow executions can span up to a year
- Task based API instead of SQS message based API
- Tasks are only assigned once
- Keeps track of all tasks and events in an application
- SWF actors
  - Workflow starters
    - Initiate workflows
  - Deciders
    - Control the task flow
  - Activity workers
    - Do actual work

### SNS

- Simple notification service
- Used for pushing events to subscribers
- Can push messages using
  - SMS
  - Email
  - HTTP
  - SQS
- Organised into topics
- Topics can publish messages in different formats

### Elastic Transcoder

- Media transcoder in the cloud
- Convert media files into different formats

### API Gateway

- Provides end points for accessing other AWS resources such as:
  - Lambda
  - DynamoDB

- Provides caching to improve response times
- Automatically scales
- Can throttle requests
- Log requests to cloudwatch
- Remember to enable CORS

### Kinesis

- Place to send streaming data to
- Three different types:
  - Kinesis streams
    - Producers generate data
    - Send to shards
    - Consumers read data from shards
    - Stored for 24h by default
  - Kinesis firehose
    - No persistent storage
    - Processed in realtime then output somewhere
  - Kinesis analytics
    - Can look at data in stream or firehose

### Cognito

- Brokers between identity providers like facebook and google
- Provides temporary credentials based on an IAM role
- No need to store credentials locally

- User pools
  - Manage sign up and sign in for users
  - Can sign in directly to user pool or use other identity provider
  - User ends up being issued a JWT
  - Authentication

- Identity pool
  - Authorisation

- User pool to get JWT issues
- Present JWT to identity pool to get access to resources