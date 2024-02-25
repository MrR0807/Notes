# Billing

* Go to *Billing Preferences* and check call preferences, like *Invoice delivery preferences* or *AWS Free Tier alerts*.
* Setup a budget by going to *Budgets* -> *Use a template* -> *Create budget*.

# Creating new accounts

* Gmail + trick. E.g. originalemailname+alias@gmail.com
* Add MFA
* Add Budget
* Enabled IAM User & Role Access to billing (*Account* -> *IAM User and Role Access to Billing Information*)

# IAM Basics

* Account and Account Root User can be thought as the same thing. Account Root User cannot be restricted in any way.
* Every AWS Account comes with its own IAM database. IAM is globally resilient service. So any data is always secure across all AWS regions. AWS Account trusts fully IAM. IAM can do anything in the account. There are restrictions around billing and account closure. Operationally, IAM can do as much as root user.

IAM let's you create three different type of identity objects:
* User - identities which represent humans or applications that need access to your account.
* Group - collection of related users e.g. development team, finance or HR.
* Role - can be used by AWS Services, or for granting external access to your account.
 
Generally, you pick IAM Users when you can identify an individual user or application. Roles tend to get used when the number of things is uncertain. If you want to grant users of external accounts access to say a simple storage service bucket or if you want to allow AWS services themselves to interact on your behalf, you use roles.

IAM Policy - they define, allow or deny access to AWS Services. You can attach them to user, group and roles.

IAM has 3 main jobs:
* It is an Identity Provider or IDP. It let's you create, delete modify identities.
* It also authenticates identities known as security principle.
* Authorizes - allows or denies access to resource.

IAM summary:
* No Cost
* Global service / Global resilience
* Allow or Deny its identities on its AWS account
* No direct control on external accounts or users
* Identity federation and MFA

# Setting up Admin User

You can set AWS Account Alias: IAM -> IAM Dashboard -> AWS Account -> Account Alias (it has to be globally unique).

Users -> Create User -> Set Permissions (attach policies directly AdministratorAccess).

To test login now, copy "Sign-in URL for IAM users in this account" (which was generated using Account Alias)

# IAM Access Keys

IAM Acess Keys are long term credentials used by IAM Users. When you're using UI you will generally user and password, while when you use CLI - Access Keys.

Access Keys are similar to user and password, but there are crucial differences:
* An IAM User has 1 username and 1 password. It cannot have more. Username is public part and password is private part.
* An IAM User can have two access keys (it can have zero, one or two maximum). Access keys can be created, deleted made inactive or active. Just like username and password, access keys are formed from two parts: Access Key ID; Secret Access Key. When you create Access Keys, AWS provides both.

By rotating access key it means to delete the current one and replace with new one. You cannot modify access key secret, only to delete and recreate. That is why you can have up to two access keys.

**Only IAM Users have access keys.** IAM Roles don't use access keys.

Click on top right IAM user -> Security Credentials -> Access keys.

## Configure AWS CLI tool

```
aws configure
```

Will configure default profile. To configure named profile (multiple profiles are supported):

```
aws configure --profile iamadmin-general
```

To use named profile listing S3 buckets:

```
aws s3 ls --profile iamadmin-general
```

# Stateless firewall vs Stateful firewall

Stateless firewal (in AWS it is Network ACL/NACL) does not understand Layer 4 connections. That is why you need to define both outbound and inbound traffic. For example, an application's destination is `<ip_address>:443`, while the source is `<ip_address>:<ephemeral_port>`. When application connects to a machine defined in ip_address, it expects a response, hence ingress traffic has to be whitelistes as well. In stateful firewall case, or security groups in AWS, is when ingress traffic is automatically allowed.

# Network Address Translation (NAT)

* NAT is designed to overcome IPv4 shortages
* Translates Private IPv4 addresses to Public

Static NAT is what Internet Gateway in AWS is.
While Port Address Translation is what NATGatway in AWS is.

