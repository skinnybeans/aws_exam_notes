---
title: "AWS DevOps Professional"
date: 2018-08-25
draft: false
menu:
  main: {}
---

# Overview
The AWS DevOps Engineer - Professional exam is intended for individuals who perform a DevOps role.

It's 80 questions over 170 minutes. ~ 2 minutes per question.

## Content
The domains of study are:

  * <a href="#1-continuous-delivery-and-process-automation">1. Continuous Delivery and Process Automation (55%)</a>
  * <a href="#2-monitoring-metrics-and-logging">2. Monitoring, Metrics, and Logging (20%)</a>
  * <a href="#3-security-governance-and-validation">3. Security, Governance, and Validation (10%)</a>
  * <a href="#4-high-availability-and-elasticity">4. High Availability and Elasticity (15%)</a>

## Exam Study Tips

### Service Knowledge

#### Elastic Beanstalk

 * How to configuration EB with `eb-init`
 * Getting Started on AWS Documentation
 * Remember that `eb-start` is not the same as `eb-init`
 * Know how Blue/Green Deployments work on Elastic Beanstalk
 * Remember the structure and syntax of `ebextensions`

#### OpsWorks

 * Read through the best practices guide
 * Understand the rolling deployment feature
 * Read how to install and update custom Cookbooks with Chef

#### CloudFormation

 * Have a solid working understanding of CloudFormation
 * Understand `AWS::CloudFormation::Init`
 * Read up on `WaitCondition` and `WaitConditionHandle`

#### AutoScaling

 * Have an in depth understanding of the ASG Lifecycle and Lifecycle Hooks
 * Understand the different ASG processes and what suspending them implies for ASG functionality

#### CloudFront / Route 53
 * Understand the interaction between deployment types (e.g. A/B testing) and services such as CloudFront and Route53

#### Data Pipeline / CloudSearch

 * Just know what these do at a really high level

### General AWS Best Practices Knowledge

 * Read AWS Operations & Deployment Whitepapers:
    * http://media.amazonwebservices.com/AWS_Development_Test_Environments.pdf
    * http://media.amazonwebservices.com/AWS_Operational_Checklists.pdf
    * https://aws.amazon.com/whitepapers/backup-and-recovery-using-aws/

 * Read Architecture Whitepapers:
    * https://media.amazonwebservices.com/AWS_Cloud_Best_Practices.pdf
    * http://media.amazonwebservices.com/AWS_Building_Fault_Tolerant_Applications.pdf
    * https://aws.amazon.com/whitepapers/storage-options-aws-cloud/

 * Read the Security Whitepapers: 
    * https://aws.amazon.com/whitepapers/aws-security-best-practices/
    * https://aws.amazon.com/whitepapers/security-at-scale-governance-in-aws/
    * https://aws.amazon.com/blogs/security/new-whitepaper-security-at-scale-logging-in-aws/
    * https://aws.amazon.com/whitepapers/encrypting-data-at-rest/

## Suggested Additional Resources

 * Use WhizLabs for Study Questions

 * Practical/Hand-ons advice:
    - Write CloudFormation to deploy a HA WordPress Instance
    - Write a CloudFormation template to a deploy an auto-scaled application and use a load testing tool to test the scaling
    - Write a small lambda function and use it as the backing resource for a cloudformation custom resource
    - Become familar with different CloudFormation template impacts: REPLACE, UPDATE, and INTERRUPT
    - Download an Elastic Beanstalk example application, make changes, create DEV and PROD EB environments, make changes, observe updates, experiment with deployment types
    - Spin up an example OpsWorks stack and make changes to the underlying custom chef recipes 
    - Deploy 2 instances, with appropriate roles, bootstrap the CloudWatch logs agent and configure detailed log ingestion into CloudWatch
 
 * Read the CloudWatch Service Developer Guide

# 1. Continuous Delivery and Process Automation

## Study Areas

1.1 Demonstrate an understanding of application lifecycle management:

1.2 Demonstrate an understanding of infrastructure configuration and automation.

1.3 Implement and manage continuous delivery processes using AWS services.

1.4 Develop and manage scripts and tools to automate operational tasks using the AWS SDKs, CLI, and APIs.

