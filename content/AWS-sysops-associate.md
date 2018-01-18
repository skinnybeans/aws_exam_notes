---
title: "AWS SysOps Associate"
date: 2018-01-17
draft: false
menu:
  main: {}
---

# Overview

## Content

  * <a href="#monitoring">Monitoring</a>
  * <a href="#high-availability">High Availability</a>
  * <a href="#deployment-provisioning">Deployment & Provisioning</a>
  * <a href="#data-management">Data Management</a>
  * <a href="#opsworks">OpsWorks</a>
  * <a href="#security">Security</a>
  * <a href="#networking">Networking</a>
  * <a href="#useful-resources">Useful Resources</a>

## Exam Overview

### Exam Blueprint

  * Aim to get around 70% on exam.
  * Takes 80 minutes.

### Exam Objectives

  * Monitoring & Metrics - 15%
  * High Availability - 15%
  * Analysis - 15%
  * Deployment & Provisioning - 15%
  * Data Management - 12%
  * Security - 15%
  * Networking - 13%
  * Networking is broken up into Route53 and VPCs


# Monitoring

## CloudWatch

### Monitoring EC2

  * Host level metrics consist of (remember these!):

      * CPU
      * Network
      * Disk
      * Status Checks

  * Exam Tips:
      - RAM Utilization is a custom metric!
      - By default, EC2 monitoring is 5 minute intervals, unless you enable detailed monitoriing which will then make it 1 minute intervals
      - 1 Minute intervals is the minimum for custom metrics (though this has changed recently I'm pretty sure?)

  * Metric Granularity
      - Many default metrics for many default services are 1 minute
      - Some services can have 3 or 5 minutes however

#### EC2 Status Checks

  * System Status Checks ( = physical )
    - Checks the underlying physical host
    - Issues that can occur:
        - Loss of network connectivity
        - Loss of system power
        - Software issues on the physical host
        - Hardware issues on the physical host
    - Best way to resolve is to stop and then start the VM again! (NOT JUST RESTART)
        - This starts the VM on another physical host

  * Instance Status Checks ( = virtual )
    - Checks the VM itself
    - Issues that can occur:
        - Failed system status checks (see above)
        - Misconfigured networking or startup configuration
        - Exhausted memory
        - Corrupted file system
        - Incompatible kernel
    - Two ways to solve issues above:
        - Turn it off and on again
        - Make modification in your VM OS

  * Exam Tips:
    - Know the difference system and instance status checks
    - Know the best way to resolve each

### Monitoring EBS

  * Types of Volumes:

    - General Purpose (SSD) - gp2
        - Recommended for most workloads (definitely for system boot volumes)

    - Provisioned IOPS (SSD) - io1
        - Critical business apps that require sustained IOPS (10k+ IOPS or 160 MiB/s)
        - Really for IOPS intensive or database servers in production

    - Throughput Optimized (HDD) - st1
        - Best for streaming workloads (data warehouses, log processing, big data)
        - Can no longer be boot volumes

    - Cold (HDD) - sc1
        - Throughput-oriented for large volumes of data seldom accessed
        - Best for data archiving scenarios

  * Burst performance: Say we have a 100 GiB volume. We get 3 IOPS per GB baseline, so have 3 x 100 = 300 IOPS baseline performance. We can burst the performance of this volume using I/O credits. Your burst balance can be calculated by subtracting the baseline IOPS value from 3000. So in this case, our default I/O credits would be 2700.

  * You want to make sure that the size of your volume is in line with your IOPS requirements. If the IOPS requirements exceeds 10,000 IOPS, or you're consistently dipping into your I/O credits then you need to move to an io1 volume type.

  * Pre-warming new EBS volumes is no longer necessary. The only time you need to pre-warm is when you restore an EBS snapshot, but you can avoid this in production environments by ensuring your EC2 instances have read all the blocks on the EBS volume (this is called initialization). It's basically done by running the following command: `sudo dd if=/dev/xvdf of=/dev/null bs=1M` (if=inputfile of=outputfile bs=blocksize-for-read-op).

  * CloudWatch Metrics for EBS Volumes:
    - VolumeReadBytes/VolumeWriteBytes size of IO operation (can be summed to trend)
    - VolumeReadOps/VolumeWriteOps total IO operations
    - VolumeTotalReadTime/VolumeTotalWriteTime total number of secs by all ops
    - VolumeIdleTime total number of secs where no read or write ops occurred
    - VolumeQueueLength is the total op requests waiting to be completed (disk is busy if high)
    - VolumeThroughputPercentage used only with io1, IOPS delivered as proportion of total provisioned IOPS (i.e. you may only use 10% of total provisioned IOPS for a given period)
    - VolumeConsumedReadWriteOps used only with io1, a count of 256K write/read ops for a period of time (e.g. you could have 1024K I/O would be 4 consumed IOPS)

  * Volume Status Checks (REMEMBER THESE!!!)
    - ok: normal
    - warning: degraded or severely degraded
    - impaired: stalled or not available
    - insufficient-data: insufficient data on the EBS

  * Modifying the EBS Volumes
    - If you EBS volume is attached to a current EC2 instance you can change the volume type and size WITHOUT detaching the volume first (on-the-fly)
    - You will need to extend the volume's file system to take advantage of the increased storage capacity

  * Exam Tips:
    - Best to remember the API names of the volume types for exam (gp2 and io1 will defintely come up)
    - Remember that gp2 instances have a base of 3 IOPS per/GiB of volume size
    - Maximum volume size is 16,384 GiB
    - Maximum IOPS size is 10,000 IOPS total before having to move to provisioned IOPS
    - You will get scenario questions around low performance applications that may be improved by moving to provisioned IOPS
    - No need to figure out how to calculate EBS I/O burst duration
    - Remember VolumeQueueLength!
    - Remember the different Volume Status Checks! You will get asked what volume statuses will be assigned to which scenarios (i.e. know that degraded or severely degraded = warning, and that stalled or not available = impaired)
    - EBS is the bread and butter questions. Know your instance status checks as well.

### Monitoring RDS

  * There are 2 types of monitoring available for RDS. CloudWatch metrics and RDS events.

  * In RDS you can create an "Event Subscription" where you can trigger a push to any sort subscriber (i.e. SNS topic, email, etc.) for certain events (e.g. instance reboots).

  * Exam Tips:
    - Have a general idea of what each metric does. You do not need to know the name of each metric, but pay attention to the following:
        - DatabaseConnections: Number of connections to the databased maintained
        - DiskQueueDepth: Similar to EBS Queue Length, indication of disk performance
        - FreeStorageSpace: Amount of storage space left
        - ReplicaLag (Seconds): lag in secs between RDS instance and your read replica (scenario could be that users are noticing that their data is not up to date, you would check this metric to ensure that there's not too much lag between the master and the replica)
        - ReadIOPS: IOPS expended for read ops
        - WriteIOPS: IOPS expended for write ops
        - ReadLatency: the latency in secs for read ops
        - WriteLatency: the latency in secs for write ops

### Monitoring ELB

  * ELB is monitored every 60 seconds provided there is traffic flowing through the ELB
  * Metrics:
    - HealthyHostCount: number of healthy hosts inside the ELB pool
    - UnhealthyHostCount: numer of unhealthy hosts inside the ELB pool
    - RequestCount: total number of requests
    - Latency: secs elapsed after the request leaves the load balancer
    - `HTTPCode_ELB_4XX`: The count of the number of 4XX client error codes generated by the ELB (indicates that a request is malformed or incomplete)
    - `HTTPCode_ELB_5XX`: The count of the number of 5XX server error codes generated by the ELB (reported if there are no back-end instances that are healthy or registered to the ELB, or if the request rate exceeds the capacity of the instances or the ELB itself)
    - `HTTPCode_Backend_2XX/3XX/4XX/5XX`: the count of HTTP response codes generated by backend instances (does not include response codes generated by the load balancer).
    - BackendConnectionErrors: The count of the number of connections that were not successfully established between the load balancer and the registered instances (ELBs will retry when there are connection errors, this count can exceed request rate).
    - SurgeQueueLength: count of the number of total requests that are pending submission to the registered instance.
    - SpilloverCount: number of requests that are rejected because the queue is full

  * Exam Tips:
    - Have a general idea of what each metric does. You do not need to know the name of each metric, but pay attention to the `SurgeQueueLength` & `SpilloverCount` metrics.

### Monitoring Elasticache

  * Elasticache consistents of Redis and Mecached
  * Four important metrics:
    - CPU Util
        - Memcached is multi-threaded, so it can handle loads of up to 90%, exceeding this you need to add more nodes to the cluster
        - Redis is not multi-threaded, so to determine the point at which you need to scale the cluster, you would take 90 and divide by the number of CPU cores (e.g. cache.m1.xlarge node type has four cores, in this case the threshold for CPU Util would be 90 / 4 = 22.5%)

    - Swap Usage
        - Put simply, swap usage is simply the amount of the Swap file that is used. The Swap File (or paging file) is the amount of disk storage space reserved on disk if your computer runs out of RAM. Typically the size of the swap file = the same of the RAM. So if you have 4GB of RAM, you'll have a 4GB swap file.
        - With Memcached, swap usage should be 0 most the time and should not exceed 50MB. If it exceeds 50MB, you should increase the `memcached_connections_overhead` parameter (defines the amount of memory to reserve for memcached connections)

    - Evictions
        - An eviction occurs when a new item enters a cache that's full and an item needs to be "evicted" from the cache to make room for the new item
        - In memcached, there is no recommended setting for evictions, but you can scale up or out when evictions are increasing
        - In Redis, you can only scale out read replicas when evictions are increasing, you can not change the instance type

    - Concurrent Connections
        - There's no recommended settings for this (just choose based on your app)
        - If there's a spike here, you're either experiencing high traffic or your app isn't releasing connections as should be (set an alarm on concurrent connections)

  * Exam Tips:
    - You won't need to calculate Redis CPU threshold, just good to know
    - Memcached should not have swap usage higher than 50MB, if this is the cache you need to increase the `memcached_connections_overhead` parameter.
    - Evictions can come up in the exam, particularly around the differing approachs to handling evictions between Memcached and Redis (scale up and out, and only scale out respectively)
    - Typically two exam questions on Elasticache (concurrent connections and evictions)


### Centralized Monitoring

  * Most enterprising have monitoriing solutions in place that provide detailed monitoring for specific applications

  * Exam may ask you how to globally monitoring your infrastructure - the answer to this would be to set up a centralized monitoring system

  * Different monitoring protocols require differing security group configurations (most basic monitoring is going to use ICMP to enable ping, but this could be for specific IP addresses i.e. SQL=1433 or Mysql=3306)

  * Exam may ask you to troubleshoot a connectivity problem between your centralized monitoring system and one of your application serverrs (allow an inbound rule from your instance security group for ALL ICMP in the monitoring server security; also use the inverse ingress rule in the instance security as a ping is a two-way interaction).

  * This scenario will come up on the example. The solution is to basically just use security groups.

### Consolidated Billing and AWS Organisations

  * AWS provides an account management service which conslidates billing for a number of AWS accounts ("Consolidated Billing")
    - You have a "paying account"
    - Then you have "linked accounts"
    - You receive a monthly bill which breaks down the bill across linked accounts
    - Paying account can not access resources in linked accounts by default
    - Linked accounts are isolated
    - You get the benefit of volume pricing across all AWS accounts (S3 pricing for example)
    - Reserved instances can be spread across linked accounts to optimise EC2 spend
    - Exam will ask you how to save money (consolidated billing is one option)

  * AWS Consolidated Billing Best Practices:
    - Always enable MFA on the root account
    - Use strong and complex passwords on root account
    - Paying account should be used for billing only and no resources should be created

  * Linked Accounts max is 20 (visit service limit increase site for increase)

  * Billing alerts is enabled on the paying account, but you can still set up alerts on linked accounts

  * CloudTrail is per AWS Account is enabled per account per region basis (logs can be consolidated using an S3 bucket)

  * Exam Tips:
    - Consolidated billing allows you to get volume discounts on all your accounts
    - Unused RIs for EC2 are applied across the group
    - CloudTrail is on a per account, per region basis and can be conslidated by aggregating logs into a global S3 bucket

### Billing and Alerts

  * You can create a billing alarm to alert you when your AWS charges exceed a value

### EC2 Cost Optimization

  * EC2 Instance Types:

    - On Demand

    - Reserved

    - Spot Price
        - Spot instances allow you to name your own price for Amazon EC2 computing capacity
        - You simply bid on spare EC2 instances and these will run automatically whenever your bid exceeds the current Spot Price.
        - Spot Prices change in line with supply and demand
        - When you are outbid, your instances are terminated

    - On Demand
        - Allows you to pay by the hour with no long-term commitments or upfront payments
        - During periods of very high demand there may not be enough capacity to allocate new instances

    - Reserved
        - Reserved instances provide you with a significant discount (up to 75%) compared to on-demand instance pricing. You are assured that your RI will always be available for the OS and Availability Zone in which you purchased it.
        - Perfect for applications that have steady state needs

  * Heavy Util vs Medium Util
    - Your cost savings are generally half or 25% as you move down instance sizes (i.e. m1.medium to m1.small is about a 25% decreasei)

      - *TYPICAL EXAM QUESTION:* You run an application which is hosted on EC2 instances in an autoscaling group spread across two AZs. Monitoring over the last twelve months shows that only one web server is necessary to handle the minimum load. During core business hours (9-6 M-F), generally six to eight servers are needed. Five to six days per year the number of web servers required might go up to 20. Which of the following choices best reduces costs while providing full availability?

          A. Six reserved instances (heavy utilization), the rest covered by Spot instances

          B. Six reserved instances (heavy utilization), the rest covered by on-demand instances

          C. Two reserved instances (heavy utilization), six on-demand instances, the rest covered by Spot instances

          D. Two reserved instances (heavy utilisation), four Reserved instances (medium utilization), the rest covered by on-demand instances

  Answer: either B or D. Cloud Guru thinks D as you get two heavy util instances to cover the baseline capacity, and 4 medium util instances to cover up to 6 during normal business hours. The rest would be on-demand.


# High Availability

## Elasticity & Scalability 101
Elasticicity is used on short time periods (hours and minutes). Elasticity allows you to scale in and out to meet demands during a period.
  - Ec2: increase the number of EC2 instances based on autoscaling to distribute load
  - DynamoDb: Increase the IOPS for additional spikes in traffic
  - RDS: not very elastic, can't scale RDS based on demand

Scalability is different. It's more long term. More about "Scaling Up":
  - Ec2: increaasing instance size
  - DynamoDb: Unlimited amount to storage to help scaling
  - RDS: Increase instance size, e.g. from small to medium utilisation

Key points:
  - Elasticity is generally on a shorter time period than scalability
  - With elasticity you can generally scale in in an automated way when demand reduces
  - With scalability you're making longer term decisions around the capacity of your infrastructure

## Scaling Up
Traditional IT - increase the number of processors, the number of RAM, or the amount of storage.
EC2 - increases the instance type from say t1.micro to t2.small, t2.medium, etc.

The problem with scaling up is that eventually you'll reach a "hard limit" of compute capacity. This is when you'll look at scaling out.

## Scaling Out
Traditional IT - adding more resources (such as webservers).
EC2 - adding additonal EC2 instances based on demand.

  * Exam Tips:
    - In the old days, the size of the instance determined it's overall network performance (to the internet, to other instances, to EBS volumes, etc.). Amazon today are saying that network performance is uniform, but the exam may have old questions on this.
    - EBS volumes have higher thoughput (Mbps), higher max IOPS, and higher max bandwidth (MB/s) for different instance types.
    - The exam may ask you how you would resolve various network bottlenecks you're having. The answer would be to SCALE UP, rather than scale out, as per the old way of determining network performance in EC2.

    - Typical Exam Scenario:
        > Your infrastructure makes use of a single t2.small NAT instance within a VPC to allow inside hosts to communicate with the internet without being directly addressable.
        > As your traffic has grown, the amount of traffic going through the NAT has increased and is overwhelming it which is slowing down your infrastructure.
        > *NAT is Network Address Translation. It's a server you put in a public subnet, which allows your servers in a private subnet to be able to communicate with the internet through a standard interface without those servers being directly address (i.e. servers on the internet won't be able to access those servers communicating through the NAT*
        > Which 2 options should you do to alleviate this issue?
        > A) Add another internet gateway to your VPC  
        > B) Increase the instance size of the NAT from t2.small to t2.medium
        > C) Use Direct Connect to route all traffic instead
        > D) Add another NAT instance and configure your subnet route tables to be spread across two NATs

        My answer: B (in the old EC2 context), and D

  - Remember for the exam scenario above, A would be incorrect because you can only have a single internet gateway attached to a VPC!
  - Eliminate obviously wrong answers.
  - Ask yourself, where is the bottleneck? Is it NETWORK RELATED? If so, it's probably a SCALE UP answer.
  - Is it a problem in relation to not having enough RESOURCES (ie. you can't increase instance type any further)? If so, it's probably a SCALE OUT answer.
  - Remember elasticity. Scaling out, you can scale back easily. Scaling up is easy, scaling down is not so easy.

## RDS Multiple AZ Failover
When you setup RDS, you have the decision to enable multi-az failover. This safeguards your data in the event of a DB instance failure or loss of an availability zone.

  - Multi-AZ deployments for MYSQL, ORACLE, and POSTGRESQL engines utilise synchronous physical replication to keep data on standy up-to-date with primary.

  - Multi-AZ deployments for SQL SERVER use synchronous logical replication to achieve the same result. It employs the Server-native mirroring technnology.

  - Benefits:
    * High availability
    * Backups are takens from secondary, which avoids I/O suspension to the primary
    * Restore's are taken from secondary which avoids I/O suspension to the primary

  - Exam Tips:
    * Remember than Multi-AZ failover happens automatically as you have a DNS Name into your RDS instances rather than a physical IP
    * You can FORCE a failover from one AZ to another by rebooting your instance
    * Multi-AZ Failover is NOT A SCALING SOLUTION!!! Read Replica's are used to scale.

## RDS Read Replica's
Read replica's make it easy to take advantage of supported engine' built-in replication functionality to elastically scale out beyond the capacity constraints of a single DB instance for read-heavy database workloads (i.e. READ ONLY COPIES OF YOUR DATABASE).

  - When to use:
    * Scaling beyond the compute or I/O capacity of a single DB instance for read-heavy database workloads
    * Serving read traffic while the source DB instance is unavailable (i.e. backups or scheduled maintenance on the primary)
    * Business Intelligence / Data Warehousing purposes

  - You can have up to 5 read replicas per primary RDS instance

  - Supported Versions:
    * MySQL - version 5.6 (not 5.1 and 5.5, and only InnoDB engine)
    * PostgreSQL 9.3.5 or new
    * Aurora
    * MariaDB

  - Creating Read Replicas:

    1. AWS will snapshot your database (if multi-az is not enabled, the snapshot will occurr on the primary database and can cause brief I/O suspension for around 1 minute)
    2. When you create a read replica you'll get a new read replica DNS endpoint

  - You can promote a read replica to it's own standalone database. Doing this will break the replication link between the primary and secondary. You can then write to this database.

  - Exam Tips:

    * Q: Can I create a Read Replica in an AWS REgion different from that of the source DB instance?
      A: Yes, RDS supports cross-region Read Replicas.

    * On creating "Read Replicas", the exam may ask you how you can prevent I/O Suspension or maintain performance during the process of creating a replica. The answer to this is to simply allow multi-az failover first, as then the snapshot will occurr on the secondary instance.

    * You can have up to 5 read replicas for both MySQL, PostgreSQL, Aurora, and MariaDB

    * You can have read replicas in different REGIONS

    * Replication is asynchronous only

    * Read replicas's can be built off multi-az databases

    * You can have read replicas of read replicas, however only in MySQL, and there will be some additional latency

    * DB Snapshots and automated backups CAN NOT be taken of read replicas

    * Again, from a monitoring perspective you may be asked how to diagnose issues where data is not being updated fast enough in the read replica. A lot of the time, the answer it just to look at the REPLICA LAG metric.

    * Have a clear understanding of the difference between non-multi-az rds instances and multi-az deployments

    * Read replicas require DB Snapshots or Backups

    * You can change the license model, db instance type, and whether or not the instance will be multi-az when restoring an instance from a snapshot or modifying in the console (this will result in downtime)

    * Consider read replica latency when placing read replicas in regions

    * Modifying single-az RDS instances can result in downtime. You'll just likely have performance issues when modifying multi-az instances.

    * Read replicas can not be multi-az enabled (but you can have multiple read replicas in different availability zones)

    * Wait for read replica latency to be zero before promoting a read replica to a primary database in production as not all transactions may be captured (i.e. the one's that are incomplete)

    * You can force a failover from the primary to a secondary by rebooting the primary

## Bastion Hosts & High Availability
A bastion host is a security measure that you can implement which acts as a gateway between you and your Ec2 instances. The bastion host helps to reduce attack vectors on your infrastructure and means that you only have to harden 1 or 2 EC2 instances rather than your entire fleet.

Private Subnet = Not directly internet accessible
Public Subnet = Are directly internet accessible

You'll have a bastion EC2 instance sitting in a public subnet with an elastic IP address, which you can securely access via RDP or SSH. Once connected to this box, you can access via SSH or RDP those EC2 instances that are in your private subnets.

Exam Tips:
  * 1 Subnet = Availability Zone
  * Bastion hosts come up in the exam in the context of high availability architecture. All you need to know is the network specifics of a bastion host in a VPC, and to enable HA you can use ROUTE53 to fail over to a secondary bastion host in another public subnet (and availability zone implicitly).

## Troubleshooting and Potential Autoscaling Issues
Here are list of things to look for if your instances are not launching in to an ASG:

  - Associated Key Pair does not exist (may have been inadvertently deleted)
  - Security group does not exist anymore
  - Autoscaling config is not working correctly
  - Autoscaling group was not found
  - Instance type specified is no longer supported by AWS in the availability zone
  - AZ is no longer supported
  - Invalid EBS device mapping
  - Autoscaling service is not enabled on your account (can happen on older accounts or there has been an IAM policy change)
  - Attempting to attach an EBS block device to an instance-store AMI

Exam Tips:
  * Wouldn't worry too much about this, but keep the above in mind (most common ones are the key pair, security group, and asg config related issues).

## Route53 Failover
If you had a large multi-region application, you could use Route53 Latency-based Routing Record Sets that resolve to Elastic Load Balancers in each region.

If you are designing to check for loss of contact with the instances you need to use "Evaluate Target Health" to confirm connectivity. The Latency policy on the Route53 record will eventually detect the unavailability, however it is not a real time test.

## Additional Information

  - RTO = Recovery Time Objective in the context of disaster recovery


# Deployment & Provisioning

## Services allowing root/admin access to OS
Remember for the exam, there are four services which give you access to the root user:

  1. Elastic Beanstalk
  2. Elastic MapReduce
  3. OpsWork
  4. EC2

Exam Tips:
  - There's a good chance this will be worth 1 point in your exam, so remember these services (four above).
  - Remember that you DO NOT have access to RDS, DynamoDB, S3, or glacier (they may try and trick you by conflating "admin access" with what you can do in the AWS Console)

## Elastic Load Balancer Configurations
  - You can use ELBs to load balance across different availability zones within the same region, but not to different regions (or different VPC's) themselves.

  - An ELB and a NAT are different things entirely.

  - Two Types of ELBs:
      1. External ELBs (with external DNS names)
      2. Internal ELBs (with internal DNS names)

  - You can configure health checks (ping protocol, port, path, timeout, interval, thresholds for healthy and unhealthy)

  - Sticky sessions or "session affinity" are not enabled by default in the load balancer. Enabling this allows you to lock a user down to a particular instance of the ELB. The key to managing sticky sessions is to determine how long you load balancer should maintain the session with the server.

  - There are two types of session stickiness:

    1. Duration based:
      - load balancer creates Cookie based on a "stickiness policy"
      - The cookie is automatically updated after after its duration.
      - If the instance attached to the load balancer and cookie dies, the load balancer invalidates the Cookie and re-assigns it to a healthy instance

    2. Application controlled:
      - load balancer uses a special cookie to associate the session with the origin server
      - key difference here is that the application controls the expiry of the cookie by associating the load balancer cookie with an existing cookie created by the application
      - load balancer only inserts a new stickiness cookie if the application response includes a new application cookie
      - load balancer cookie does not update with each request
      - if application cookie is explicitly removed or expires, the session stops being sticky until a new application cookie is issued
      - if an instance if an unhealthy, the load balancer redirects traffic to an instance which is healthy using the algorithm assigned. It's up to the application to deal with sessions it has never seen before

  - CloudWatch metrics for the ELB:
    * LATENCY: time measured in seconds from when the load balancer establishes a request with a web server to when it receives a response
    * SURGE QUEUE LENGTH: important for exam
    * SPILLOVERCOUNT: important for exam

  - Exam Tips:
    * Know the different types of stickiness
    * Do not necessarily need to know the failover behaviour

    * Typical exam question:
        - A customer has an online store that uses cookie-based sessions to track logged-in customers. It is deployed on AWS using ELB and ASG. When the load increases, Auto Scaling automatically launches new web servers, but the load on the web servers does not decrease. This causes the customers a poor experience. What could be causing this?
          * A. The ELB DNS record's TTL is set too high
          * B. The ELB is continuing to send requests with previously established sessions
          * C. The website uses CloudFront which is keeping sessions alive
          * D. The new instances are not being added to the ELB during the ASG cool down period

        * Answer: B (due to session stickiness, the load balancers are not distributing traffic to newly created instances)

    * Typical ELB monitoring exam question:
        - You want to monitor your web application to make sure that is maintains a good quality of service for your customers which is defiend by the application's page load time. What CloudWatch metric should you use to accomplish this?
          * A. Latency reported by the ELB
          * B. Request Count reported by the ELB
          * C. Total CPU Utlization for the ELB
          * D. Total Network throughput for the ELB

        * Answer: A

## Pre-warming the ELB
AWS staff can pre-configure the load balancer to have the appropriate level of capacity based on expected traffic. Useful for when "flash traffic" is expected or when load tests are about to be started.

You typically need to provide AWS with three critical pieces of information:

  1. Start and end dates of your expected surge in traffic
  2. The expected request rate per second
  3. The total size of the typical request/response that you will be handling

# Data Management

## Disaster Recovery (DR)
Is about preparing for and recovering from a disaster. Any event that has a negative impact on a company's business continuity or finances could be termed a disaster.

Traditional approach to DR usual involves an N+1 approach and has different levels of off-site duplication of data and infrastructure:

  - Facilities to house the infrastructure
  - Security to ensure physical protection of assets
  - Suitable capacity to scale the environments
  - Support for repairing, replacing, and refreshing the infrastructure
  - Contracts with ISPs to provide bandwidth
  - Network infrastructure (firewalls, routers, switches, and load balancers)
  - Enough server capacity
  - DNS and DHCP

### Why use AWS for DR?

  - Only minimum hardware is required for "data replication"
  - Allows you to be flexible depending on what your disaster is and how to recover from it
  - Opex cost model (pay as you use)
  - Automate DR

### What services?

  - Regional deploys
  - Storage
  - S3 - 99.9999999999% durability and cross region replication
  - Glacier - same as S3
  - EBS
  - Direct Connect
  - AWS Storage Gateway: essentially a virtual appliance that you install on-site that sends information back to S3 for DR using direct connect or a VPN tunnel
  - EC2 and EC2 VM Import Connector (virtual appliance which allows you to import virtual machine images from your existing environment to Amazon EC2 instances)
  - Networking (Route53, ELB, VPC, Direct Connect)
  - Databases (RDS, DynamoDB, Redshift)
  - Orchestration (Cloudformation, ElasticBeanstalk, OpsWork)
  - LAMBDA

### Three Types of AWS Storage Gateways:

  1. Gateway-cached volumes - store primary data and cache most recently used data locally (if there is data infrequently accessed, this is a good approach). If there's a cache miss, the gateway retrieves the file from S3 for the user. Bit of latency in this scenario however.

  2. Gateway-stored volumes - store entire dataset on site and asynchronously replicate data back to S3. Good for fast access.

  3. Gateway-virtual tape library - store your virtual tapes in either S3 or Glacier (batched backups or archives).

### RTO vs RPO
  - Recovery Time Objective: length of time from which you can recover from a disaster
  - Recovery Point Objective: is the amount of data your organisation is prepared to lose in the event of a disaster (1 days worth of emails, 5 hours of online transactions record etc)
  - Typically the lower RTO & RPO the more costly your infrastructure will become

### DR Scenarios
Respectively, each method described here goes from highest to lowest RTO/RPO.

#### Backup & Restore
  - In most traditional environments, data is backed up and then send off-site. This method can be lengthy process from a timing perspective.
  - Amazon S3 is an ideal system in the event of a disruption or disaster as data is typically transferred to S3 via a network it can be restored from any location
  - You can use AWS Import/Export to transfer very large data sets by shipping storage devices directly to AWS
  - For longer-term storage where retrieval times of several hours are adequatem there is Amazon Glacier, which has the same durability model as Amazon S3.
  - S3 and Glacier can be used in conjunction to create a tiered backup solution (EXAM)

  - Key steps for a backup & restore:
    * Select an appropriate tool or method to back up your data into AWS
    * Ensure that you have an appropriate rentention policy for this data
    * Ensure that appropriate security measures are in place for this data, including encryption and access policies
    * Regularly test the recovery of this data and restoration of the system

#### Pilot Light (popular on exam)
  - The term pilot light is often used to describe a DR scenario in which a minimal version of an environment is always in running in the cloud.
  - Infrastructure elements for the pilot light typically includes database servers replicating data to Amazon EC2 or RDS (critical core). This might also include active directory or exchange servers.
  - To provision the remainder of the infrastructure to restore business-critical services, you would typically have some preconfigured servers bundled as AMIs that you could enable at a moments notice
  - From a network perspective, you have two main options for provisioning:
    1. Use pre-allocated elastic IP addresses and associated them with your instances when invoking DR. You can also use pre-allocated elastic network interfaces (ENIs) with pre-allocated MAC Addresses for applications with special licensing requires (EXAM)
    2. Use ELB to distribute traffic to multiple instances. You then need to update your DNS records to point at your Amazon EC2 instances or load balancer CNAME.

#### Warm Standby
  - Warm standby extends the pilot light scenario, whereby a scaled-down version of a fully functional environment is always running in the cloud.
  - In a disaster, the system is scaled up quickly to handle production load (scaling up or out, but scaling out is preferred).
  - Key steps for preparation:
    1. Set up Amazon EC2 instances to replicate or mirror data
    2. Create and maintain AMIs
    3. Run your application using a minimal footprint of Amazon Ec2 instances
    4. Patch and update your software and config in line with your live environment
  - Key steps for recovery:
    1. Increase size of fleet or scale up
    2. Manually update DNS or use Route53 Health Checks to Automate Failover
    3. Consider using ASG rules to right-size the cluster
    4. Add resilience or scale up your database

#### Multi-site solution
A multi-site solution runs in AWS as well as on your existing on-site infrastructure in an active-active configuration. The data replication method that you employ will be determined by the recovery point that you choose.

You can use Route53 to root traffic to both sites either symmetrically or asymmetrically.

In an on-site disaster situation, you adjust the DNS weighting and send all traffic to AWS servers. The capacity of the AWS service can be rapidly increased to handle the full production load. You might need some application logic to detect the failure of the primary database services and cut over to the parallel database services running in AWS.

Key steps for multi-site prep:
  1. Either manually or by using DNS failover, change the DNS weighting so that all requests are sent to the AWS site
  2. Have app logic for failover to use the local AWS database servers for all queries
  3. Consider using ASG rules to atomatically right-size the AWS fleet

### Failing Back

#### Backup and Restore
  1. Freeze data changes to the DR site
  2. Take a backup
  3. Restore the backup to the primary site
  4. Re-point users to the primary site
  5. Unfreeze the changes

#### Pilot light, warm standby, and multi-site
  1. Establish reverse mirroring/replication from the DR site back to the primary, once the primary site has caught up with the changes
  2. Freeze data changes to the DR site
  3. Re-point users to the primary site
  4. Unfreeze the changes

### Exam Tips
  - You WILL be asked what the appropriate device for replicating data back to Amazon S3 for DR purposes will be. So you will need to understand the 3 types of AWS Storage Gateways.
  - Know that S3 and Glacier work together to create a tiered backup solution
  - Know that ENIs can be pre-allocated in DR scenarios where software requires special licensing
  - Horizontal scaling is always going to be preferred in a warm standby scenario
  - Two DR scenarios likely to come up on the exam with be backup and restore, and pilot light.
  - Pay particular attention to the Pilot Light Scenario and specifically the fact that you can have ENIs with preconfigured MAC addresses for licensing purposes!

## AWS Services and Automated Backups

  * Services which support backups:
    - RDS
    - ElastiCache (Redis Only)
    - RedShift

  * Services that do not have Automated Backup
    - EC2 (snapshots do not count, as they are not automated)

### RDS Automated Backups

  * Requires InnoDB Transactional Engine
  * Performance hit if Multi-AZ is not enabled
  * Instance deletion deletes all automated backups (manual snapshots will not be deleted)
  * All stored on S3
  * EXAM: When you do a restore, you can change the engine type (SQL Standard to SQL Enterprise for instance) provided you have enough storage space

### Elasticache Automated Backups

  * Available for Redis only
  * Entire cluster is snapshot
  * Snapshot will degrade performance
  * All stored on S3

### RedShift Automated Backups

  * Stored on S3
  * By default, enables automated backups of your data warehouse with a 1-day retention rate
  * Amazon Redshift only backs up data that has changed so most snapshots only use up a small amount of your free backup storage

### Backing up EC2 instances

  * No automated backups
  * Manual snapshots degrade performance (schedule these times wisely)
  * Stored on S3
  * Can create automated backups using command line interface or Python
  * They are incremental:
    - Snapshots only store incremental changes since last snapshot
    - Only charged for incremental storage
    - Each snapshot still contains the base snapshot data

## EC2 Types - EBS vs Instance Store

  * Instance store = ephemeral
  * EBS allows for permanent storage of data
  * Different volume types:
    1. Root Volume - OS is installed here
    2. Additional Volume

  * Root volumes:
    - Root Device volumes can either by instance or EBS store
    - An instance store volume's maximum size is 10GB
    - EBS root device volume can be up to 1 or 2TB depending on the OS
    - EBS Root Device Volumes are terminated by default (can be set otherwise)

  * Additional Volumes:
    - If instance volume, they will be deleted when the attached EC2 instance is terminated
    - EBS additional volumes will persist

  * Rebooting instances will allow the instance store to persist it's data

  * Instance store data is lost under the following circumstances:
    - Failure of underlying drive
    - Stopping an Amazon EBS-backed instance (where the instance store is an additional volume)
    - Terminating an instance

  * DO NOT RELY ON INSTANCE STORE VOLUMES FOR VALUABLE, LONG-TERM DATA (store on EBS volume, or in S3)

  * Exam Tips:
    - If it's an instance store backed EC2 instance you can't upgrade instance type, kernel, RAM disk, or user data.
    - Stopped state: you can't stop an instance store backed EC2 instance without losing the data on the volume
    - "Delete on Termination" is the default for all EBS root device volumes. You can this to FALSE, but ONLY at instance creation time
    - Additional volumes will persist automatically (EBS-only)
    - EBS Volumes can be changed on the fly (except for the magnetic standard where you have to use the snapshotting approach above)
    - Best practice is to stop the EC2 instance and then change the volume
    - You can change volume types by taking a snapshot and then using the snapshot to create a new volume
    - If you change a volume on the fly you must wait for 6 hours before making another change
    - You can scale EBS Volumes UP ONLY
    - Volumes must be in the same AZ as the EC2 instance (you can take a snapshot of a volume and re-create the volume in a different AZ if necessary)

## Storing Log Files & Backups

  * Centralised Monitoring & Logging:
    - Using a third-party, centralised monitoring platform, such as Zennos, Splunk, RSyslog, Kiwi, etc. These logs can then be stored on S3.
    - Using CloudWatch Logs
    - Access Logging on S3 Buckets

  * The best way to store to store your logs is probably S3
    - 11 Nines durability (99.9999999%)
    - Life cycle management (log rotation)
    - Achive off to Glacier (cheaper archive storage that S3)
    - Effectively allows you to tier your backup solution

  * Typical Exam Scenario:
    - You need to implement a tiered storage strategy for your database backups and logs. The backups need to be archived to a durable storage solution at the end of each day. After 28 days, the backups must be archived off to cheaper storage but must still be retained for legislative reasons. Which tiered storage proposal meets the recovery scenario minimises costs and addresses the legislative requirements?
         * A. Use an independent EBS volume to store the backups and log files. Take daily snapshots of this volume. After 28 days, rotate your EBS snapshots.
         * B. At the end of the day back up your database and copu the backups files to S3. AFter 28 days copy the data from S3 to RDS Log File Collector.
         * C. Store you log files on EC2 ephemeral storage and then randomly terminate your EC2 instances. Some men just want to watch the world burn.
         * D. Use an independent Amazon EBS volume to store the daily backups and copy the backup files to S3. Create a lifecycle policy on S3 to archive off the backups after 28 days.

  * Quiz:
    - 300GB MongoDB database which requires random I/O reads greater than 110,000 4kB IOPS.
        ANSWER: Any I2 instance type has been optimised for random IO NoSQL DBs

# OpsWorks

## Overview
AWS OpsWorks provides a simple way to create and manage stacks and their associated applications and resources.

Official Amazon Definition:
"AWS OpsWorks is an application management service that helps you automate operations tasks like code deployment, software configurations, package installations, database setups and server scaling using Chef. OpsWorks gives you flexibility to define your application architecture and resource configuration and handles the provisioning and management of your AWS resources for you. OpsWorks includes automation to scale your application based on time or load, monitoring to help you troubleshoot and take automated action based on the state of your resources, and permissions and policy management to make management of multi-user environments easier."

### What is Chef?
Chef turns infrastructure into code. With Chef, you can automate how you build, deploy, and manage your infrastructure.

  This makes it:
    - Versionable
    - Testable
    - Repeatable

Chef server stores your recipes (infrastructure code) as well as other configuration data. The Chef client is installed on each server, virtual machine, container, or networking device you manage - these are called nodes. The client periodically polls Chef server for latest policy and state of your network. If anything on the node is out of date, the client brings it up to date.

### What is OpsWorks

  * GUI to deploy and configure infrastructure quickly
  * OpsWorks consists of two elements:
    1. Stack: is a container or group of resources such as ELBs, EC2 instances, RDS instances etc.
    2. Layer: exists within a stack and consists of things like a web application, or database layer.

  * Layers:
    - one or more in a stack
    - an instance must be assigned at least 1 layer
    - which chef layer runs is determined by the layer the instance belongs to
    - Preconfigured Layers include applications, databases, load balancers, caching

  * Instance Types:
    - 24x7 Instances (default)
    - Time-based instances (specific schedule for instances starting)
    - Load-based start instances based on CPU, Memory, and application load changes across all instances in a layer

  * Under layers, in the app server, Recipe's can be provided for different "lifecycle events"
     - Setup
     - Configure
     - Deploy
     - Undeploy
     - Shutdown

  * Exam Tips:
    - After you attach an ELB to a layer, OpsWorks removes any currently registered instances and then manages the load balancer for you. If you subsequently use the ELB console or API to modify the configuration, the changes will not be permanent.

# Security

## Building IAM Policies

AWS Managed Administrator Access Policy Example:

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": "*",
              "Resource": "*"
          }
  	]
  }
  ```

Statements - You can have multiple statements as part of a policy
Effect - "Allow" or "Deny"
Action - AWS Action
Resource - An Amazon Resource Name (ARN) to restrict the policy to a given resource or array of resources

## Roles & Custom Policies

  * Once you create a policy, the name cannot be changed
  * You can attach an IAM Role to an EC2 instance a launch time
  * You can assign a role to an existing EC2 instance
  * Can take 5-10 seconds when changing policies
  * You can have up to 5 versions of a policy. At which point you need to delete old versions or request a service limit increase from AWS.

## S3 CLI & Regions

  * You can create different buckets in different AWS regions
  * YOU NEED TO SPECIFY THE REGION FLAG WHEN OPERATING THE S3 CLI
  * Understanding this can help you with troubleshooting questions on the exam potentially

## IAM & MFA

  * You can active MFA in the IAM console using virtual (application) or hardware (mobile phone) options
  * Using the supported MFA app, you need to scan a QR code and enter 2 codes

## Security Token Service (STS)

  * Grants users limited and temporary access to AWS resources. Users can come from three sources:

    1. Federation (typically Active Directory)
      - Uses Security Assertion Markup Language (SAML)
      - Grants temp access based off the users AD credentials, does not need to be an IAM user
      - Single sign on allows users to log in to AWS console without assigning IAM credentials

    2. Federation with Mobile Apps
      - Use FB, Amazon, Google, or other OpenID providers to log in.

    3. Cross-account access
    	- Let's users from one AWS acount access resources in another

  * Federation: combining or joining a list of users in one domain (such as IAM) with a list of users in another domain (such as AD, of Facebook)

  * Identity Broker: a service that allows you to take an identity from point A and join it (federate it) to point B. Often must be custom built ( -- EXAM --).

  * Identity Store: Services like AD, Facebook, Google, etc.

  * Identities: a user of a service like Facebook etc.

  * Exam Scenario:
    You are hosting a company website on some EC2 web servers in your VPC. Users of the website must log in to the site which then authenticates against the companies active directory servers which are based on site at the companies head quarters. Your VPC is connected to your company HQ via a secure IPSEC VPN. Once logged in the user can only have access to their own S3 bucket. How do you set this up?

      Employees --> Enterprise Reporting App --> Identity Broker --> LDAP/STS ----- IAM <--> S3

     * Breaking down the scenario:
    	1. Employee enters username and password
    	2. App calls Identity Broker. Broker captures username and password
    	3. Identity Broker users the organisations LDAP directory to validate the username and password
    	4. The Identity Broker calls the new GetFederationToken function using the IAM credentials. The call must include an IAM policy and a duration (1 to 36 hours), along with a policy that specifies the permissions to be granted to the temporary security credentials.
    	5. The Security Token Service confirms that the policy of the IAM user making the call to GetFederationToken gives permission to create new tokens and then returns four values to the applications: An access key, a secret access key, a token, and a duration (the token's lifetime)
    	6. The identity broker returns the temp security credentials to the reporting application
    	7. The data storage application uses the temp security credentials (including the token) to make requests to Amazon S3
    	8. Amazon S3 uses IAM to verify that the credentials allow the requested operation on the given S3 bucket and key
    	9. IAM provides S3 with the go-ahead to perform the request operation

     * Distilling the scenario for the exam:
    	1. Develop an identity broker to communicate with LDAP and AWS STS
    	2. Identity Broker always authenticates with LDAP first, THEN with AWS STS
    	3. Application then gets temporary access to AWS resources

     * Alternative exam scenario:
    	1. Develop an Identity Broker to communicate with LDAP and AWS STS
    	2. Identity Broker always authenticates with LDAP first, gets an IAM Role associated with a user
    	3. Application then authenticates with STS and assumes that IAM Role
     	4. Application uses that IAM role to interact with S3

## Overview of Security Processes

  * Shared Security Model (Exam)
    - AWS is responsible for the security of the underlying infrastrcuture (usually up until the host)
    - You are responsible for the anything you put on the cloud or connect to the cloud

  * Storage Decommissioning
    - When a storage device has reached the end of its useful life, AWS procedures are to destroy the disks

  * Network Security
    - Transmission Protection - you can connect to an AWS access point via HTTP or HTTPS using SSL.
    - Customers who require additional layers of network security, AWS offers VPC which provides private subnets within AWS cloud, with ability to use IPSec VPN devices to establish encrypted tunnels.
    - Amazon Corporate Segregation - logically, the AWS Production network is segregated from the Amazon Corporate netowrk by means of a complex set of network security / segregation devices
    - Network Monitoring & Protection - DDoS, MITM, IP Spoofing, Packet Sniffing by other Tenant
      - Ip Spoofing
        - AWS-Controlled, host-based firewall infrastructure will not permit an instance to send traffic with a source IP or MAC address other than its own.
        - (EXAM) Unauthorised port scans by Amazon EC2 customers are a violation of the AWS Acceptable Use Policy. You may request permission to conduct vulnerability scans as required to meet your specific compliance requirements. These scans must be limited to your own instances and must not violate the AWS Acceptable use policy. YOU MUST REQUEST A VULNERABILITY SCAN IN ADVANCE.

  * AWS Credentials
    - Passwords - assigned to all IAM users
    - MFA - always recommended for root account
    - Access Keys - for CLI
    - Key Pairs - to access EC2 instances via SSH
    - X.509 Certificates (EXAM) - making sure S3 is secure, currently only used with S3 to sign SOAP-based requests. You can have AWS create an X.509 certifacte and private key that you can download, or you can upload your own certificate using the Security Credentials page. These are also referred to as "Signed Requests". You'll get a special signed URL that you can provide to users which will only allow access to those users.

  * AWS Trusted Advisor
    - Inspects your AWS environment and makes recommendations when opportunities may exist to save money, improve system performance, or close secuirty gaps.
    - Very basic suite of changes that it recommends in terms of security
    - (EXAM): AWS Trusted Advisor only relates to IAM

  * Instance Isolation
    - Xen Hypervisor provides isolation between instances
    - AWS firewall resides within the hypervisor layer, between the physical network interface and the instance's virtual interface. So all packets must pass through this layer, thus an instance's neighbours have no more access to that instance than any other host on the internet.
    - The physical RAM is separated using similar mechanisms
    - Customer instances have no access to raw disk devices, but instead are presented with virtualised disks. AWS automatically resets every block of storage used by the customer.
    - Memory allocated to guests in scrubbed (set to zero) by the hypervisor when it is unallocated to a guest.

  * Other Considerations:
    - Guest OS are completely controlled by you, the customer
    - AWS does not have any access rights to the Guest OS
    - EC2 provides a complete firewall solution in a deny-all state by default
    - AWS provides the ability to encrypt volumes and snapshots with AES-256. Encryption features only available on more powerful instance types (M3, C3, R2, G2)
    - ELBs support SSL termination and identification of origin IP addresses of clients
    - (EXAM) Direct Connect - Bypass ISPs in your network path. You can procure rack space within the facility housing the AWS Direct Connect location and deploy your equipment nearby. Once deployed, you can connect this equipment to AWS Direct Connect using a cross-connect. Industry standard 802.1q VLANs, allow you to share the private address space of your data centres.

  * (EXAM) Read the Amazon Web Service: Overview of Security Processes whitepaper

## AWS & IT Audits

  * Compliance
    - SOC 1 / SSAE 16/ISAE 3402 (fomerly SAS 70 Type II)
    - SOC 2
    - SOC 3
    - FISMA, DIACAP, and FedRAMP
    - PCI DSS Level 1 (you need a delta certificate to have your cloud environment certified to accept card payments from a QSA)
    - ISO 27001
    - ISO 9001
    - ITAR
    - FIPS

  * Industry Specific Standards:
    - HIPAA
    - Cloud Security Alliance (CSA)
    - Motion Picture Association of America (MPAA)

  * Auditing on AWS
    - Your org may undergo an audit. This could be for PCI Compliance, ISO 27001, SOC, etc. There is a level of shared responsibility in regards to audits:
      - AWS Provides: their annual certifications and reports for global infrastructure including all hardware, datacentres, physical security, etc
      - Customer Provides: everything they have put on AWS, such as EC2 instances, RDS Instances, Applications, Assets in S3, etc.

  * Typical AWS Exam Question:

    Your organisation is preparing for a security assessment. What two configuration management practices should be implemented prior to the assessment?

    Choose two of the following:

      - A. Audit whether or not remote administrative access is performed securely
      - B. Verify that the AWS Trusted Advisor has identified and disabled all unnecessary users and services on your AWS EC2 instances
      - C. Verify that all AWS S3 bucket policies and ACL's correctly implement your security policies
      - D. Determine whether unnecessary users and services have been identified on all Amazon-published AMI's

    Answer: A & C (B is irrelevant to EC2 instances, D is only specific to Amazon AMIs)


# Networking

## Network Bottlenecks
If you see an exam question on network performance and one of the answers is to upgrade the instance to a higher type, that's probably going to be the correct answer even though it's slightly outdated. It's only EBS throughput which is affected by instance type. If you scale up, you get more throughput.

Exam Question:
Maintain an application on AWS to provide development and test platforms for your developers. Currently both environments consist of an m1.small EC2 instance. Your developers notice performance degradation as they increase network load in the test environment.

How would you mitigate these performance issues in the test environment?

  - A. Upgrade the m1.small to a larger instance type
  - B. Add an additional ENI to test instance
  - C. Use the EBS optimised option to offload EBS traffic
  - D. Configure Amazon CloudWatch to provision more network bandwidth when network utilisation exceeds 80%

Answer: Probably A

## Route53

### DNS 101

  * You can use IPv6 address space with AWS Services now
  * TLDs can be found a www.iana.org/domains/root/db
  * A domain registrar can assign domain names directly under one or more top-level domains. They are registered with InterNIC, a service of ICANN, which enforces uniqueness of domain names across the internet. Each domain becomes registered in a central database known as the WhoIS database.
  * The SOA Record stores information about:
    - The name of the server that supplied the data for the zone
    - The administrator of the zone
    - The current of the data file
    - The number of seconds a secondary name server should wait before checking for updates
    - The number of seconds a secondary name server should wait before retrying a failed zone transfer
    - The maximum number of seconds that a secondary name server can use data before it must either be refreshed or expire
    - The default number of seconds for the TTL file on resource records
  * The NS Record standards for Name Server Records and are used by TLD servers to direct traffic to the content DNS server which contains the authoritative DNS records
  * An "A" Record is an "Address Record", computer translates name of domain to IP address
  * TTL - Length of time a DNS record is cached on either the Resolving Server or the users own local PC. The lower the vlaue, the faster changes to DNS records take ot propagate throughout the internet (when migrating DNS servers, lower TTL to 5 minutes)
  * CNAME - Canonical Name can be used to resolve one domain to another.

#### Alias Records Vs CNAME

  * Alias Record (in AWS Route53) - Very similar to a CNAME. Used to map to resource recordsets in your hosted zone to ELBs, CloudFront distributions, or S3 Buckets that are configured as websites. Key difference to CNAME: can't be used for naked domain names (zone apex). You can't have a CNAME for acloud.guru it must be either an A record or alias.
  * Alias records save you time because Amazon Route53 automatically recognises changes in the record sets that the alias resource record set refers to.

#### Exam Tips

  * ELB's do not have a pre-defined Ipv4 addresses, you resolve them using a DNS Name
  * Understand the different between an Alias Record and a CNAME (basically you can apply alias records to the apex domain). You'll be charged in Route53 for CNAME requests, but not for Alias records.
  * Given the choice, always choose an Alias Record over a CNAME

### Route53 - Weighted Routing
You can configure Route53 to perform "weighted routing", whereby you route a percentage of traffic to different locations. It does this over a period of time, it does this over the length of the day - NOT INSTANT (your ISP may be caching the records for some time).

  * Create a record set in the apex domain.
  * Routing Policy: set to Weighted
  * The Weight property are values between 0 and 255. Example: two records have weights of 1 and 3 (sum = 4), so on average, Route53 selects the first record 25% of the time and other record 75% of the time.
  * Exam scenario may be A/B testing - weighted routing would be perfect for this.

### Route53 - Latency Routing
Route traffic based on the lowest network latency for the end user.

  * For the latency routing recordset, you have to set the "Region" field for your target alias, A record, or CNAME

### Route53 - Failover Routing
Route traffic based on the health of targets. Useful if you want to create an active/passive set up.

  * Click on Health Checks from the Route53 console
  * Health checks can be based on IP Addresses or Domain Names
  * Similar to ELB health check configuration
  * You can create SNS notifications on the health check failure
  * After configuring the health check, add a failover record, set "Evaluate Target Health" to "Yes" and then assign one of the pre-configured health checks.
  * Note that a health check cannot be assigned to a recordset if it's checking the zone domain name. Instead you must set it to the target alias, address, or CNAME.
  * Failover records can only be of a primary and secondary type.

### Route53 - GeoLocation Routing
Route traffic based on the geographic location of your users.

  * When creating the recordset, simply select the location (or in the US specific states)
  * Route53 will return a "no answer" response for queries from locations it can't resolve a record to. So it's a good idea to set a default recordset still even if you set up routing for every 7 continents.

### DNS Exam Tips

  * ELB's never have IPv4 or IPv6 address. Only a DNS Name.
  * Understand the difference between an Alias Record and a CNAME.
  * Given the choice, always choose an Alias Record over a CNAME.
  * Remember the different routing policies and their use cases:
    - Simple
    - Weighted
    - Latency
    - Failover
    - Geolocation

## Direct Connect
AWS Direct Connect makes it easy to establish a dedicated network connection from your premises to AWS. Using AWS Direct Connect, you can establish private connectivity between AWS and your datacenter, office, or colocation environment, which in many cases can reduce your network costs, increase bandwidth throughput, and provide a more consistent network experience than Internet-based connections.

  * Advantage of Direct Connect over VPN?
    - BANDWIDTH, THROUGHPUT
    - CONSISTENT NETWORK EXPERIENCE
    - Site-to-site VPN can just drop out but may be appropriate where there is immediate need, with low to modest bandwidth requirements, and can tolerate variability in internet-based connectivity.
    - AWS Direct Connect bypasses the internet, instead uses dedicated, private network connections between your intranet and Amazon VPC.
    - Exam: always go with bandwidth when asked for the benefit

## VPC

## VPC Overview
Virtual Private Cloud allows you to provision a logically isolated section of the Amazon cloud.

You can customise the network configuration for your Amazon VPC. For example, you can create public-facing subnet for your webservers that has access to the internet, and private servers in a private subnet for sensitive backend systems such as database servers. Additionally, you can create hardware network connections between your VPC and on premises environment.

IGW / VPG --> Router --> Route Table --> Network ACL --> Security Group --> Pub/Priv Subnet

  * Document RFC 1918 - Defines three private IP address ranges used around the world:
    - 10/8 Prefix (10.0.0.0 - 10.255.255.255)
    - 172.16/12 Prefix (172.16.0.0 - 172.31.255.255)
    - 192.168/16 Prefix (192.168.0.0 - 192.168.255.255)
    - Largest network you can have is a /16

  * VPG - Virtual Private Gateway, where we terminate our VPN connection
  * IGW - Internet Gateway, we we accept connections from the internet into the VPC
  * Route Table
  * Network ACL - Access Control List - Second Layer of Defence
  * Subnets are always going to have different address ranges. Subnets are always mapped to a different availability zone.

  * What can you do with a VPC?
    - Launch instances into a subnet of your choosing
    - Assign custom IP address ranges in each subnet
    - Configure route tables between subnets
    - Create internet gateway and attach it to our VPC (1:1 IGW:VPC)
    - Much better security control over your AWS Resources
    - Instance security groups
    - Subnet network access control lists (ACLs)

  * Default VPCs:
    - Default VPC is user friendly, allowing you to immediately deploy instances
    - All Subnets in default VPC have a route out to the internet
    - Each EC2 instance has both a public and a private IP address
    - If you delete the default VPC you have to contact AWS and raise a ticket to restore it

  * VPC Peering:
    - Allows you to connect one VPC to another via a direct network route using private IP addresses.
    - Instances behave as if they were on the same private network
    - You can peer VPCs between accounts
    - Peering is in a star configuration (i.e. 1 central VPC peers with 4 others NO TRANSITIVE PEERING).

  * TRANSITIVE PEERING?
    - One VPC can't talk to another VPC via another VPC. VPC Peering has to be configured point-to-point.

  * Exam Tips:
    - Think of a VPC as a logical datacenter in AWS
    - Consists of IGW, VPG, Route Tables, Network ACLs, Subnets, Security Groups
    - 1 Subnet = 1 AZ
    - Security Groups are Stateful, Network ACLs are Stateless (lecture on this later)
    - NO TRANSITIVE PEERING

## Setting up a Custom VPC

 * Exam Tips:
    - Maybe remember that besides the .0 and .255 addresses, AWS will always reserve 3 IP addresses in every subnet you create.
    - An Internet Gateway allows an instance in public subnets to connect to the internet if it has a public IPv4 address or IPv6 address. Given these public addresses though, traffic from the internet can be routed to instances in those public subnets.

  * Security groups are stateful - outbound traffic is enabled by default

## Creating a NAT
We need secure internet access from our private subnet. There are two ways in which we can do this:

  - NAT Instance: an EC2 instance that will act as a gateway to the internet
  - NAT Gateways: a managed NAT service as part of the AWS VPC

### NAT Instance
To create a NAT Instance, just deploy an instance with the standard Amazon Linux VPC NAT AMI into your public subnet.

After creating the NAT instance, go to actions > networking > "Change Source/Destination Check" and disable this (we do this because by default EC2 instance are required to either be a source or destination for traffic, but for NAT Instances they're neither a source or destination, but merely a gateway through which other instances connect to destinations).

Now we need to add a route to our NAT instance, so edit the default route table for the VPC and route all traffic (destination 0.0.0.0/0) to the target which should pop up as the EC2 NAT Instance (target instance).

### NAT Gateway
Just go to the VPC console, create a NAT Gateway (assign to a public subnet always, create new EIP).

Edit your default/main route table to route all traffic (dest 0.0.0.0/0 to the nat gateway).

### Exam Tips
  * Exam Scenario: NAT Instances may need to be scaled up to the benefit of network performance.
  * NAT Instances feature heavily on the exam
  * When creating a NAT Instance, disable source/destination check on the instance
  * NAT instance must be in a public subnet
  * There must be route out of the private subnet to the NAT instance in order for this to work
  * The amount of traffic that the NAT instance supports, depends on the instance size. If you are bottlenecking, increase the instance size
  * You can create high availability auto-scaling groups, multiple subnets in different AZ's and a script to automate failover
  * NAT Instances are always behind a security group
  * NAT Gateways
    - Very new
    - Preferred by enterprise
    - Scale automatically up to 10Gbps
    - No need to patch
    - Not associated with security groups
    - Automatically assigned a public IP address
    - Remember to update your route tables to route out through the NAT
    - No need to disable src/dst checks on a NAT Gateway

## Network ACLs vs Security Groups

  * A security group operates at the instance level first (first layer of defence) and an ACL operates at the subnet level (second layer of defense).

  * Security Groups support allow rule only, while ACLs support ALLOW AND DENY rules.

  * Security groups are STATEFUL - that is, return traffic is automatically allowed. Whereas, ACLs are STATELESS in that all traffic in and out must have explictly defined rules.

  * We evaluate all rules on a security group before deciding whether or not to allow traffic. ACLs however process rules in numerical order when deciding whether to allow traffic, which means that you can explicitly override rules by giving them higher weightings.

  * Security groups only apply to specific instances, whereas a network ACL applies to all instances in a subnet.

  * 1 Subnet = 1 AZ = 1 ACL

### Network ACL

  * You can only associate 1 subnet with 1 ACL.
  * Best practice to specify rule numbers in increments of 100
  * You must specify inbound and outbound rules as ACLs are stateless
  * SPECIFYING INBOUND AND OUTBOUND RULES IS NOT ENOUGH!
    - You must specify the ephemeral port range 1024-65535 as an allow rule in your ACL as often a client initiates a request choosing an ephemeral port range to communicate on (e.g. a request comes in through a VPC to a webserver, your webserver will talk back to the client on ports 1024-5000 if they're using Windows XP. A NAT Gateway uses ports 1024-65353). These only apply stricrly to outbound rules.

### ACL Exam Tips

  * Your VPC automatically comes with a default network ACL which allows ALL OUTBOUND AND INBOUND TRAFFIC.

  * Custom ACLs are DENY ALL by default

  * 1 Subnet == 1 ACL (if not associated, it will be associated with the default ACL)

  * 1 ACL can apply to many subnets

  * Network ACL contains a numbered list of rules that is evaluated in order, starting with the lowest rule

  * Network ACLs always have separated inbound and outbound rules that can be set to ALLOW or DENY

  * Network ACLs are stateless; responses to allow inbound traffic are subject to the rules for outbound traffic (and vice versa)

  * You can block IP addresses using network ACL's, not security groups.

## Custom VPC's and ELBs

  * If you want high availability from your load balancer you'll want at least 2 selected subnets.

  * "This is an Internet-facing ELB, but there is no Internet Gateway attached to the subnet you have just selected: subnet-01bdae65" <-- the load balancer would not be able to serve traffic to the private subnet

  * If you want something to be highly available, you want at least two subnets for your ELB.

## NAT's vs Bastions

  * A NAT instance is used to route traffic from instances within the private subnets out to the internet, but disallow access from the internet to those same instances.

  * Whereas a bastion host or jump box allows secure access to specific users for administrative access to hosts in a private subnet.

  * You'll get questions around how to make bastions highly available (autoscaling groups, multiple subnets, route53 running health checks on bastion server).

  * NAT Instances would require a script to automatically failover your NATs on the private instances.

## VPC Flow Logs

  * Enable you to capture all traffic to and from network interfaces and report it to cloudwatch

  * You need a cloudwatch log group and IAM role

  * Results look like:
  `2 196116328602 eni-3d4d1b71 10.0.2.44 10.0.1.221 58222 80 6 3 180 1503405980 1503406040 ACCEPT OK`

## Final Exam Tips for VPCs

  * As preparation, create a VPC from scratch, from memory. With both public and private subnets, NAT Gateways/Instances, Bastion hosts, Network ACLs, Flow Logs, etc.

  * Think of a VPC as a logical datacenter within AWS

  * VPCs can span multiple availability zones, but not regions

  * VPCs consist of IGW or VPGs, route tables, network ACLs, subnets, security groups

  * 1 Subnet = AZ

  * Security groups are stateful, ACLs are stateless ("state" in this case refers to whether inbound traffic is automatically associated to outbound traffic - i.e. security groups only care about ALLOW rules for a given port because the record the state of traffic as it goes in and automatically allow it as the traffic returns to the client, whereas ACLs must have explicit rules defined for inbound and outbound traffic).

  * You can peer VPCs in the same account and other AWS accounts

  * NO TRANSITIVE PEERING (i.e. you can't get from VPC A to VPC C through VPC B like this (A) --> (B) --> (C) )

  * When creating a NAT instance, Disable Source/Destination Check on the instance (which just basically disables the requirement for the instance to be a source or destination for traffic)

  * NAT instances must be in a public subnet and have an elastic IP to work

  * There must be a route out of the private subnet to the NAT instance in order for this to work

  * The amount of traffic that NAT instances support depends on the instance size - if you're bottlenecking, increase the instance size.

  * You can create high availability using ASGs, multiple subnets, and scripts to automate failover of NAT instances

  * NAT Instances are always behind a security group

  * NAT Gateways preferred by enterprise

  * NAT Gateways automatically scale up to 10Gbps, no need to patch, not associated with security groups, automatically assigned public IP address, and remeber to update your route tables to route traffic through the NAT Gateway outbound.

  * VPC automatically comes with a default ACL which default ALLOW ALL

  * Custom network ACLs DENY ALL by default

  * Each subnet must be associated with an ACL (default ACL by default)

  * Can associate a single network ACL with multiple subnets. Subnets associated with a new ACL will disassociate automatically with it's old ACL.

  * A network ACL contains a numbered list of rules that is evaluated in order, starting with the lowest numbered rule

  * A network ACL has separate inbound and outbound rules, and each rule can either allow or deny traffic

  * Network ACLs are stateless; responses allow inbound traffic are still subject to the rules for outbound traffic (and vice versa).

  * If we want to block traffic from a specific IP, we must use network ACLs as security groups do not provide this feature.

  * Have at least 2 public subnet and 2 private subnets in different availability zones for resiliency. With ELB's, make sure they are in 2 public subnets in 2 different AZs.

  * To make bastion hosts resilient, put them behind an ASG with a minimum size of 2. Use Route53 (either round robin or using a health check) to automatically fail over.

  * To make NAT instances resilient, you need two in different AZs with a script to failover. But instead, use a NAT Gateway.

  * You can monitor traffic within your custom VPC's using VPC Flow Logs.

# Useful Resources:

  * Answers for the free AWS-provided sample exam: http://blog.ambadkar.org/2015/12/12/solutions-to-aws-certified-sysops-administrator-associate-level-sample-exam-questions/

  * Useful set of practice questions for self evaluation: https://www.udemy.com/aws-certified-sysops-administrator-associate-practice-exams