# DDoS

* HTTP layer, where simulating GET requests
* Protocol Attach (SYN attack)
* Volumetric / Amplification (using DNS services)

# VLANs, TRUNKS and QinQ

![image](https://github.com/MrR0807/Notes/assets/24605837/15ee2f5d-e5b8-401b-9c37-61fb8c874d65)

![image](https://github.com/MrR0807/Notes/assets/24605837/4e69ddaa-3768-4bd4-aaf3-30dfe6a08dcd)

**802.1Q means (VLANS)** and **802.1AD (nested QinQ VLANS)**.

# Border Gateway Protocol (BGP)

Border Gateway Protocol (BGP) is a standardized exterior gateway protocol used to exchange routing information between different autonomous systems (ASes) on the internet. It's the protocol used by internet service providers (ISPs) and large organizations to route traffic between their respective networks.

BGP is path-vector protocol it exchanges the best path to a destination between peers.

# IPSEC VPN Fundamentals

IPSEC has two main phases:
* IKE (Internet Key Exchange) Phase 1 (Slow & heavy) - this defines a protocol how crypto keys are exchanged. Where Asymmetric encryption to agree on Symmetric key.
* At the end of this phase, there is a tunnel .
* IKE Phase 2 (Fast & Agile) - uses the keys agreed in phase 1. Agree encryption method, and keys used for bulk data transfer. 

Phase 1 and Phase 2 are two different stages, because phase 2 tunnel can be discarded, while Phase 1 remains. Hence when new Phase 2 tunnel is required, it can happen faster.

![image](https://github.com/MrR0807/Notes/assets/24605837/38c4cba3-c06b-45ee-b63a-a217816ef2f9)

![image](https://github.com/MrR0807/Notes/assets/24605837/63887db1-b952-4cd0-a573-d358540f8f69)

There are two types of VPNs:
* Policy based VPNs. Rules match traffic. It can have different rules/security settings per traffic.
* Route based VPNs. Matches a single pair of security associations.

![image](https://github.com/MrR0807/Notes/assets/24605837/56f87e50-209b-46e9-bed1-456ffef78031)

## HTTPS vs IPSEC VPN

* HTTPS operates at the application layer (Layer 7). IPsec operates at the network layer (Layer 3).
* IPsec VPN tunnels are used to create secure connections between two networks or between a remote client and a network. They encrypt all traffic passing through the VPN tunnel, including all types of IP traffic (not just HTTP).
* IPsec VPN tunnels can be configured as point-to-point tunnels (connecting two network devices directly like HTTPS) or site-to-site tunnels (connecting entire networks together).
* IPsec VPN tunnels support various authentication methods, including pre-shared keys, digital certificates, and Extensible Authentication Protocol (EAP).

# DNS

* DNS Zone is a like a database which contains records e.g. netflix.com
* The data is stored in a file caleed ZoneFile
* Name Server is a DNS server which hosts 1 or more Zones and stores 1 or more ZoneFiles
* There are two types of records: Authoritatives (source of truth) and Non-Authoritative/Cached copies of records.

![image](https://github.com/MrR0807/Notes/assets/24605837/7ef7dea9-30f9-4823-9a01-5510a81878d7)

![image](https://github.com/MrR0807/Notes/assets/24605837/408dffe5-3932-4f46-a8a9-704bfbd06c77)

![image](https://github.com/MrR0807/Notes/assets/24605837/bf0586b8-bdf8-4389-8bef-23f275f0d67f)

# Recovery Point Objective (RPO) & Recovery Time Objective (RTO)

* RPO - maximum amount of data (time) that can be lost during a disaster recovery situation before that loss will exceed what the organisation can tolerate. Banks have almost zero RPO.
* RTO - maximum tolerable length of time that a system can be down after a failure or disaster occurs.

# AWS fundamentals

## Public vs Private Services

**Public and Private Service is from Networking perspective**.

![image](https://github.com/MrR0807/Notes/assets/24605837/98f93b1f-dcc5-4192-89ff-6bc26001d05f)

## AWS Global Infrastructure

Service Resilience:
* Globally Resilient - very few AWS services provide this. IAM and Route 53 are globally resilient services.
* Region Resilient - they generally replicate data between AZs.
* AZ Resilient - if AZ fails, then the service will fail.

## Virtual Private Cloud (VPC) Basics

* VPC is within 1 account & 1 region. They are region resilient.
* VPC is private and isloated unless you decide otherwise. Services within VPC can communicate, but are isolated from public AWS zone and public internet.
* There are two types of VPCs - Default VPC and Custom VPC. You get only **one Default VPC per region**. Default VPCs come pre-configured in very specific way and all networking configuration is handled on your behalf by AWS. But because of that they are a lot less flexible than cusotm VPCs.
* Unless you configure otherwise, there is no way for Custom VPC to communicate outside their specific network. In other words, by default it is private.

### Default VPC

Default VPC always gets the same VPC CIDR - `172.31.0.0/16`. The default VPC is configured to have a subnet in every AZ.

![image](https://github.com/MrR0807/Notes/assets/24605837/89edce53-0696-48b4-8c66-1a6274329715)

* One per region - can be removed & recreated. In this case you can have 0 VPCs in the region. Some AWS services assume that default VPC will be present, hence it is best to keep them as is and not use for any production related work.
* One of strenghts and weakness of default VPC is always the same CIDR range - `172.31.0.0/16`.
* `/20 ` subnet in each AZ in the region created.
* By default, VPC has configured Internet Gateway (IGW), Security Group (SG) & NACL.
* By default, anything placed inside default VPC, gets assigned a public IPv4 address.

## EC2 Basics

* Private service by default - uses VPC networking.
* AZ resilient - instance fails if AZ fails.
* Main Instance LifeCycle states: Running, Stopped, Terminated. Stopped instance still generates storage charges.
* AMI (Amazon Machine Image) can be created to run EC2 instance or AMI can be created from running EC2 instance.  
* AMI is either private - only the owner can use it; or you can explicitly add other AWS accounts, so those can use it; or you can make it public, hence everybody can use it.
* To connecto EC2 instance running Windows you have to use 3389 Remote Desktop Protocol, for Linux - SSH port 22.

## S3 Basics

* Global Storage PLatform - Regional Resilient.
* Public service.
* S3 Objects are identifiable by file name. Object value is content being stored. One object can be from 0 to 5 TB.
* Bucket name needs to be globally unique - accross all regions and all accounts. Buckets can hold unlimited amount of objects.
* Bucket has flat structure. There are no folder in practice. However, say S3 UI can present structures which resemble folders. **Folders are referred as prefix names, because they are part of object name**.
* Bucket name has to be between 3 - 63 characters, all lower case, no underscores.
* Start with a lowercase letter or a number.
* Buckets - 100 soft limit, 1000 hard per account.

### S3 Patterns and Anti-Patterns

S3 is an object storage - not file or block. Object storage means that if you have a requirement where you're accessing the whole of these entities, the whole of an object (e.g. an image, an audio file) and you're doing all of that at once, then it is a candidate for object storage. If you have Windows server, which nees to access a network file system, then it has to be file-based storage. S3 is also not block storage, because it cannot be mounted. Block storages can be used to create disks that can be attached to VMs.

**In Exam S3 should be the default pick for INPUT/OUTPUT storage**.

## CloudFormation (CFN) Basics

* CloudFormation is written either in YAML or JSON.
* CloudFormation `Resources` tells what to create. If `Resources` are added then CloudFormation creates them. If they are update - then updates, if removed from the template - they are removed for infrastructure.
* **NOTE! If you have `AWSTemplateFormatVersion` and `Description`, then Description has to immediately follow `AWSTemplateFormatVersion` field. This is a trick question used in Exam. `AWSTemplateFormatVersion` is not mandatory**
* `Metadata` field has many fucntions, but one of the things that it does is it can control how the different things in the `CloudFormation` template are presented through the AWS console UI. You can specify groupings, order etc.
* `Parameters` field promps the user to add more information.
* `Mappings` field allows you to create lookup tables.
* `Conditions` field allows decision making in the template depending on some parameter value.
* `Outputs` field, after template is finished, presents outbus based on what's being created, updated or deleted (e.g. return instance ID).

NOTE! When you upload a template file to CloudFormation, it automatically creates an S3 bucket.

## CloudWatch Basics

CloudWatch is three products in one. CloudWatch is public service.
* Metrics - comes from AWS Products, Apps, on-premises system. You can do actions on said metrics. Some other kinds of services, which are not managed by AWS might need a CloudWatch Agent to collect data.
* CloudWatch Logs - comes from AWS Products, Apps, on-premises system. Same thing as with metrics, you need to install CloudWatch Agent if you need logs from other non-AWS managed products.
* CloudWatch Events - two types of situations: something happens, for example CPU limit reached - event is generated; it can generate events at a certain times to do something.

CloudWatch Namespace is a container which separate data into different areas. There is one important aspect, all AWS data goes to `AWS/service` namespace e.g. AWS/EC2, AWS/S3 etc.

## Shared Responsibility Model

Red is AWS responsibility.

![image](https://github.com/MrR0807/Notes/assets/24605837/9359dfb5-2497-4369-bd45-c1677e179ce5)

AWS is responsbile for the security of the cloud. You are responsible for the security in the cloud.

## High-Availability vs Fault-Tolerance vs Disaster Recovery

### High-Availability

Definition - aims to **ensure** an agreed level of operational **performance**, usually **uptime**, for a **higher tan normal period**. Usually people think that having HA is that system never experience outage or user never experience failure. **That is not true**. A highly available system is design in providing services as often as possible. High Availability is about maximasing system's online time.

### Fault-Tolerance

Definition - is the property that enables a system to **continue operating properly** in the event of the **failure of some** (one or more faults within) of its **components**.

Fault tolerance means that if a system has faults, then it should continue to operate properly, even while those faults are present and being fixed. It means it has to continue to operate through a failure without impacting customers.

For example, if the monitor is tracking a person's health indicators, any downtime could be fatal. That means having HA is not enough. In this instance, monitor should have additional server and communicate in active active scenario. We can go even further, by adding additional monitor which also talks with two servers. HA helps to minimize outages, but FA is another level, which means levels of redundancy and system components. You have two lungs, if one fails, you have FA, because you can continue to breath. In HA scenario, you need intubation ASAP otherwise you're dead.

### Disaster Recovery

Definition - a set of policies, tools and procedures to **enable the recovery** or **continuation** of **vital** technology infrastructure and systems **following a natural or human-induced disaster**.

## Route53 Fundamentals

* You can register domains. It has integration will all high level domain registry (e.g. .com, .io, .net)
* Host Zones Files for you on managed name servers, which it provides.

Below steps which paint the picture of what happens when you register a domain:
* Route 53 check with the registry if the domain is available.
* Then Route 53 creates a zone file with said domain.
* Allocates name services for this zone. These services are created, managed and distributed globally.
* It takes the zone file and places them into those services.
* As part of registering a domain, it communicates with high level domain registry and adds these name server records into a zone file for the top level domain. By adding the name server records to the high level domain zone, they indicate that oru four name servers are all authoritative for the domain.
* Every time you create Hosted Zone, it always create 4 named servers to contain Zone files.

![image](https://github.com/MrR0807/Notes/assets/24605837/9ba09789-66b6-4cc7-affb-bf6e4649b0e3)

* Global Service with a singal database.
* Globally Resilient.

![image](https://github.com/MrR0807/Notes/assets/24605837/9dcf2545-b26d-4719-8493-5cc9f9bf6459)
