## Core Concepts

  * CI/CD Concepts

  * Deployment types:

    * Single Target Deployment: 
      - Simple, small, legacy, non-HA deployment
      - Incurs brief outage, limited rollback, testing opportunity limited as can only test post-prod-deploy

    * All-at-Once Deployment: 
      - Deployment happens in one-step, destination is multiple targets, requires orchestration tooling
      - No ability to test, less than ideal rollback, incurs outage

    * Minimum in-service Deployment:
      - Deployment happens in multiple stages, orchestration and health checks required, allows for automated testing, generally no downtime
      - Often quicker than rolling deployments
      - High availability baseline established (perhaps one target per availability zone)
      - Stage 1 of the deployment is taking all targets minus the baseline and initiating a deployment
      - After a health check following Stage 1 deployment, Stage 2 is initiated to updating the remaining targets

    * Rolling Deployment:
      - Deployment happens in multiple stages with the number of targets per stage is user-defined
      - Orchestration and health checks are required
      - Least efficient deployment time
      - Overall applicable health isn't necessarily maintained
      - Generally no downtime assuming number of targets per run isn't large enough to impact application
      - Can be paused to allow for multi-version testing

    * Blue/Green Deployment:
      - Enabled by the public cloud
      - Requires advanced orchestration tooling
      - Carries significant cost (maintaining two environments per deployment - less important with hourly billing)
      - Cutover and migration is clean and controlled via a DNS change
      - Rollback is equally clean and controlled by a DNS change
      - Using advanced template systems like CloudFormation, entire process can be fully automated
      - Highest risk mitigation of all deployment strategies

    * Summary:
      - Know the pros and cons of each deployment type
      - Know when each should be used and when not
      - Know the limitations of each, how quick is deployment, how quick is rollback
      - Know how each deployment type impacts your applications
      - Know which AWS Services support which deployment type

  * A/B Testing

    * Different to Blue/Green Deployment as the cutover is gradual in that you send a percentage of traffic to the new environment and monitor operational health as you slowly increase the traffic levels. As confidence in the change increases, you increase traffic until the entire user base it consuming the change.
  
    * Why A/B Test?
      - Separates different version of your code, which may have different levels reliability or performance
      - Allows gradual performance/stability/health analysis
      - Allows new features to be tested with a subset of users
      - If bugs are detected, rollback can be enacted quickly and the damage caused by the failed update is minimised
      - End-goal isn't always to migrate to a new environment, just exists to test changes

    * How to A/B Test in AWS:
      - Generally Route53 is used
      - Two records are created, one pointing at A, the other point at B.
      - Traditionally, this is called round-robin with 50/50 request distribution
      - Weighted-Round-Robin is an advanced feature where you can specify a weight for each environment
      - This allows a granular and adjustable distribution such as 100%/0%, 30%/70%, 50%/50% or anything in between
      - IMPORTANT: Based on DNS, caching and other DNS related issues can impact the overall accuracy of this technique

  * Bootstrapping:
    
    * Process during which you start with a base image (ISO/AMI) and via automation you build on it to create a more complex object in the environment

    * This process can be achieved with `cloud-init` (as part of User Data) or `cfn-init` as part of CloudFormation.

    * With an AMI-based approach to deployment, you would need to generate an AMI for each individual variant of system configuration (e.g. application versions, OS Type, etc). With a Bootstrapping approach, you would only need to maintain a BASE AMI for your application OS configuration, and then have a parameterised bootstrap script which would identify patches, server configuration, and versioned software artifacts to apply to the OS in order to generate an instance.

    * Why use AMIs over Bootstrapping?
      - AMIs offer quick start capabilities, but offer less runtime configuration
      - Bootstrap Configuration often takes longer to apply, but is more configurable
      - Overall, there are implications for deployment time. If you need to rapidly scale or get infrastructure up and running quickly, AMIs are generally a better approach.

  * Immutable Architecture:

    * Immutable architecture is the practice of replacing infrastructure instead of upgrading or repairing faulty components...

    * Cattle over pets

    * An AWS Perspective:
      - Treat servers as unchangeable objects
      - Don't diagnose and fix, throw away and re-create
      - Nothing is bootstrapped, nothing is changed, an image for every version, pre-created, ready to go

    * Principles of Immutability:
      - Traditional architecture treats servers as precious - if they develop a problem, you diagnose, fix and return to service (require a team to perform this diagosis and remediation!)
      - Immutable architecture treats servers as throwaway objects, if a failure is detected you remove the server, create a new one from an AMI, and bring it into service.

  * Virtualisation vs Containerisation

    * Traditional VMs contains an entire OS replica putting much more stress on the underlying system to run a workload (Guest OS)

    * Containers do not contain the Guest OS per application, and can in fact share logical layers of with other containers allowing for higher density

    * Containers are generally faster to start than VMs

    * Containerisation Benefits:
      - Escape dependency hell: efficient packaging of application with all it's explicit dependencies (FULLY RESOLVED ARTIFACT)
      - Consistent progression of the container image artifact between stages of development, test, qa, production
      - Isolation of applications running in containers of a given host OS (performance or stability issues with App A should not impact App B)
      - Resource scheduling at the micro level (can be more explicit about the resources required to run your application for efficient density)
      - Extreme portability of code between systems and platforms
      - Enables a microservice architecture

    * Docker Components:
      - Docker Image - artifact to produce a container
      - Docker Container - instance of an image
      - Layers / Union File Systems - union file systems allow multiple branches of a file system to be overlayed to create a single file system for a container. These layers make Docker lightweight and fast.
      - Dockerfile - series of instructions to build an image
      - Docker Daemon / Engine
      - Docker Client
      - Docker Registries / Docker Hub

## CI/CD/Automation - AWS Practicals

### CloudFormation

  * CloudFormation Primer:
    - CF is a building block service designed to provision infrastructure within AWS
    - Structured in JSON/YAML
    - Overall terms to understand: Stack (grouping of infra), Template (declarative instructions), Stack Policy (IAM style policy statement which governs what can be changed and by who)
    - Template Anatomy: Parameters, Mappings, Resources (MANDATORY), Outputs, Conditions
    - Why? Repeatable deployments. Quicker than manual provisioning. Enables immutable infrastructure and advanced deployment types.

  * Stack Creation 
    - Template Upload / S3 Template Reference
    - Template Syntax Check
    - Stack Name and Parameter Verification / Ingestion
    - Cloud Formation Template Processing and Stack Creation
    - Resource Orderinig
    - Resource Creation
    - Output Generation
    - Stack Creation or Rollback

  * DependsOn
    - Allows you to direct CloudFormation to handle more complex dependencies (e.g. EC2 instance needs to wait for a NAT Gateway to be provisioned)

  * Resource Deletion Policies
    - Delete (Default): Good for short lifecycle or immutable environments
    - Retain: non-immutable infrastructure, orphaned resources incur costs
    - Snapshot: Supports EC2 Volumes, RDS Instances/Clusters, Redshift Clusters where resources are snapshotted before deletion

  * Stack Policies
    - When updating stacks the first thing that happens is the stack policy is checked
    - IMPORTANT: Once a stack policy is defined is cannot be deleted and all objects are protected with an explicit DENY by default
    - Stack Policies scope statements by resources which represent the "logical identifiers" of those resources in the stack
    - Stack Policies have the following actions: `Update:Modify`, `Update:Replace`, and `Update:Delete` (or `Update:*`)
    - The `Principal` field must be `"*"` in a stack policy
    - Conditions in the stack policy allow you to apply statements to specific resource types (e.g. `AWS::EC2::Instance`)

  * Stack Updates
    - Updates can impact resources in the following ways: `No Interruption`, `Some interruption`, `Replacement`, or `Delete`

  * Nested Stacks
    - Nested stacks can have nested stacks

  * CloudFormation Creation Policies, Wait Conditions, and Handlers
    - DependsOn just waits for the AWS resources to be created
  
    - `CreationPolicies` can only be used on EC2 instances and ASGs currently
      - TWO components in CreationPolicies: the policy definition and the signal configuration
      - The policy defines the count and timeout for the resource signals that should be fired on instance startup
      - The signal configuration leverages `cfn-signal` to notify CloudFormation of a success instance start up in the ASG
 
    - `Wait Conditions` and `Wait Condition Handlers` can be used in much more complex scenarios involving complex interaction between resources
      - A Wait Condition Handle is a CloudFormation resource with no properties that generates a signed URL which can be used to communicate SUCCESS or FAILURE
      - Wait Conditions generally have four components:
        1. They DependOn resource/s you are waiting on
        2. A handle property references the above handle
        3. They have a response timeout
        4. THey have a "count" - if none is specified the default is 1
      - Example Wait Condition Handle:
          WaitHandle:
            Type: AWS::CloudFormation::WaitConditionHandle
      - Example of a Wait Condition:
          WaitCondition:
            Type: AWS::CloudFormation::WaitCondition
            DependsOn: MyEc2Instance
            Properties:
              Handle: !Ref WaitHandle
              Timeout: 3600
              Count: 1
      - Use of the wait handle could be in the EC2 User Data or in a Lambda-backed custom resource (`!Ref WaitHandle` gets you the `Signed URL`)
      - JSON Data can be passed to the Signed URL and referenced in the `!GetAtt` function

    - How are they different?
      - Creation Policies only valid for EC2 instances within an ASG
      - Wait Conditions and Wait Condition Handlers are useful for more complex orchestration
      - You can have many resources depending on a Wait Condition, making it a convenient hold point for the entire template
      - Wait Conditions and Handlers allow for Data to be Passed back to the template!

  * CloudFormation Custom Resources
    - Resource type backed by SNS or Lambda
    - Can be specified with the "Custom::ResourceNameHere" type property
    - The property "ServiceToken" must be the ARN of the SNS topic or Lambda function
    - The topic or lambda is invoked when created, updated, or deleted
    - The event sent to the resource will have the following fields:
      - RequestType (Create, Update, Delete)
      - ResponseURL - Pre-signed URL for app to respond to CloudFormation on completion
      - StackId
      - RequestId
      - ResourceType
      - LogicalResourceId
      - ResourceProperties - Custom properties added to the resource
    - The pre-signed URL exists to respond to CloudFormation, the response payload has the following fields
      - Status (SUCCESS, FAIL)
      - StackId
      - RequestId
      - LogicalResourceId
      - PhysicalResourceId
      - Data - custom response data that can be referenced in CF via !GetAtt

