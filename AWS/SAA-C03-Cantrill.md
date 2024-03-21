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

IAM Acess Keys are long term credentials used by IAM Users. When you're using UI you will generally use user and password, while when you use CLI - Access Keys.

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
While Port Address Translation is what NAT Gateway in AWS is.

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
* To connect to EC2 instance running Windows you have to use 3389 Remote Desktop Protocol, for Linux - SSH port 22.

## S3 Basics

* Global Storage Platform - Regional Resilient.
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

Definition - aims to **ensure** an agreed level of operational **performance**, usually **uptime**, for a **higher than normal period**. Usually people think that having HA is that system never experience outage or user never experience failure. **That is not true**. A highly available system is design in providing services as often as possible. High Availability is about maximasing system's online time.

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

## DNS Record Types

A and AAAA usually go both, and point to the same IP, just different IP protocol versions.

![image](https://github.com/MrR0807/Notes/assets/24605837/62bf2510-7b9c-4ff6-8d1d-690b0202f1e6)

**CNAME**

CNAME records cannot point to IP, but only to a different record. NOTE! This might be a trick question in exam. For example, we have a record with "A server" which points to "172.217.25.36". It is fairly common for a server to perform multiple tasks. In this case, maybe it provides FTP, Mail, Web Services. All of them point to `A` record. It is like alias, with a caveat that it points to another object (A or AAAA record) instead of IP.

**MX Records**

MX (Mail Exchange) records are a type of DNS (Domain Name System) record used to specify the mail servers responsible for receiving email messages on behalf of a domain. MX records play a crucial role in the email delivery process, as they help route email messages to the correct mail servers based on the recipient's domain.

Priority: MX records have an associated priority value, which indicates the order in which mail servers should be contacted for email delivery. Mail servers with lower priority values (e.g., 0, 10, 20) are preferred over those with higher priority values. If multiple MX records have the same priority, they are considered equally preferred, and the sender's mail server can choose any of them for delivery.

Format: MX records consist of two main components: the mail server hostname or domain name and the priority value. The format of an MX record looks like this:

```
example.com.   IN   MX   10   mail.example.com.
```

* The first field specifies the domain name to which the MX record applies (e.g., example.com).
* The second field (IN) indicates the DNS record class (Internet).
* The third field (MX) specifies the record type (Mail Exchange).
* The fourth field (10) is the priority value.
* The fifth field specifies the hostname of the mail server responsible for receiving email for the domain (e.g., mail.example.com).

Fallback Mechanism: If the sender's mail server cannot establish a connection with the primary mail server specified in the MX record (due to network issues or server unavailability), it will attempt delivery to the next mail server in the priority list. This fallback mechanism ensures that email delivery is resilient to server failures.

Multiple MX Records: A domain can have multiple MX records, each with a different priority value. This allows administrators to specify backup or secondary mail servers to handle email delivery if the primary servers are unavailable.

![image](https://github.com/MrR0807/Notes/assets/24605837/290aaea9-050f-45fa-866f-9ba01cd03196)

**TXT**

A TXT (Text) record is a type of DNS (Domain Name System) record used to store arbitrary text data associated with a domain name. TXT records can contain any type of human-readable text, including descriptive information, configuration settings, authentication tokens, or verification codes. These records are commonly used for various purposes, such as email authentication, domain ownership verification, and service configuration.

Here's everything you need to know about TXT records:

Purpose: The primary purpose of TXT records is to store textual information associated with a domain name. They provide a flexible mechanism for domain owners to add custom text data to their DNS records, which can be used by DNS resolvers, email servers, web services, and other systems for various purposes.

Format: TXT records consist of a domain name, a TTL (Time to Live) value, and one or more text strings enclosed in double quotes (""). The format of a TXT record looks like this:

```
example.com.   IN   TXT   "v=spf1 include:_spf.example.com ~all"
```

* The first field specifies the domain name to which the TXT record applies (e.g., example.com).
* The second field (IN) indicates the DNS record class (Internet).
* The third field (TXT) specifies the record type (Text).
* The fourth field contains the text data enclosed in double quotes.

Usage:

* Email Authentication: TXT records are commonly used for email authentication mechanisms such as SPF (Sender Policy Framework), DKIM (DomainKeys Identified Mail), and DMARC (Domain-based Message Authentication, Reporting, and Conformance). These records contain policy information that helps email servers verify the authenticity and integrity of email messages sent from a domain.
* Domain Ownership Verification: TXT records are often used to verify domain ownership for services such as Google Workspace, Microsoft 365, and domain registrar services. Providers may require domain owners to add specific TXT records to their DNS configuration to prove ownership of the domain.
* Service Configuration: TXT records can store configuration settings or service parameters used by various applications and services. For example, they can be used to specify configuration options for web servers, messaging services, or domain verification services.
* Informational Data: TXT records can contain arbitrary text data, such as descriptive information, contact details, or usage instructions provided by domain owners.

Length and Content: **TXT records can contain up to 255 characters per string, and a single TXT record can contain multiple strings.** Each string is typically separated by whitespace or enclosed in double quotes. Some DNS providers support multiple TXT records for the same domain, allowing for more extensive text data storage.

# IAM, ACCOUNTS AND AWS ORGANISATIONS

## IAM Identity Policies

IAM Policy is just a set of security statements to AWS. It grants or denies accesses to resources to anybody who uses the policy.

There are four properties for each policy statement:
* Sid (Optional) - The Statement ID, or SID, provides a way to reference and manage individual statements within a policy. It helps identify and distinguish one statement from another, especially in complex policies with multiple statements. **The SID is a string value that must be unique within the policy**. In other words  it is simply a label used for identification purposes.
* Effect - it is either allow or deny.
* Action - The "Action" element within a statement specifies the actions that are allowed or denied for the associated AWS resources. Actions are defined using AWS service-specific action names. The format is `service:operation` e.g. `s3:GetObject`. You can also use asterix to indicate all actions.
* Resource - The "Resource" element within an IAM policy statement identifies the AWS resources to which the permissions are applied. You can use a wildcard, or you can use specific resources list. If you specify a resource, you have to use ARN.

Multiple allow and deny statements can overlap.

![image](https://github.com/MrR0807/Notes/assets/24605837/044ffe6e-5a8a-4cce-87b0-50fc6885a62a)

There are explicit rules on allow and deny:
* If there is an explicit DENY action, then it overrulles everything.
* Explicit ALLOWs take effect unless, there is explicit deny.
* By default it is implicit deny.

Even if the user has multiple policies, belongs to a group, and a resource has its own policy, the same rule applies - deny, allow, implicit deny. If there is a deny in any policies, then it wins. If there is an explicit allow, then use can access a resource. If there is no allow, then implicit deny.

There are two types of policies:
* Inline policies - Inline policies are policies that are directly embedded or "inlined" within a specific IAM entity (user, group, or role). They are defined and managed directly within the configuration of the IAM entity to which they apply. Inline policies are scoped to a single IAM entity and cannot be reused across multiple entities. Each IAM entity can have its own set of inline policies. Inline policies are created, edited, and deleted directly within the IAM entity's configuration. Changes to inline policies are applied immediately and are specific to the IAM entity to which they are attached.
* Managed policies - Managed policies are standalone policies that are created and managed independently of IAM entities. They are stored separately from IAM entities and can be attached to multiple IAM users, groups, or roles.
  * AWS managed policies – Managed policies that are created and managed by AWS.
  * Customer managed policies – Managed policies that you create and manage in your AWS account. 

## IAM Users and ARNs

IAM Users are an identity used for anything requiring long-term AWS access e.g. Humans, Applications or Service Accounts. **NOTE! If you can imagine a concrete user, application or service account in the scenario provided by the exam - 99% the answer is IAM User**. 

ARN (Amazon Resource Name) - uniquely identify resources within any AWS account.

ARN format

![image](https://github.com/MrR0807/Notes/assets/24605837/94465ef6-7141-4eb7-973f-4e015e2453be)

```
arn:aws:s3:::catgifs
arn:aws:s3:::catgifs/*
```

These are two very different ARNs. One refers to a bucket, while the other refers to the objects inside this bucket. The top one would allow IAM policy to create, delete or modify the bucket itself, while the bottom one would allow actions on objects.

**NOTE!** Double collon like with S3 example is not the same thing as providing a wildcard. Double collon means that there is nothing to specify, like S3 not have a region. While if you'd specify an EC2 instance, you would need to specify a region or use wildcard.

**NOTE!** You can only have 5000 IAM Users per account. IAM User can be a member of 10 groups.

**NOTE!** Only 5000 AWS users per account.

## IAM Groups

IAM Groups are containers for Users. **NOTE! Exam might try to trick you - you cannot login into groups.**

IAM Groups can have inline or managed policies attached.

**There is no limit to how many users can be in a group. But again, one user can be a member of 10 groups.**

**You can't have any nesting. You can't have groups within groups.**

**There is a limit of 300 groups per account. But this can be increased with a support ticket.**

**Resource policy cannot grant access to an IAM Group.** It can, however, grant access to Roles or Users via ARN.

## IAM Roles

The role isn't something that represents you. A role is something which represents a level of access inside an AWS account. It is used short term by other identities. IAM Role can have two types of policies: Trust Policy and Permissions Policy. Trust Policy defines which identities can assume that role. Trust policy can reference different things - reference identities in the same account (other IAM users, other roles and AWS services like EC2) and it can reference identities in other AWS accounts. Roles can be assumed by unknown players. Once role is assumed, AWS provides temporary Security Credentials.

STS - secure token service generates these temporary credentials. 

## When to use IAM Roles

Good situations to use roles:
* When services (e.g. AWS Lambda) want to do actions on your behalf. Like start a Lambda service after an alert to start more EC2 instances. It is better to create a Lambda Execution Role which trusts Lambda Service. Other alternative would be to hard code security credentials into the code.
* For emergency or out of the normal kind of situations. A user can obtain short term emergency role to do certain actions, which under normal circumstances, is not allowed.
* In cases when you have, say on-premise, active directory (Windows) users who want to use AWS with their accounts. In this case Active Directory user assumes an AWS role in order to do certain actions within AWS.
* For example you have a mobile application which accesses a certain AWS resource. The mobile application can have millions of users, hence AWS users are out of the question. Furthermore, users might use Web Identity Fedaration (Facebook, Google etc).
* Multi AWS account scenario.

**NOTE! External accounts cannot be used directly to access AWS resources**.

![image](https://github.com/MrR0807/Notes/assets/24605837/8c3c4da0-67ce-418f-8a76-0dacd290815a)

![image](https://github.com/MrR0807/Notes/assets/24605837/bb330458-c5ae-4314-a7a2-5903aae998a7)

![image](https://github.com/MrR0807/Notes/assets/24605837/874ba568-c756-4045-87a1-05809fb73308)

## Summary in limitations of IAM

As of my last update, the default service limits for IAM entities are as follows:

IAM users: 5,000 per AWS account
IAM groups: 300 per AWS account
IAM roles: 1,500 per AWS account
IAM instance profiles: 1,500 per AWS account
IAM policies: 10,000 per AWS account

## Service-linked Roles & PassRole

A service-linked role is an IAM role linked to a specific AWS service. They provide a set of permissions which is predefined by a service. Provides permissions that a service needs to interact with other AWS services on your behalf. Service might create/delete the role or allow you to create the role during the setup process of that service or created in IAM.

Key difference between IAM Role and Service-linked Role is that you cannot delete service-linked role until it is no longer required.

Futhermore, you can have a permission to pass a role, which supersedes your permissions. For example, you might only have a role to pass service-linked role to CloudFormation, but passed role in CloudFormation has all possible permissions to create all resources in AWS, hence the `PassRole`.

## AWS Organizations

AWS organisations allows for business to manage multiple accounts in a cost effective way.

One AWS account can create an organisation. Said account becomes Management Account (Previously Master). Using master account you can invite other accounts into organisation. When they join the organisation, they become member accounts. At the top of organisation tree, sits an Organization Root. This is not equal to Account Root User. Organization root can have within itself Organizational Units, which can contain Member Accounts or Master Account. 

Billing changes in Organisation. Management/Master account gets all the bills, while payments are removed from member accounts. 
**Organisation can benefit from reservations and volume discounts, because resources are pooled.**
Organisation also feature a service called Service Control Policies (SCP) which can control what AWS member accounts can and cannot do.
You can create new accounts inside organisation - all you need is unique email address.
Organisation also changes best practices around AWS accounts. Not all AWS accounts require separate IAM Users. Instead, you can utilise them via IAM Roles.

If you create AWS account in Organisation from Organisation dashboard, it automatically gets assigned a role, which allows to access it from other accounts. If you add an existing account - this role is not attached, hence you need to create one yourself.

## Service Control Policies

![image](https://github.com/MrR0807/Notes/assets/24605837/8e16afc7-e140-4026-9a7b-36247616a7ab)

**NOTE! Management account is not affected by SCP.**

**SCPs are account permissions boundaries. They limit what the account can do (including account root user)**. The thing is that you're not restricting account root user, but account itself.

Policy Evaluation: SCPs are evaluated before IAM policies, service control policies, and resource policies. This means that if an SCP denies access to a service or action, IAM policies, service control policies, and resource policies cannot grant access, even if they allow it.
Granular Control: SCPs support granular control over permissions, allowing you to specify which AWS services, actions, and resources are allowed or denied for member accounts. You can create custom SCPs tailored to specific use cases or compliance requirements.

SCP do not grant permissions. They set boundaries.

When you enable SCP on your organisation, AWS Apply a default policy which is called full AWS access. In other words, it means that SCP have no effect.

To have an allow list, you'd have to remove default Full AWS access SCP and then implict deny all would start to work. Hence, you'd need to add resources into allow list.

## CloudWatch Logs

* **Public Service** - usable from AWS or on-premises.
* Store, Monitor and access logging data.
* AWS Integrations - EC2, VPC Flow Logs, Lambda, CloudTrail, R53 and more.
* Can generate metrics based on logs - metric filter.


Three ways to integrate with CloudWatch:
* By using Managed Services.
* By deploing unified CloudWatch Agent into custom applications.
* By using developer's kit inside the code.

Architecture of CloudWatch Logs. **NOTE! Log Groups are where you define retention and permissions. It is also where Metric Filters are defined.**

![image](https://github.com/MrR0807/Notes/assets/24605837/e527f41b-5b54-46b0-ae65-f6c8d6aa8409)

## CloudTrail Essentials

* Logs API calls/activities as a CloudTrail Event
* 90 days stored by default in Event History
* Enabled by default - no cost for 90 day history
* To customise the service you need to create 1 or more *Trails*
* There are three types of events: Management Events, Data Events, Insight Events.

* Management events - These events track management actions performed on AWS resources, such as creating, modifying, or deleting resources. Management events include API calls made through the AWS Management Console, AWS CLI, AWS SDKs, and other AWS services.
* Data Events - Data events provide insight into access and usage of data within AWS resources. These events track actions such as object-level access in Amazon S3, API calls related to AWS Key Management Service (KMS) keys, and database activity in Amazon RDS, DynamoDB, and other AWS services.
* Insight Events - Insight events provide additional context and analysis for certain types of events. For example, AWS CloudTrail Insights can analyze CloudTrail logs to identify unusual activity patterns, security threats, or operational issues.

CloudTrail by default track only management events. Data events create a very high load of data.

CloudTrail Trail can be configured in two ways:
* One Region - only tracks events in given region.
* All Regions - tracks events in every region.

As mentioned earlier, there are region based services and global services (e.g. IAM, STS, CloudFront). CloudTrail needs to enable this in order to receive global region events. They always log their events to US-EAST-1 (N.Virginia). This feature is enabled by default if you enable CloudTrail via UI. 

### Summary

* Enabled by default in all AWS accounts, but persists data only for 90 days.
* You can configure special Trails which will save data in S3 or CloudWatch Logs.
* Management events only by default.
* IAM, STS, CloudFront are Global Service Events.
* **Data is not real time - there is a 15 minute delay**.

**Because global service events are only available in US East (N. Virginia) beginning November 22, 2021, you can also create a single-Region trail to subscribe to global service events in the US East (N. Virginia) Region, us-east-1. Single-Region trails will no longer receive global service events beginning November 22, 2021, unless the trail already appears in US East (N. Virginia) Region, us-east-1. To continue capturing global service events, update the trail configuration to a multi-Region trail.**

## AWS Control Tower

* Quick and Easy setup of multi-account environment.
* Orchestrates other AWS services (e.g. Organizations) to provide this functionality.
* Control Tower uses Organizations, IAM Identity Center (formally known as AWS SSO), CloudFormation, Config and more.
* Think of Control Tower as evolution to Organizations, by adding more features and automation.
* Landing Zone - multi-account environment. This is what most people will be interacting with when they think of Control Tower.
* Provides SSO/ID Federation (provided using IAM Identity Center), Centralised Logging & Auditing (uses combination of CloudWatch, CloudTrail, SNS etc).
* Provides Guard Rails - Detect/Mandate rules/standards across all accounts within the Landing Zone.
* Account Factory - Automates and Standardises new account creation.
* Dashboard - single page oversight of the entire environment.

You create with any AWS account and it becomes Management Account.

![image](https://github.com/MrR0807/Notes/assets/24605837/4e370c30-513f-4954-bd3c-117528249b03)

![image](https://github.com/MrR0807/Notes/assets/24605837/6f55a104-c345-4ecd-a757-945bc8eb7853)

![image](https://github.com/MrR0807/Notes/assets/24605837/c384f15d-33fd-4e9e-ab4f-f4e302f6ba8a)

### Account Factory

The AWS Control Tower Account Factory is a feature provided by AWS Control Tower, a service that automates the setup and management of a multi-account AWS environment based on AWS best practices and guidelines. The Account Factory simplifies and accelerates the process of creating new AWS accounts within your AWS Control Tower environment.

Here's an overview of the AWS Control Tower Account Factory:

Account Provisioning: The Account Factory automates the process of provisioning new AWS accounts, allowing you to create accounts quickly and consistently. It provides a centralized interface for account creation, streamlining the process and reducing the potential for errors.

Templates and Guardrails: The Account Factory allows you to define templates and guardrails for new AWS accounts. Templates specify configurations, settings, and resources that should be provisioned in each new account, while guardrails enforce policies and controls to ensure compliance with organizational standards and security requirements.

Customization: The Account Factory supports customization to accommodate the specific needs and requirements of your organization. You can define custom templates, guardrails, and policies to tailor the account creation process to your organization's preferences and standards.

Integration with AWS Organizations: The Account Factory integrates with AWS Organizations, the service that centralizes management of multiple AWS accounts. It leverages AWS Organizations to provision new accounts within the organizational structure defined by AWS Control Tower, ensuring consistency and alignment with organizational policies.

Lifecycle Management: The Account Factory supports lifecycle management of AWS accounts, including account creation, modification, and deletion. It provides visibility and control over the entire lifecycle of accounts, helping you manage resources efficiently and maintain compliance with organizational policies.

Automation and Scalability: The Account Factory is designed for automation and scalability, allowing you to provision and manage large numbers of AWS accounts efficiently. It leverages AWS services such as AWS CloudFormation, AWS Lambda, and AWS Step Functions to automate account provisioning workflows and scale to meet the needs of your organization.

Overall, the AWS Control Tower Account Factory simplifies the process of creating and managing AWS accounts within your AWS Control Tower environment. It provides a centralized, automated solution for account provisioning, customization, and lifecycle management, enabling you to maintain consistency, compliance, and efficiency across your AWS environment.

# Simple Storage Service (S3)

## S3 Security

S3 is private by default. Only the AWS account which created it has access to it. Anything else has to explicitly granted. There are few ways how this can be done:

### S3 Resource policy

By defining resource policy. Resource policy just like identity policy is attached to an entity. From identity policy perspective, it tells what said user can and cannot do. From resource policy, it tells who and what can do with said resource. **Unique thing is that resouce policy can reference other accounts (any accounts)**, which provides cross account access. **Resource policies can allow or deny anonymous principals**.

Resource policy has an **Principal** field as well. This field specifies the IAM users, roles, federated users, AWS accounts, or AWS services to which the permissions apply. The principal is typically identified by its ARN. For example, specifying "*" as the principal allows the permissions to apply to all principals. In identity policy it is implied.

![image](https://github.com/MrR0807/Notes/assets/24605837/a82dcfb4-c301-49b1-9b0b-a9ce32e01944)

### Bucket Policies

Bucket policies can control who can access buckets, even control at the IP level. This bucket policy denies access to everybody unless you're IP is `1.3.3.7/32`.
 
![image](https://github.com/MrR0807/Notes/assets/24605837/76746fe4-a2a0-4108-a9b8-fb19cc0b83bc)

NOTE! If there is access from other AWS account to said bucket, that AWS account has to have access rights to S3 in general, otherwise, even if the bucket explicitly allows for said identity to access it, it might not due to AWS account policies.

### Access Control Lists (ACL)

**These are legacy**.

### Summary

* Identity - Controlling different resources
* Identity - You have a preference for IAM
* **Identity - Same Account**
* Bucket - Just controlling S3
* Bucket - Anonymous or Cross-Account
* ACLs - NEVER

## S3 Static Website Hosting

When you create a static website from S3, you have to point to Index page and Error page.

If you want to use custom domain via R53, then bucket name matters. You can only use a custom domain with a bucket if the name of the bucket matches the domain. For example, if website is called `top10.animalsforlife.org`, then my bucket name would need to be called `top10.animalsforlife.org`.

## Object Versioning & MFA Delete

Bucket starts in disabled versioning. You can enabled it. **Once it is enabled - you cannot disable it**. However, it can be moved to suspended.

Object Key in S3 is objects name. For example, if we have an object and versioning is disabled, then we have this metadata:

```
KEY = examplename.txt
id = null
```

If versioning is enabled, then key becomes some value:

```
KEY = examplename.txt
id = 1111
```

New version keeps the old one and add a new one:

```
KEY = examplename.txt
id = 2222
KEY = examplename.txt
id = 1111
```

By default, S3 returns the latest version.

If we want to delete `examplename.txt` and don't provide any version id, it will add a new object, called delete marker. But it doesn't delete anything, it just hides the object. If we want to undo the deletion, the delete marker is just deleted and objects are visible again. If you really want to delete an object, you need to specify a version ID.

### MFA Delete is always in the exam

MFA delete is a configuration property within the versioning of the bucket. When you enable MFA delete, it means that MFA is required every time you want to change bucket versioning state (suspend, re-enable versioning) or delete an object version.

## S3 Performance Optimization

Problems with single PUT upload to S3:
* Single data stream to S3 - both not reliable and slow.
* Stream fails - upload fails. Hence requires full restart.
* Limited to upload only up to 5 GB.

Multipart Upload:
* Min data size for multipart upload is 100mb.
* 10000 max parts.
* Range between 5mb and 5gb.
* Last part can be smaller than 5mb.
* Parts can fail and be restarted.
* Transfer rate = speeds of all parts. In other words, it is much faster.

### S3 Accelerated Transfer

Transfer Accelerator uses network of AWS Edge locations. There are restrictions: bucket name cannot contain periods and it needs to be DNS compatible in its naming. Instead of upload data directly to S3, it is uploaded to nearest edge location. Then Edge Location can upload directly to S3 using AWS global network, because it is in control of it.

When you enable Transfer Acceleration it will provide a different URL, which is an edge location. This URL has to be used in order to take advantage of Transfer Accelarator.

## Key Management Service (KMS)

* Regional & Public Service.
* Create, Store and Manage Keys.
* Symmetric and Asymmetric Keys.
* Cryptographic operations (encrypt, decrypt).
* **Keys never leave KMS**. Its primary feature that keys are securely placed within KMS.
* **For exam, provides FIPS 140-2 (Level 2) standard**.


KMS contains KMS keys. They are **logical**. Contains data like ID, date and resource policy. They are backed by physical key material. Actual keys. Actual keys can be generated or imported. **KMS keys can be used to encrypt or decrypt up to 4KB of data**.

Key Material is just crypto key. For example, for asymmetric keys, it would be a private key.

![image](https://github.com/MrR0807/Notes/assets/24605837/3c5093ea-f951-4986-8ab1-79dbd0da1c7f)

Data Encryption Keys (DEKs) are another type of key which KMS can generate. They are generated using KMS key.

The flow:
* KMS generates two DEKs - one is plaintext and another is encrypted.
* Plaintext key should be used to encrypt the data **by you**. **KMS does not encrypt data in this case.**
* Once the data is encrypted, the key should be discarded and data saved together with encrypted key.
* When you want to decrypt data, you provide encrypted key to KMS, which then decrypts it and provides you again, with plaintext key to decrypt the data.
* NOTE! KMS in this case does not keep the data, nor encrypted keys.
* KMS does not track the usage of data encryption keys.

**NOTE! Services like S3 generate encryption key for every single object**.

* KMS Keys are isolated to a **region** & never leave.
* There are multi-region keys.
* KMS keys can be customer owned or AWS owned (generated by AWS and used by AWS services). Customer owned can be either AWS managed (aka KMS key) or Customer Managed (aka DEK).
* Rotation - AWS managed keys can't be disabled and rotate every year. With Customer managed keys it is optional and by default also rotates every year.

### Key Policies and Security

Permissions on keys are controlled in a few ways. Many services will always trust the account that they're contained in. Meaning if you grant access via an identity policy, that access will be allowed unless there's an explicit deny. KMS different because it needs to have an explicit allow or deny on key policy. Key policy is like a resource policy (e.g. bucket policy). For customer managed keys, you can change it. Again, each KMS key has to be explicitly told that they trust AWS account which they are contained in.

```json
{
"Sid": "Enable IAM User Permissions",
"Effect": "Allow",
"Principal": {"AWS": "arn:aws:iam:111122223333:root"},
"Action": "kms:*",
"Resource": "*"
}
```

This allows for AWS account with `111122223333` to access KMS key.

## S3 Object Encryption CSE/SSE

CSE - customer side encryption. SSE- server side encryption.

**Note! Buckets aren't encrypted - objects are**.

This defines who encrypts the data at rest.

![image](https://github.com/MrR0807/Notes/assets/24605837/cf1cfae3-e192-4ecb-8fe0-6161978450c1)

**AWS recently made server side encryption mandatory**.

There are three types of Server Side Encryption:
* SSE-C Server Side Encryption with Customer Provided Keys. You provide Crypto Key and plaintext to encrypt. Data is encrypted and hash of the key is added to metadata. The Key is disgarded by S3 after. Customer should handle the keys on his side.
* SSE-S3 Server Side Encryption with Amazon S3 Managed Keys (Default). With this method, AWS handles both the encryption process and management of keys. When putting object into S3 you just provide the data. S3 generates a unique key for every object. You have 0 control over the key.
* SSE-KMS Server Side Encryption with KMS Keys stored in AWS Key Management Service.

![image](https://github.com/MrR0807/Notes/assets/24605837/47613b0b-7df5-4bb0-bfd7-b584568ddcec)

