### OpWorks 

  * Sits in the middle of convenience vs control
    - Most convenient: AWS Elastic Beanstalk
    - Slightly less convenient, but more control: OpsWorks
    - Ultimate control, least convenience: CloudFormation, Custom AMIs, EC2, etc.

  * Structure
    - Stack: Grouping of infrastructure (could be apps, or environments)
    - Layer: Shared functionality or architecture which is applied to a group of components (e.g. database, node app, etc.)
    - Instance: units of compute adopting some of their configuration from the stacks and layers they reside within
    - Application: applications deployed onto one or more instances

  * Chef vs Opsworks: OpsWorks is chef in addition to native AWS elements

  * OpsWork Agent = CHEF (handles configuration of machines), OpWorks Automation Engine allows for lifecycle management of various AWS infrastructure components (load balancers, autoscaling, lifecycle events, etc.)

  * OpsWorks and CHEF are declarative desired state engines. You declare WHAT you want to happen, and leave CHEF/OpsWorks to handle the HOW.
  
  * Within CHEF/OpsWorks you have resources (e.g. packages to install, services to control, config files to update)

  * Recipes tell OpsWorks WHAT you want the end result to be.

  * Cookbooks contain recipes and all associated data to support them.

  * Stack Configuration
    - OpsWorks instances NEED to have internet access!!!
    - OpsWorks does not support mixing Windows and Linux instances.
    - OpsWorks support providing a standard SSH key for accessing instances, but also allows per user instance access as well.
    - You can use custom Chef cookbooks when using Chef 12 (not chef 11)
    - You can pass custom JSON configuration into your stack from the OpsWorks console.
    - The VPC and Subnet configuration can not be changed for a stack.
    - Changing the OpsWorks agent version only applies to NEW instances in the stack.
    - The "resources" tab on the stack page allows the registration of existing resources with the stack: (Elastic IPs, EBS Volumes, or RDS Instances)

  * Layer Configuration
    - Auto-healing is an option enabled from the LAYER
    - Custom JSON can be passed into your chef recipes at the layer-level
    - You can also set an "Instance Shutdown Timeout"
    - You can setup network, volume, and security settings (groups, key, roles) for the layer
    - Layer Types: OpsWorks (traditional feature sets via Chef recipes on ECS), ECS, RDS
    - EXAM: AN RDS INSTANCE CAN ONLY BE ASSOCIATED WITH ONE OPSWORKS STACK
    - EXAM: A STACK CLONE OPERATION DOES NOT COPY AN EXISTING RDS INSTANCE
    - EXAM: BLUE/GREEN DEPLOYMENT ON OPSWORKS (IMPORTANT TO KEEP IN MIND RDS COPY LIMITATION ABOVE)

  * Lifecycle Events
    - EXAM: OpsWorks has FIVE "events" which occur during the lifetime of an instance
      1. Setup - event occurs when an instance has finished booting
      2. Configure - triggered when _ANY INSTANCE_ enters/leaves online state, associate/disassociate EIP, attach/detach a load balancer to a layer (generally useful for updates when servers are taken "offline")
      3. Deploy - Occurs when you run the deploy command on an instance
      4. Un-Deploy - Delete an application or explicitly run an "Undeploy"
      5. Shutdown - Runs when an instance is shutdown, but before termination (allows for cleanup)
    - When an event occurs, it runs a set of recipes ASSIGNED to that event
    - EACH LAYER has its OWN recipes for each event
    - Note: Rebooting an instance DOES NOT trigger any lifecycle events

  * OpsWorks Instances
    - Instances can be added in two locations: the layer or the stack instances menu
    - Three types of instances (Scaling Types): 
      - 24/7 (started or stopped manually or controlled via APIs)
      - Time-based (initially provisioned and configured to turn on or off at certain times during the day)
      - Load-based (automatically turn on or off based on configurable criteria)
    - General Instance Configuration
      - Hostname
      - Size (e.g. t2.medium)
      - Subnet
      - Scaling Types (24/7, time-based, load-based)
      - SSH Key (overrides stack/layer settings)
      - Specific OS
      - Root Device Type (EBS-based or Instance Store)
      - Volume Type (SSD, Magnetic, etc.)
    - Time-based Scaling Configuration
      - Per app server
      - Can block out times for daily or specific days during the week
    - Load-based Scaling Configuration
      - PRIOR to setting up a load-based instance, you need to setup PER-LAYER scaling configuration
      - Simple Scaling = based on thresholds of CPU/MEMORY/LOAD + parameters (server start batches, threshold exceeded time, and ignore metrics time... normally refered to as "cooldown")
      - Complex Scaling = CLOUDWATCH ALARMS

  * OpsWorks Applications
    - Configuration includes:
      - Application name
      - Document root
      - Data Source (RDS / None)
      - Application Source (GIT / HTTP / S3)
      - Environment Variables
      - Domain Names
      - SSL Enable & Settings
    - Deployment Process:
      - Executes the deploy recipes on the instances targetted by the command
      - Passed to the command in the application-id
      - Application Parameters are passed into the chef environment with "Databags" (these DataBags are JSON objects containing data)
      - Deploy recipe accesses the application source information from the DataBag and pulls the application payload onto the instance
      - EXAM: OpsWorks maintains 5 version of the app (1 current, 4 historic) to allow for rollbacks

  * OpsWork "create-deployment" command
    - Two main functions: creates app deployments & allows stack-level commands to be executed against the stack
    - EXAM: this command is misleading as it is not just for app deployments it is for general stack-level configuration
    - Syntax:
      - "--region"
      - "--stack-id"
      - "--app-id"
      - "--instance-ids"
      - "--comment"
      - "--custom-json"
      - "--generate-cli-skeleton" & "--cli-input-json" allows for automation of this command usage
      - "--command" - available commands are install_dependencies, update_dependencies, update_custom_cookbooks, execute_recipes, configure, setup, deploy, rollback, start, stop, restart, and undeploy. 
    - Sample command to deploy an app:
      ```
      aws opsworks --region eu-west-1 create-deployment \
        --stack-id 1111-1111-1111 \
        --app-id 1111-1111-1111 \
        --command "{\"Name\": \"deploy\"}"
      ```
    - Sample command to "undeploy" an app, that is, remove the application and it's dependencies:
      ```
      aws opsworks --region eu-west-1 create-deployment \
        --stack-id 1111-1111-1111 \
        --app-id 1111-1111-1111 \
        --command "{\"Name\": \"undeploy\"}"
      ```
    - Sample command to "rollback" an app, that is, go back to four historic deployments:
      ```
      aws opsworks --region eu-west-1 create-deployment \
        --stack-id 1111-1111-1111 \
        --app-id 1111-1111-1111 \
        --command "{\"Name\": \"rollback\"}"
      ```
    - Other commands:
      - update_custom_cookbooks: causes all instances to perform a full re-download of their custom cookbooks from the repository
      - execute_recipes: allows the manual spawn of an individual recipe, useful for diagnostics and debugging
      - setup: runs the instances setup recipes
      - configure: runs the instances configure recipes (useful to manually force service discovery)
      - update_dependencies: only valid in CHEF 11 and Linux, forces updates to security packages on underlying OS
      - upgrade_operating_system: full upgrade of the RHEL or Amazon Linux OS
      - EXAM: UNDERSTAND THESE COMMANDS AND WHAT THEY DO

  * BerkShelf
    - EXAM: Was added in Chef 1.10
    - EXAM: Allows you to install cookbooks from multiple repositories
    - You need to "Enable Custom Chef Cookbooks" at the stack level
    - You need to create a `Berksfile` in your repository (which specify the dependency cookbooks to integrate)
    - Example `Berksfile`:
      ```
      source "https://supermarket.chef.io"
      cookbook 'apt'
      cookbook 'bleh', git: 'git://somewhere/bleh.git'
      ```

  * DataBags
    - Global are global JSON configuration objects accessible from within the CHEF framework
    - There are multiple databags, including STACK, LAYER, APP, and INSTANCE
    - Data is accessed via the Chef `data_bag_item` and `search` methods within compute assets
    - DataBags are constructed with the `Custom_JSON` field for the OpsWorks components
    - Can contain any JSON supported data types
    - The search method allows access via a search index:
      - `aws_opsworks_app` - App DataBag
      - `aws_opsworks_layer` - Layer DataBag
      - `aws_opsworks_instance` Instance DataBag
      - `aws_opsworks_user` Users Databag, a set of users for a stack
    - To search an applications DataBag, your Chef recipe might look like: `app = search(:aws_opsworks_app).first`
    - To use the data in the application DataBag: `repository app["app_source"]["url"]`
    - EXAM: Understand the concept (they allow access to contextual information within recipes)
    - EXAM: Know that they exist at the Stack, Layer, Instance, and Application level
    - EXAM: Know that DataBags are JSON and are updated/provided in OpsWork context via the `CUSTOM_JSON` field in most settings areas in AWS Console

  * OpsWorks Instance Auto-Healing
    - Each OpsWorks instance has an agent installed
    - The agent performs ongoing heartbeat style health checks
    - If the heartbeat fails, OpsWorks treats the instance as unhealthy and WILL PERFORM AN AUTO-HEAL
    - Workflow for EBS-backed Instances:
      1. Instance Stopped (status goes from ONLINE, to STOPPING, to STOPPED)
      2. Instance is Started (status goes from REQUESTED, PENDING, BOOTING, to ONLINE)
      3. Configure Event is Run for consistency
    - Workflow for Instance-Store Instances (ephemeral storage):
      1. Instance is terminated (status goes from ONLINE to SHUTTING_DOWN)
      2. Root Volume is Deleted (standard in AWS obviously)
      3. OpsWorks launches a new instance (same hostname, config, layer membership)
      4. Re-attaches any EBS Volumes
      5. Assigns a new public and private IP (standard in AWS obviously)
      6. If applicable, re-associates any Elastic IP addresses
      7. Configure Event is Run for consistency
    - EXAM: What ISN't Auto-Healing?
      - Auto-Healing WILL NOT recover instances from serious corruption and may start with a "START_FAILED" error for EBS-backed instances. This will require manual intervention (new instance and full re-provision)
      - Auto-Healing WILL NOT upgrade the OS of instances even if the default OS is changed at a stack level, it will simply recover the instance with the same OS settings as before the healing process.
      - Auto-Healing is not a performance response, it only recovers instances automatically when there is an agent failure

### Elastic Beanstalk

 * Abstract focus on application infrastructure, focusing on components and performance, not configuration and specifications. It attempts to remove, or significantly simply, infrastructure management. It allows applications to be deployed into infrastructure environments quickly and easily.

 * Key Architecture Components:
    - _Applications_ are the high level structure in EB 
      - EB infrastructure can be composed of one single applications or many applications representing logical components
    - Applications run within one or many _Environments_ (i.e. Test/Prod, V1/V2, or Frontend/Backend)
      - There are broadly two types of environments: _Web Environments_ and _Worker Environments_
      - EXAM: Each web environment has it's own URL which can be swapped between environments to help with blue/green deployment
    - _Application Versions_ is the last component, which is a unique package representing an application version represented as an application bundle - `.zip`

 * EB Environment Types:
    - Web environments and Worker environments are integrated in a decoupled manner via an `SQS` queue
    - Worker environments are scaled based on the SQS `Queue Depth`
    - Worker environments do not need to know about the SQS queue, a daemon exists on the worker instances which fetches data from the queue and submits a HTTP request to the worker application.

 * EB Applications are composed of several tiers: 
    - Web:
      - Scaling - single instance/auto-scaling/time-based-scaling on various custom metrics
      - Instances (can use custom AMI)
      - Notifications
      - Software Configuration
      - Updates & Deployments - (EXAM) Deployment types: Rolling (%/# batch), Rolling with Additional Batch (for full availability you can start instances before terminating them), Immutable
      - Health (Basic or Enhanced Reporting - Enhanced gives you a range of custom metrics)
      - Managed Updates
    - Network (Load Balancing)
    - Data (RDS - recommended to use existing, 1-to-1 relationship between RDS instance and environment, cannot be cloned)

 * The Application Source Bundle (.zip):
    - Application Source Code
    - Any Dependencies
    - Scripts
    - .ebextensions

 * Supported Platforms
    - NodeJS
    - PHP
    - Python
    - Ruby
    - Tomcat
    - MS IIS
    - Java
    - GoLang
    - Docker (EXAM: For adding non-supported framework)

 * .ebextensions
    - A configuration folder within an application zip file allowing for configuration of EB environment and resources it contains (EC2, ELB, and others)
    - Files within .ebextensions are YAML formatted, and end with a `.config` extension
    - Structure of configuration files:
      - `option_settings`: allow you to declare global configuration options
      - `resources`: allows you to write CLOUDFORMATION! You can !Ref and !GetAtt any of these resources through the `.config` file templates as well as EB leverages cloudformation for provisioning!'
      - The `container_commands` key to execute commands that affect your application source code. Container commands run after the application and web server have been set up and the application version archive has been extracted, but before the application version is deployed. Non-container commands and other customization operations are performed prior to the application source code being extracted.
      - `packages`, `sources`, `files`, `users`, `groups`, `commands`, `container_commands`, and `service` allow for customisation of the EC2 instances as part of your environment - similar to the cloud formation `AWS::CloudFormation::Init` directive.

 * Leader Instance
    - A leader instance in EB is an EC2 instance witin a load-balanced, auto-scaled environment chosen to be the leader/master
    - EB only uses the leader instance concept during environment creation, once the environment is established all nodes are equal
    - The `leader_only` directive can be used within the container_commands section of a .config file within ebextensions, for example:
    ```
    container_commands:
      name of container_command:
        command: "command to run"
        leader_only: true
    ```
    - `leader_only` directives are useful for running certain commands only once during environment creation (e.g. populate a DB with initial data, perform a migration, register the environment with an external CMDB, etc.)

 * Example .ebextensions config to set up CloudWatch Custom Metrics for Memory:
    ```
    sources:
      /aws-scripts-mon: http://ec2-downloads.s3.amazonaws.com/cloudwatch-samples/CloudWatchMonitoringScripts.zip
    
    commands:
      01-setupcron:
        command: echo "*/5 * * * * root perl /aws-script-mon/mon-put-instance-data.p1 --mem-used > /dev/null" > /etc/cron.d/cwpump
      02-changeperm:
        command: chmod 644 /etc/cron.d/cwpump
    ```

 * Example .ebextensions config to set up users in groups:
    ```
    users:
      - myuser:
        groups:
          - group1
          - group2
        uid: 50
        homedir: "/tmp"
    
    groups:
      - group1: 45
      - group2: 99
    ```

 * Example .ebextensions config to ensure service is running:
    ```
    services:
      sysvinit:
        - myservice:
            enabled: true
            ensureRunning: true
    ```

 * Docker in EB
    - Running Docker in EB allows us to deal with unsupported platforms or applications with specific dependencies
    - Allows for single-container (per EC2 instance) or multi-container (on ECS) deployments
    - There are several generic Docker configurations for popular software stacks such as: Java with Glassfish, Python with uWSGI.
    - EB Supports public and private Docker Image Registries
    - There are roughly two components to a container deployment on EB
      1. A Docker Image from a Registry or an included `Dockerfile` to be built at environment creation time
      2. A `Dockerrun.aws.json` file which specifies how EB should retrieve and run your Docker image (Image, Registry Authentication, Ports to Expose, Volume Mapping, and Log File destinations to forward to S3)
    - For EB to use a provide registry it will need a `.dockercfg` or `config.json` file holding the private registry authentication details. This file should be stored in S3 and referenced in the `Dockerrun.aws.json` file.

# 2. Monitoring, Metrics, and Logging

## Study Areas

2.1 Monitor availability and performance.

2.2 Monitor and manage billing and cost optimization processes.

2.3 Aggregate and analyze infrastructure, OS and application log files.

2.4 Use metrics to drive the scalability and health of infrastructure and applications.

2.5 Analyze data collected from monitoring systems to discern utilization patterns.

2.6 Manage the lifecycle of application and infrastructure logs

2.7 Leverage the AWS SDKs, CLIs and APIs for metrics and logging.

## Monitoring/Metrics/Logging

### CloudWatch

 * CloudWatch is a metric gethering, monitoring, alerting, and visualisation service
 * (EXAM) CloudWatch stores metrics for a maximum time period of 2 WEEKS
 * "Enable Detailed Monitoring" provides further namespace dimensions for detailed analysis
 * (EXAM) Important to know that you can aggregate metrics by ASG in CloudWatch (e.g. understand network throughput for entire ASG)
 * CloudWatch has it's own CLI which prefixes API actions with `mon-*` (e.g. `mon-put-metric-alarm` vs just `put-metric-alarm` via the AWS CLI)
 
 * Custom Metrics:
    - `aws cloudwatch put-metric-data --metric-name CustomMetric --namespace MyCustomNamespace --value 1`
    - CloudWatch aggregates data by 1 minute periods

 * Alarms:
    - Used to initialise actions on your behalf, based on parameters you specify, against metrics you have in use
    - Actions are sent to SNS (for notification) or ASGs (for scaling)
    - Metrics are for _*ONE or FIVE*_ minutes. The alarm period should be equal to or greater than this metric frequency. 
    - Alarms periods are also set in seconds and valid values are 10, 30, 60, or any multiple of 60.
    - Alarms can't invoke actions because they are in a state, the state must change before the alarm is invoked
    - All alarms must be in the same region as the alarm
    - Some AWS Resources don't send metric data to CloudWatch under certian conditions (may report INSUFFICIENT_DATA state)
    - Alarm states: OK, ALARM, INSUFFICIENT_DATA
    - Limits: 5000 alarms per account
    - With the CloudWatch CLI, you can create or update alarms with the `mon-put-metric-alarm` command
    - With the CloudWatch CLI, Enable or disable alarms with the `mon-enable-alarm` or `mon-disable-alarm` commands
    - Describe alarms with the `mon-describe-alarms` command
    - You can create an alarm before you create metrics
    - Example `put-metric-alarm` command: 
      ```
      aws cloudwatch put-metric-alarm \
        --alarm-name TimeAlarm \
        --alarm-description "Alert at a certain time" \
        --metric-name "CurrentUnixTime" \
        --namespace "CustomNamespace" \
        --statistic "Average" \
        --period 60 \
        --threshold 500000 \
        --comparison-operator GreaterThanOrEqualToThreshold \
        --evaluation-periods 1 \
        --alarm-actions "arn:aws:sns:ap-southeast-2:123456789101:MySnsTopic"
      ```
    - CloudWatch allows you to scale your ASGs in and out based on metrics using alarms. For example, the command below will execute the "my-scaleout-policy" scaling policy given Average CPU Util for the ASG "my-asg" exceeds 80% for two consecutive periods of 60 seconds:
      ```
      aws cloudwatch put-metric-alarm \
        --alarm-name AddCapacity \
        --metric-name CPUUtlization \
        --namespace AWS/EC2 \
        --statistic Average \
        --period 60 \
        --threshold 80 \
        --comparison-operator GreaterThanOrEqualToThreshold \
        --dimensions "Name=AutoScalingGroupName,Value=my-asg" \
        --evaluation-periods 2 \
        --alarm-actions arn:aws:autoscaling:ap-southeast-2:123456789101:scalingPolicy:blah-blah-blah:autoScalingGroupName/my-asg:policyName/my-scaleout-policy
      ```

 * Logs:
    - Monitor your existing system, application, and custom logs in real time
    - Send your existing logs to CloudWatch
    - Create patterns to look for in your logs
    - Alert based on the findings of these patterns
    - CloudWatch Logs includes a free agent
    - It also logs AWS CloudTrail events
    - Terminology
      - _Log Events:_ a log record with a message and timestamp
      - _Log Streams:_ sequence of log events that share the same source
      - _Log Groups:_ groups of log streams that share retention, monitoring, and access control settings.
      - _Metric Filters:_ used to define how a service would extract metric observations from events and turn them into data points for a CloudWatch metric.
      - _Retention Settings:_ how long log events are kept in CloudWatch log groups (between 1 day and 10 years)

 * CloudWatch Log Filters:
    - Metric Filters define search patterns to look for in a log
    - Filters can be used to turn logs into metrics
    - Filters WILL NOT work on existing data, only on data pushed to CloudWatch AFTER the filter was created
    - CloudWatch Log Filters will only return the first 50 results
    - Components of a metric filter: pattern, metric name, metric namespace, metric value

 * Real-Time Log Processing:
    - Real-Time processing enabled via CLoudWatch Subscriptions
    - Example subscription configuration:
      ```
      aws logs put-subscription-filter \
        --log-group-name "Syslog" \
        --filter-name "SomeLogs" \
        --filter-pattern "Invalid user" \
        --destination-arn "arn:aws:kinesis:ap-southeast-2:123456789101:stream/someKinesisStream" \
        --role-arn "arn:aws:iam::123456789101:role/CloudWatchToKinesisRole"
      ```
 
 * CloudTrail 
    - Service which records all of the AWS API calls in an account
    - Log records contain:
      - Identity of who made the API call
      - The time of the call
      - The Source IP of the call
      - The request parameters
      - The response elements returned by the AWS service
    - Purpose: enable security analysis, track changes, audit records, and compliance
    - Two types: ALL REGIONS, ONE REGION
    - Storage: stored in S3 using Server-side-encryption
    - Logs are delivered within 15 minutes of the API call being made
    - New log files are published every 5 minutes
    - When creating cloud trails, you can enable "Log File Validation" which includes a SHA256 hash for every log to make it computationally infesible to alter the log messages. The AWS CLI can be used to verify the integrity of the log
    - When creating cloud trails, you can also set up an SNS Topic to notify of every new log file delivery

 * CloudWatch Events
    - Similar to CloudTrail, but FASTER
    - Near real-time stram of events that describe changes to your AWS resources, these can be routed to other services
    - Events are created in three ways:
      1. State Change - when AWS resources changes state, such as an EC2 instance moving from a pending to a running state.
      2. API call - When an API call is made that is delivered to CloudWatch events via CloudTrail
      3. Your own code - When your own code generates an application-level event which you publish for processing
    - Events have _Rules_ associated with them:
      - These rules match incoming events and route them to one or more targets for processing
      - They're not ordered, and all rules that match an event will be processed
      - Rules can customise the JSON that flows to the target (only including certain attributes or sending plaintext values)
    - Events can have _*multiple*_ targets. Supported targets are:
      - Lambda Functions
      - Kinesis Streams
      - SNS Topics
      - SQS Queues
      - ECS Tasks
      - EC2 instances
      - SSM Run Command
      - SSM Automation
      - AWS Batch Jobs
      - Step Functions
      - Pipelines in CodePipeline
      - CodeBuild Projects
      - Inspector Assessments
      - Built-in targets: EC2 `CreateSnapshot`, `RebootInstances`, `StopInstances`, and `TerminateInstances`
      - CloudWatch Event Bus for other Accounts

# 3. Security, Governance, and Validation

3.1 Implement and manage Identity and Access Management and security controls.

3.2 Implement and manage protection for data in-flight and at rest.

3.3 Implement, automate and validate cost controls for AWS resources.

3.4 Implement and manage automated network security and auditing.

3.5 Apply the appropriate AWS account and billing set-up options based on business requirements.

3.6 Implement and manage AWS resource auditing and validation.

3.7 Use AWS services to implement IT governance policies.

## Notes

### Delegation & Federation
Read AWS Overview Documentation.

### Corporate Identity Federation
Read AWS Overview Documentation.

### Web Identity Federation
Read AWS Overview Documentation.

# 4. High Availability and Elasticity

4.1 Determine appropriate use of multi- Availability Zone versus multi-region architectures.

4.2 Implement self-healing application architectures.

4.3 Implement the most appropriate front-end scaling architecture.

4.4 Implement the most appropriate middle-tier scaling architecture.

4.5 Implement the most appropriate data storage scaling architecture.

4.6 Demonstrate an understanding of when to appropriately apply vertical and horizontal scaling concepts.

## Notes

### AutoScaling Lifecycle

 * Refers to the lifecycle of an EC2 instance within an AutoScalingGroup

 * Lifecycle:
    - "Pending": On instance starting (lifecycle hooks: "Pending: Wait" and "Pending: Proceed")
    - "InService": On instance started
    - "Terminating": On health check failure or scale in (lifecycle hooks: "Terminating: Wait" and "Terminating: Proceed")
    - "EnteringStandby": On entering standby (for debugging or something)
    - "Standby": When entered standby (transitioning out of standby triggers the pending lifecycle event again)
    - "Detaching": On being detached from an autoscaling group
    - "Detached": On finally detached from the ASG
    - "Terminated": When terminated

 * Lifecycle Hooks:
    1. AutoScaling responds to a scale out event by launching an instance
    2. AutoScaling puts the instance into the "Pending:Wait" state
    3. AutoScaling sends a message to the notification target defined for the hook, along with information and a token
    4. AutoScaling Waits until you tell it to continue or the timeout ends (by default this is 1 hour before it changes the state to "Pending:Proceed", from there it will enter the "InService" state)

 * Timeouts can be extended by sending a heartbeat or increasing the default timeout (`heartbeat-timeout` option on CLI).

 * You can call the `CompleteLifecycleAction` API command to tell AutoScaling to proceed

 * You can call the `RecordLifecycleActionHeartbeat` API command to add more time to the timeout

 * 48 HOURS is the MAXIMUM timeout time

 * Cooldowns:
    - Cooldowns help ensure the ASG doesn't launch/terminate more instances than needed
    - Cooldowns start when an instance enters the InService state
    - If an instance is left in the "Pending:Wait" state, AutoScaling WILL WAIT before adding any additional servers.

 * ABANDON or CONTINUE:
    - At the conclusion of a lifecycle hook, an instance can result either of these states
    - ABANDON will cause AutoScaling to terminate the instance and launch a new one if necessary
    - CONTINUE will put the instance into service

 * Spot Instances and Lifecycle Hooks
    - You CAN use Lifecycle hooks with spot instances
    - This does NOT prevent an instance from terminating due to a change in the spot price
    - When a spot instance terminates, you must still complete the lifecycle action 

### Launch Configurations

 * Template used by AutoScaling to launch EC2 instances
 * Defines:
    - AMI
    - Instance Type
    - Key Pair
    - Security Group
    - Block Device Mapping
 * Launch configurations are immutable (must always create a new version)
 * One-to-one relationship between Launch Configuration and ASG

### AutoScaling Groups

 * Contains a collection of EC2 instances that have similar characteristics
 * Performs periodic health checks
 * Scaling policies influence the horizontal scaling of the instances within the group
 * You can create an AutoScalingGroup from an EC2 instance (this automatically generates a launch configuration)

### AutoScaling Self Healing

 * Used for creating low cost, self-healing, immutable infrastructure
 * Keep servers running and highly available without user interaction

### Amazon RDS

 * Supports six engines: MySQL, MariaDB, SQL Server, PostgreSQL, Oracle, Aurora
 * Managed administration
    - Provision Infra
    - Install database software
    - Automatic Backups
    - Automatic Patching
    - Data Replication
    - Automatic Failover
 * Your responsible for:
    - Settings
    - Schema
    - Performance tuning
 * RDS can be vertically scaled (changing the instance type)
    - You are able to use Reserved Instances for RDS
    - You can scale your storage online (5GB to 6TB of GP SSD storage)
    - Microsoft SQL Server will NOT let you scale your storage however
 * RDS can also be horizontally scaled
    - Read replicas (great for high read/write ratios)
    - You can also shard your database (spit tables into multiple databases when they aren't joined by queries)
 * Storage:
    - General Purpose SSD Storage: 3 IOPS/GB, burstable up to 3,000 IOPS
    - Provisioned IOPS (SSD) should be used for I/O intensive transactional workloads
    - IOPS available varies by database engine

### Amazon Aurora

 * Fast, reliable, simple, cost effective
 * 5x throughput of MySQL on same hardware
 * Compatible with MySQL 5.6
 * Storage is fault tolerant and self healing
 * Disk failures are repaired in background without affecting availability
 * Detects crashes and restarts
 * No crash recovery or cache rebuilding
 * Automatic failover to one of up to 15 read replicas
 * Automatic Storage AutoScaling from 10GB to 64TB

 * Backups
    - Automatic continuous incremental backups
    - Point-in-time-restore within a second
    - UP TO 35 DAYS RENTENTION PERIOD
    - Stored in S3
    - No impact on database performance

 * Snapshots
    - User-initiated snapshots are stored in S3
    - Kept until you explicitly delete them
    - Incremental

 * Database Failures
    - Maintains 6 copies of your data
    - 3 availability zones
    - Recovery takes place in a healthy AZ
    - PIT / Snapshot Restore automatically

 * Fault Tolerance
    - Data divided into 10GB segments across many disks
    - Can lose up to 2 copies of data without affecting write
    - Can lose up to 3 copies of your data without affecting read
    - All storage is self healing

 * Replicas
    - Amazon Aurora Replicas share the same volume as the primary instances
    - You can have up to 15 Aurora replicas
    - Replicas incur a low performance penalty on the primary
    - A replica can be failover target with no data loss
    - You can also use up to 5 MYSQL READ REPLICAS
    - MySQL Read Replicas incur a high performance impact of the primary
    - If you failover over to a MySQL read replica you may lose minutes of data

 * Security
    - All instances must be created in a VPC
    - SSL (AES-256) used to secure data in transit
    - You can encrypt databases using KMS
    - YOU CAN NOT ENCRYPT AN EXISTING UNENCRYPTED DATABASE

### DynamoDB

 * Fully managed NoSQL Database service
 * Predictable fully managed performance with seamless scalability
 * Fully resilient and highly available
 * Performance scales linearly
 * Fully integrated into IAM
 
 * EVENTUALLY CONSISTENT (unless STRONGLY CONSISTENT READS enabled at an extra cost)
 
 * On each table you can specify performance requirements:
    - Write Capacity Units - Number of 1KB blocks per second
    - Read Capacity Units - Number of 4KB blocks per second
    - Read queries are rounded to nearest 4KB, so this would be 1 RCU for any single query operation

 * Tables are the main storage primitive in DynamoDB

 * Items are effectively rows within the table

 * Attributes are flexible and dynamic (i.e. no table-level schema exists and not every row with have every attribute)

 * DynamoDB allows for full table scanning or key queries

 * Primary Keys
    - This is a special type of attribute that uniquely identifies each item in the table (must be scalar / single-valued)
    - PARTITION/HASH KEYS are inputs into the internal hash function which determines the physical storage the item will be stored on
    - SORT/RANGE KEYS are an additional key value which creates a composite primary key (i.e. duplicate partition keys are allowed but not if they share the same sort key). Items with the same partition key are stored together in sorted order as defined by the sort key (this makes retrieval of related data faster).

 * Partitions
    - Underlying processing and storage nodes for DynamoDB
    - 1 Table initially creates 1 Partition
    - 1 Partition = 10GB of data
    - 1 Partition handles 3000 RCU and 1000 WCU
    - When WCU, RCU, or storage exceeds aforementioned thresholds a new partition is added and data is spread between them over time based on the primary key
    - Allocated WCU and RCU influences the number of partitions created
    - Structure of your partition and sort keys very important for performance as partitions are limited to certain WCU/RCU
    - YOU CAN NOT DECREASE YOUR PARTITIONS! You can decrease your WCU and RCU, however these will simply be allocated evenly to each partition and ultimately reduce the per-partition WCU and RCU such that performance issues will manifest.

 * Secondary indexes allow you to query your table with an alternative key

 * "Attribute Projection" refers to the process of _asynchronously_ copying attributes related to a GSI or LSI to the table index

 * Global Secondary Indexes (GSI)
    - GSIs are an entirely additional PARTITION and SORT keys for the table
    - You can add up to 5 GSIs
    - Can be created at ANY TIME
    - As with an LSI, you can choose (`Keys Only`, `Include` for specific attributes, or `ALL` attributes)
    - RCU and WCU are defined on the GSI itself, distinct from the table
    - Only supports _eventually consistent_ reads

 * Local Secondary Indexes (LSI)
    - LSIs are essentially an additional sort key for an existing partition key
    - Can only be added at the time of table creation
    - You can add up to 5 LSIs
    - LSIs share the WCU and RCU with the table, so it's important to consider their performance impact
    - By default, non-key attributes are not added to the LSI
    - If you query an attribute which is NOT projected, you are charged the entire ITEM cost of pulling it from the main table. This can have a major performance and cost impact.
    - An `ITEM COLLECTION` is any group of items (along with it's secondary indexes) that have the same partition key value.
    - (EXAM) `ItemCollectionSizeLimitExceededException` is thrown when an item collection exceeds 10GB. This is due to the partition storage limit of 10GB.

 * Streams & Replication
    - Ordered recordd of updates to a DynamoDB table
    - When enabled, it records changes to a table and stores those values for 24 hours
    - AWS guarantee that each change to a DynamoDB table occurs in the stream ONLY ONCE
    - ALL changes to the Table occur in the stream in near real time
    - Available _DynamoDB Stream Views_:
        1. `KEYS_ONLY`: only key attributes are written to the stream
        2. `NEW_IMAGE`: the entire item, as it appears after it was modified
        3. `OLD_IMAGE`: the entire item, as it appeared before it was modified
        4. `NEW_AND_OLD_IMAGES`: the pre and post operation state of the item is written to the stream, allowing for complex comparison operations

 * Performance
    - Partitions, partitions, partitions!
    - Number of Partitions for *Performance* = Desired RCU / 3000 RCU + Desired WCU / 1000 WCU
    - Number of Partitions for *Capacity* = Data Size in GB / 10GB
    - Select the maximum of the performance and capacity calculations above to determine correct number of partitions
    - Allocated reads and writes are distributed across partitions, so ensure this is adequate per partition and be aware of "hot keys" that might be consuming a lot of read and write units.
    - (EXAM) Ensure you understand how to select an appropriate key for the DB workload:
        1. Attribute should have many DISTINCT VALUES
        2. Attribute should have a UNIFORM READ/WRITE PATTERN OVER TIME
        3. Consider creating a synthetic key by combining randomly generated and attribute values if the above is not possible to ensure good partition distribution for the data
        4. You should not mix HOT and COLD key values within a table as this will lead to performance bottlenecks
    - GSIs receive 300 seconds of RCU/WCU BURST
    - (EXAM) Improving read performance should not rely on increasing RCU as this will provison more parititons
    - (EXAM) Improving read performance in DynamoDB should use external cache to reduce RCU required

### SQS
 * 256kb messages
 * For larger messages, the SQS Extended Client Library uses S3 to store larger payloads
 * Ensures delivery of message at least one
 * Not guaranteed FIFO unless explicitly enabled
 * Messages can be retained for 14 days
 * Long polling reduces API calls required which reduces costs (waits up to 20 seconds for messages per call)
 * $0.50 for every 1 million requests
 * SQS Architectures:
    - Priority Messages (simply use two queues: a "High Priority" queue and a "Low Priority" queue)
    - Fanout (SNS --> Multiple SQS Queues --> Different Processing Applications)

### Kinesis
 * Streams
    - Collect and process large streams of data in real time
    - Scenarios (fast log and data feed intake and processing, real-time metrics and reporting, real-time data analytics, complex stream processing)
    - Parallel application readers

 * Kinesis Analytics (Not examinable)
 
 * Firehose
    - Streaming data can be pushed straight into S3, Redshift, or both
    - Supports compression
    - Supports automatic encryption
    - Control delivery by batch size (up to 1MB) or interval (up to 60 seconds)

 * Kinesis VS SQS
    - Use Kinesis if you need to fan out to multiple consumers
    - You can replay data in Kinesis for up to 24 hours
    - SQS has a 14 day retention period

## EC2 Instance Types

 * Para-Virtualisation (PV) vs Hardware Virtualisation (HVM):
    - PV used to be primary form of virtualisation in AWS
    - PV requires instance OS and driver modifications, as the host OS presents an API which the guest OS needs to support
    - PV originally offered better performance than HVM (hardware virtualisation), as HVM incurred a performance penalty for virtualising all hardware interfaces adding essentially an extra processing step to every hardware instruction.
    - Certain AMIs and instance types only support either PV or HVM

 * Instance Types:
    - Neumonic "Dr McGif Pics" or the scottish duck... D.R..M.C.G.I.F.T..P.X.
    - D ... Density
    - R ... Memory Optimised (think R for RAM)
    - M ... General Purpose (think "main choice")
    - C ... Compute Optimised
    - G ... Graphics
    - I ... IO Optimised (think IOPS)
    - F ... FPGA (new for hardware modified code)
    - T ... Cheap general purpose (think t2.micro)
    - P ... Graphics (think pics)
    - X ... Extreme Memory

## EBS Performance

 * Storage Fundamentals:
    - Capacity: the amount of data in GB which can be stored on a volume
    - Throughput: the data throughput in MB/s for read/write operations
    - Block Size: The size of each read and write operation, measured in KB
    - IOPS: The number of inpu9t and output operations per second
    - Latency: The delay between read/write request and the completion measured in ms

 * Storage Types:
    - Magnetic: near-archival or cold workloads (~100 IOPS, 2-40ms latency, 10MB max throughput, not burstable)
    - SSD: (base 3 IOPS per GB up to 10,000 IOPS, burstable up to 3,000, 160 MB/s max throughput, assumes 256KB block size)
    - PIOPS: (30 IOPS per GB of storage MAX, maximum 20,000 IOPS, 320 MB/s max throughput, 99.99% performance consistency)

 * Storage performance elements:
    - _Instance Type_: if CPU and memory can not drive volume of data, you will get a bottleneck. Additionally, you have the "EBS-Optimised" flag, which provides an optimised configuration that reduces contention for traffic between your instance and the EBS volume (i.e. Amazon provides dedicated throughput to EBS with options between 500MB/s and 4000MB/s ).
    - _IO Profile and throughput_: depending of the workload you may need a maximum IOPS of 10,000 or 20,000 or throughput of either 160 MB/s or 320 MB/s(difference between SSD and PIOPS)
    - _Network Speed_: The bigger the instance type, the better the network bandwidth which affects throughput to EBS. 

 * `Throughput = Block Size * IOPS` (e.g. 32KB block size * 3000 IOPS = 96MB/s)
    - You can see that even through IOPS has been maximised for a gp2 EBS SSD volume, because of the smaller block size we aren't utilising a proportion of the available throughput. You need to tune the block size and IOPS in order to strike a good balance between IOPS and throughput.

 * For exam:
    - GP2 volumes can achieve 10,000 IOPS and 160 MB/s throughput
    - PIOPs volumes can achieve 20,000 IOPS and 320 MB/s throughput
    - Large EBS Optimised instances can achieve 32,000 IOPS at a 16 KB block size up to a maximum of 500 MB/s throughput
    - The maximum achievable IOPS is 48,000 IOPS at a 16 KB block delivered by the larger 10Gb/s network stacks
    - You can scale out your IO performance by attaching multiple volumes to a high network throughput EBS-optimsed instance in a RAID 0 configuration (where writes are striped - or distributed - over a number of independent volumes).

 * GP2 Burst Pool
    - You can 5.4 Million IOPS credits 
    - Pool is replenished each second by the number of base IOPS (3/GB for GP2)
    - Pool can be spent up to the 3,000 MAX burst rate
    - Pool provides enough additional performance for boot and traditional usage spikes
    - Example: A 500GB volume can burst @ 3,000 IOPS for 60 minutes.

 * Pre-warming *NEW* EBS volumes is no longer required

 * Volumes restored from S3 snapshots are "lazy restored", so reading the full volume to force a restore is recommended
