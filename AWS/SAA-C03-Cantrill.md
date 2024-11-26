![image](https://github.com/MrR0807/Notes/assets/24605837/92ac6e9e-6edd-4458-b0db-40cd9672007a)

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
* `Metadata` field has many functions, but one of the things that it does is it can control how the different things in the `CloudFormation` template are presented through the AWS console UI. You can specify groupings, order etc.
* `Parameters` field promps the user to add more information.
* `Mappings` field allows you to create lookup tables.
* `Conditions` field allows decision making in the template depending on some parameter value.
* `Outputs` field, after template is finished, presents outputs based on what's being created, updated or deleted (e.g. return instance ID).

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
* SSE-C Server Side Encryption with Customer Provided Keys. You provide Crypto Key and plaintext to encrypt. Data is encrypted and hash of the key is added to metadata. The Key is disgarded by S3 after. Customer should handle the keys on his side. When you want to decrypt the data, you have to provide plaintext key, which is again hashed and compared with metadata in S3.
* SSE-S3 Server Side Encryption with Amazon S3 Managed Keys (Default). With this method, AWS handles both the encryption process and management of keys. When putting object into S3 you just provide the data. S3 generates a unique key for every object. You have 0 control over the key.
* SSE-KMS Server Side Encryption with KMS Keys stored in AWS Key Management Service.

![image](https://github.com/MrR0807/Notes/assets/24605837/47613b0b-7df5-4bb0-bfd7-b584568ddcec)

SSE-S3 is a good default for most cases, unless you have specific regulatory requirements - if you need to control the keys and access to those keys, if you need to control rotation cadency, if you need role separtion. The role separation refers to the situation where S3 administrator can encrypt and decrypt objects. In some cases, we want administrators to administer buckets, but not able to managed content.

When S3 wants to encrypt an object using SSE-KMS, it has to liaise with KMS and request a new data encryption key to be generated using KMS Key. KMS delivers two data encryption keys - plaintext and encrypted. S3 uses plaintext to encrypt data and discards it, while saves encrypted key with object metadata. You also have logging and tracing regards using KMS keys. KMS master key is used to decrypt the encrypted copy of DEK key.

The biggest benefit of SSE-KMS is that you can role separate administration tasks and encryption/decryption process. 

![image](https://github.com/MrR0807/Notes/assets/24605837/fa194b2d-e010-48c1-87f2-77f86a2c9193)

![image](https://github.com/MrR0807/Notes/assets/24605837/f78114d3-366c-443b-a50f-32223516b525)

You can use both client-side encryption and server-side encryption. NOTE! This is on by default now.

**EXAM NOTE!** SSE-S3 uses AES-256. If you see AES-256, think SSE-S3.

In AWS Console you can define encryption method when uploading an object or on bucket.

## S3 Bucket Keys

Calls to KMS have a cost & levels where throttling occurs:
* 5.5k per second
* 10k per second
* 50k per second

So if we're uploading 100k objects into S3 per second, KMS will not support that.

![image](https://github.com/MrR0807/Notes/assets/24605837/82d732f4-71c8-439c-a815-caa0dcf1b686)

Bucket keys improve the situation. AWS KMS key generates a time limited bucket key, which generates DEKs within S3, without needing to call KMS every time.

![image](https://github.com/MrR0807/Notes/assets/24605837/54e39c51-6b9c-4cde-a060-b9353a66d6e3)

You will see fewer CloudTrail events, because of less interactions with KMS.

**NOTE!** If you're replicating S3 data between two buckets, where the source objects are not encrypted, while target uses default encryption, then said objects are encrypted.

## S3 Object Storage Classes

### S3 Standard

S3 Standard objects are replicated across at least 3 AZs in the AWS region. This replication part is crucial in order to determine S3 other tiers' costs.

**EXAM NOTE!** When objects are stored **durably** then a HTTP/1.1 200 OK response is provided by S3 API. S3 Standard has a milliseconds first byte latency.

**EXAM NOTE!** Use S3 Standard for **Frequently Accessed** Data which is **important and Non Replaceable**.

### S3 Standard-IA

Also everything is the same as in S3 Standard, differs:
* Half the price of S3 Standard for storage*
* It has a retrieval fee per GB. Overall costs increases with frequent data access.
* Minimum duration charge of 30 days - objects can be stored for less, but the minimum billing always applies.
* Standard-IA has a minimum capacity charge of 128KB per object.

**EXAM NOTE!** S3 Standard-IA should be used for **long-lived data**, which is **important** but where access is **infrequent**.

### S3 One Zone-IA

Share similarities with S3 Standard-IA: retrieval fee, minimum duration charge, object minimum capacity. The main difference - data is deployed in one AZ. You still get the same durability (11 9), but only if AZ is operational.

**EXAM NOTE!** S3 One Zone-IA should be used for **long-lived data**, which is **Non-Critical and Replaceable** and where access is **infrequent**.

### S3 Glacier - Instant

Standard-IA is for when the data is required once a month, Glacier - once every quarter. 

**EXAM NOTE!** S3 Glacier Instant should be used for long-lived data, accessed once per qtr with millisecond access.

### S3 Glacier - Flexible

It is similar to S3 Standard - same durability, stored in multiple AZs. However, they are **not immediately available and objects cannot be made public** anymore (e.g. using static website hosting). You can see objects in S3, but these are pointers and requires a retrieval process. There are 3 different retrieval processes:
* Expedited (1-5 minutes).
* Standard (3-5 hours).
* Bulk (5-12 hours).

Has minimal 40KB min size and 90 day min charge duration. 

First byte latency = minutes or hours depending on chosen retrieval process.

**EXAM NOTE!** Archival data where frequent or realtime access isn't needed (e.g. yearly) and retrieval process is from minutes to hours.

### S3 Glacier Deep Archive

Same as S3 Glacier. Differences:
* Minimum 180 day duration charge.
* Retrieval process:
  * Standard (12 hours).
  * Bulk (up to 48 hours).

**EXAM NOTE!** Archival data that rarely if ever needs to be accessed - hours or days for retrieval e.g. Legal or Regulation data storage.

### S3 Intelligent-Tiering

It is different from all previous tiers, because it contains 5 different tiers in itself:
* Frequent Access (S3 Standard)
* Infrequent Access (S3 Standard-IA)
* Archive Instant Access (S3 Glacier Instant)
* Archive Access (S3 Glacier Flexible)
* Deep Archive (S3 Deep Archive)

![image](https://github.com/MrR0807/Notes/assets/24605837/7bb18203-0a31-43da-adf6-747f54954ae5)

Mainly it moves between Frequent Access and Infrequent Access. You can configure additional Glacier tiers.

Instead of retrieval cost, Intelligent-Tiering has monitoring and automation cost per 1000 objects.

**EXAM NOTE!** S3 Intelligent-Tiering should be used for long-lived data, with changing or unknown patterns.

## S3 Lifecycle Configuration

* A Lifecycle configuration is a set of rules.
* Rules consist of actions, which can be performed on a Bucket or group of objects defined by prefix or tags.
* The actions are of two types: transition actions (change the storage class); expiration actions (delete object or objects or object versions).

Transition can happen like watefall. The top tier can move to any bottom tier with one exception - S3 One Zone - IA cannot move to S3 Glacier - Instant Retrieval:
* S3 Standard.
* S3 Standard-IA.
* S3 Intelligent-Tiering.
* S3 One Zone-IA.
* S3 Glacier - Instant Retrieval.
* S3 Glacier - Flexible Retrieval.
* S3 Glacier Deep Archive.

**EXAM NOTE!** There is a 30 day minimum period, where an object needs to remain on S3 Standard before then moving into Infrequent Access or One Zone Infrequent Access. You can always directly configure object's storage class via UI or CLI. But when you're using Lifecycle policy, needs to be in S3 Standard for 30 days. Of course you can start with Infrequent Access from the get-go.

A single rule cannot transition to Standard-IA or One Zone-IA and THEN to glacier classes. So if we're moving from S3 Standard to Glacier, then it will be 60 days minimum: 30 days in Standard, 30 days in IA. However, you can move from IA to Glacier faster with two rules.

## S3 Replication

There are two types of replications:
* Cross Region Replication.
* Same Region Replication.

Everything is the same when replication happens in same account or different account, the only difference is that in different account scenario, bucket policy has to allow role to access it.

![image](https://github.com/MrR0807/Notes/assets/24605837/5c4e8554-d660-4bb8-9a9b-da4c27aa6d23)

Replication options that are:
* Replicate objects or a subset.
* You can choose a storage class for objects - default is to maintain current.
* Define ownership - default is the source account. But differ owner can be defined.
* Replication Time Control (RTC) - it adds a guarantee 15 minute replication SLA onto this process. Without it - it's a best efforts process.

Important to remember:
* By Default replication is not retroactive. It means that if objects existed before replication was turned on, those objects are not replicated.
* Versioning needs to be turned on for replication to work.
* You can use Batch replication to replicate existing objects, but this needs to be specifically configured.
* One way replication only. Source to Destination. Recently AWS added a bi-directional replication, but needs to be specifically configured.
* Replication is possible with unencrypted data (which does not exist anymore), SSE-S3, SSE-KMS (with extra configurations) and SSE-C.
* It cannot replicate Glacier or Glacier Deep Archive objects.
* Replication requires that the owner of the source bucket has permissions on the objects which will replicate. You might create an S3 bucket an allow multiple accounts to place objects into it, then some objects will not belong to the source owner, hence no replication on those objects. **Destination account needs to own those objects**.
* Lifecycle management events are not replicated.
* Delete markers are not replicated. You can enable for those to be replicated as well, but they are not by default.

### S3 PreSigned URLs

There are three main ways how to provide access to S3 bucket to unauthenticated user:
* Give an AWS identity.
* Provide AWS Credentials.
* Make objects in S3 public.

None of these are ideal, hence there is a presigned urls. An AWS user has to have permissions to generate presigned URL. When a presigned URL is used, the holder of that URL is actually **interacting with S3 as the person who generated it**.

![image](https://github.com/MrR0807/Notes/assets/24605837/0a439b87-d6e8-4830-ac8c-09ad916b9ea3)

There is another type of architecture, where presigned urls are provided along with static website content. This way, download is happening directly from S3 instead of passing through a server.

![image](https://github.com/MrR0807/Notes/assets/24605837/43d34727-ac67-4bf5-9a01-3a8f8d3e4cda)

**EXAM NOTE!** 
* You can create a URL for an object you have no access to. But because you don't have access to it, then URL as well will not be able to access it. You can also generate presigned URL for non existing objects.
* When using the URL, the permissions match the identity which generated it **right now**. If you're getting access denied then it could mean that the generating ID never had access or **doesn't have right now**.
* Dont generate with a role. URL stops working when temporary credentials expire. URL might have a longer expiration period than role.

## S3 Select and Glacier Select

Most of the time you want to interact with full object from S3. However, if the object is really large, and you don't need it fully, you can use select which is SQL-Like statement. You only consume pre-filtered by S3. You select objects from CSV, JSON, Parquet, BZIP2 compression for CSV and JSON. This is kind of predicate pushdown.

## S3 Events

Notification is generated when an event occurs in a bucket. These can be delivered to SNS, SQS and Lambda Fucntions.

Events can be generated when object is created, delete, restore operations and when replication happens. You have to define event notification config. You have create a resourcep policy allowing S3 principal access. You can also use EventBridge which supports more types of events and more services.

![image](https://github.com/MrR0807/Notes/assets/24605837/af63dea3-c006-46fa-9934-6f7585150ac1)

## S3 Access Logs

![image](https://github.com/MrR0807/Notes/assets/24605837/b5b052b1-a821-45e6-8f7c-e2d5a4ccb302)

## S3 Object Lock

* Object Lock enabled on new buckets (you have to contact support for existing buckets).
* When you enable object locking, versioning is also enabled. 
* Object lock implements write once read many architecture. Once set, object versions can't be deleted or overwritten.
* These are individual versions that are locked.
* There are two ways to manage object retentions. Object version can have both of these, one or the other or none. These can be set on individual objects and also defaults on the whole bucket:
  * Retention periods
  * Legal holds

### Object Lock - Retention

You specify a retention period in days and or years. There are two ways to setup retention period:
* Compliance - the retention itself cannot be adjusted (it stays for that amount of time), the object cannot be deleted or overwritten. Even includes the account root user.
* Governance - special permissions (IAM) can be granted allowing lock settings to be adjusted - `s3:BypassGovernanceRetention`. And they have to provide a header along with their request - `x-amz-bypass-governance-retention:true`.

### Object Lock - Legal Hold

Set on an object version - ON or OFF (binary). No retention. You can't delete or change a specific object version until it is removed. An extra permission is required `s3:PutObjectLegalHold` to add or remove legal hold.
Prevents accidental deletion of critical object versions.

![image](https://github.com/MrR0807/Notes/assets/24605837/2c61ade2-7bec-4550-9ee6-0a8761feadaf)

## S3 Access Points

* Simplify managing access to S3 Buckets/Objects (when you have millions or billions).
* Rather than having 1 bucket with 1 bucket policy, you can have many access points allowing you to have different policies with different network access controls.
* Each access point has its own endpoint address.
* You can create via Console or via CLI command `aws s3control create-access-point --name <> --account-id <> --bucket`

**EXAM NOTE**. Remember `aws s3control create-access-point` command.

![image](https://github.com/MrR0807/Notes/assets/24605837/671b06f5-ac64-488c-a584-17a01168715f)

**NOTE!** You have to define matching permissions both in AccessPoints policy and Bucket Policies. However, you can do delegation where on the Bucket Policy you grant wide open access via the Access Point. This means that any action on any object is allowed as long as Access Point is used. Then, you can define granular controls via Access Point policy.

# VPC Basics

## VPC Sizing and Structure - PART1

VPC Considerations:
* What size should the VPC be. Because each service will occupy at least one IP.
* Are there any Networks we can't use.
* Try to predict other VPCs, Cloud, On-premises etc IP ranges.

Animals4Life ranges to avoid:
* On-premise 192.168.10.0/24 (192.168.10.0 -> 192.168.10.255)
* AWS Pilot 10.0.0.0/16 (10.0.0.0 -> 10.0.255.255)
* Azure Pilot 172.31.0.0/16 (172.31.0.0 -> 172.31.255.255)
* London offise 192.168.15.0/24 (192.168.15.0 -> 192.168.15.255)
* New York office 192.168.20.0/24 (192.168.20.0 -> 192.168.20.255)
* Seattle office 192.168.25.0/24 (192.168.25.0 -> 192.168.25.255)
* Google 10.128.0.0/9 (10.128.0.0 -> 10.255.255.255)

When planning IP address space, these should be avoided in this scenario.

* AWS VPC minimum /28 (16 IPs), maximum /16 (65536 IPs)
* Avoid common ranges - from 10.0 up to 10.10. Recommendations to use 10.16.0.0.
* Deciding how many IP ranges is required, helps to think how many AWS regions the business will require. Be cautios! Think about the highest number of regions the business could opperate and a add few as buffer.
* Suggestion is to have at least two ranges which can be used in each region, in each AWS account.

Animals4Life example doesn't have a clear number of regions that the business will operate, hence we can make assumptions:
* Maximum number of regions the business will use is three in US, one in Europe and one in Australia. Because we want to have two ranges in each regions then it is 10.
* Assume 4 AWS accounts, which means 40 IP ranges.
* We can use the whole range from 10.16.0.0 up to 10.127.0.0 (10.128.0.0 is Google).

## VPC Sizing and Structure - PART2

![image](https://github.com/MrR0807/Notes/assets/24605837/db3792e6-ce90-409b-9662-cc15be4fff1e)

When deciding the size of VPC, these questions need to be answered:
* How many subnets will you need?
* How many IPs total? How many per subnet?

**NOTE!** VPC services run from subnets, not directly from VPC.
**NOTE!** Subnet is located in one Availability Zone.

Steps to answer how many subnets you need:
* Decide how many Availability Zones your VPC will use. By default, pick 3 + 1 spare, because it will work in almost any region.
* Usually within a VPC you'll have tiers (e.g. Web, App, Database). By default, pick 3 + 1 spare.

This leads to total of 16 subnets. If we chose /16 VPC we have to split into 16 subnets = /20.

**NOTE!** Every increase in CIDR block creates two networks. /16 -> /18 creates 4 smaller networks.

![image](https://github.com/MrR0807/Notes/assets/24605837/8157fde5-38a3-4dc7-8ea0-440799dab97b)

 So for Animals4life we could split:
 * 10.16 (US1)
 * 10.32 (US2)
 * 10.48 (US3)
 * 10.64 (EU)
 * 10.80 (Australia)

The layout then:
* 10.16 US1, General Acc, VPC1
* 10.17 US1, General Acc, VPC2
* 10.18 US1, General Acc, VPC3
* 10.19 US1, General Acc, VPC4
* 10.20 US1, Prod Acc, VPC1
* 10.21 US1, Prod Acc, VPC2
* 10.22 US1, Prod Acc, VPC3
* 10.23 US1, Prod Acc, VPC4
* 10.24 US1, Dev Acc, VPC1
* 10.25 US1, Dev Acc, VPC2
* 10.26 US1, Dev Acc, VPC3
* 10.27 US1, Dev Acc, VPC4
* 10.28 US1, Reserved, VPC1
* 10.29 US1, Reserved, VPC2
* 10.30 US1, Reserved, VPC3
* 10.31 US1, Reserved, VPC4
* 10.32 US2, General Acc, VPC1
* 10.33 US2, General Acc, VPC2
* 10.34 US2, General Acc, VPC3
* 10.35 US2, General Acc, VPC4
...

In the end, that means that we assign /16 split per account, which is later on split into subnets /20, which gives us 4091 IPs per subnet.

## Custom VPCs - PART1 - THEORY

![image](https://github.com/MrR0807/Notes/assets/24605837/d18e6c31-37b0-47d5-a9c0-573d911d2464)

* Regional Service - All AZs in the region.
* Isolated network.
* Nothing IN or OUT without explicit configuration.
* Hybrid Networking - other cloud & on-premises can connect to your VPC.
* When creating a VPC, you have the option of picking default or dedicated tenancy. This controls whether the resources created inside the VPC are provisioned on shared hardware or dedicated hardware. If you chose the latter, then you're locked in. If you pick the former, then you can choose per service later on whether on dedicated or shared.
* VPC can use IPv4 private and public IPs.
* VPC is allocated one mandatory primary private IPv4 CIDR block. This primary block has a min/max restrictions - /28 (16 IPs) to /16 (65536 IPs).
* You can add secondary IPv4 CIDR block after creation (there is a maximum of 5 of such blocks, can be increased with a ticket).
* Optional single assigned IPv6 /56 CIDR block.
* Have fully feature DNS. Provided by Route53.
* VPC address is Base IP + 2, e.g. 10.0.0.0 then DNS address is 10.0.0.2.
* There are two properties that are important. If anything is not working with DNS, these settings should be checked first:
  * `enableDnsHostnames` - gives instances DNS Names
  * `enableDnsSupport` - enables DNS resolution in VPC.

## Custom VPCs - PART2

What is created at the end of the lesson:
![image](https://github.com/MrR0807/Notes/assets/24605837/fe03fecf-73ac-4d79-bb51-6d39070ba693)

**NOTE!** Do not forget to enable DNS Hostnames in VPC, by going into VPC -> Edit -> Enable DNS Hostnames.

## VPC Subnets

* AZ resilient.
* **EXAM NOTE!** One subnet in one AZ. It can never be in more than one.
* By default uses IPv4 CIDR and is allocated a subset of VPC CIDR.
* **EXAM NOTE!** Subnet CIDR cannot overlap with any other subnets in that VPC.
* Subnet can have allocated IPv6 CIDR as long as VPC is has it enabled.
* Subnets can communicate with other subnets in the same VPC (by default).
* There are 5 IP addresses within every VPC subnet that you can't use whatever the size of subnet.
  * Example, 10.16.16.0/20 (10.16.16.0 -> 10.16.31.225).
  * First unsuable IP address is 10.16.16.0 - Network Address. This isn't specific to AWS, but a case for any other IP networks.
  * Network+1 10.16.16.1 - VPC Router.
  * Network+2 10.16.16.2 - Reserved DNS.
  * Network+3 10.16.16.3 - Reserved Future Use.
  * Broadcast Address 10.16.31.255 (Last IP in subnet).
* VPC has a configuration object applied to it called DHCP Options Set. DHCP - dynamic host configuration protocol. How compute devices receive IP addresses automatically. One DHCP options set applied to a VPC at one time and this configuration flows through to subnets. **You cannot edit them. Only create new one and assigned to VPC**.

## VPC Routing, Internet Gateway & Bastion Hosts

VPC router:
* Every VPC has a VPC Router - Highly available.
* Network+1 address.
* Routes traffic between subnets.
* Controlled by route tables each subnet has one.
* A VPC has a Main route table - subnet default. When you associate a custom route table, the main route table is disassociated.
* A subnet can only have one route table associated with it, but a route table can be associated with many subnets.

Each packet has a source, destination and data. Destination is being matched against VPC router. The more specific the match, the higher the priority. For example, a packet can match 0.0.0.0/0, which means all internetet, then 10.16.0.0/16 which means a network and 10.16.125.125/32 which means one specific IP. In this example, the last IP will have the highest priority and will be matched.

**EXAM NOTE!** Route tables are attached to 0 or more subnets. A subnet has to have a route table. It's either the main route table of the VPC or a custom one that you've created. The route table controls what happens when data leaves the subnet or subnets. Local routes are always there, **uneditable**, and match the VPC CIDR range.

Internet Gateway (IGW):
* Region resilient gateway attached to a VPC.
* One to one relationship between IGW and VPC.
* Runs from within the AWS Public Zone.
* Gateways traffic between the VPC and the Internet or AWS Public Zone (S3, SQS, SNS etc).
* Managed by AWS.

Steps to expose a subnet to Internet:
1. Create IGW.
2. Attach IGW to VPC.
3. Create custom Route Table.
4. Associate RT.
5. Default Routes -> IGW.
6. Subnet allocate public IPv4 addresses.

### IPv4 Addresses with a IGW

Let's say we have an EC2 instance with a private IPv4 address, an IGW and a server in public Internet. EC2 instance has an address of 10.16.16.20. Let's say EC2 gets a public address of 43.250.192.20. **The trick is that this public IP address does not "touch" EC2 instance. In other words, it is not configured at OS level, which would make EC2 instance aware of its public IP address. The public address is just an entry in IGW, which associates private IP address with public address**.

**EXAM NOTE!** Do not follow for any exam questions where it tries to convince you to assign the public IPv4 address of an EC2 instance directly to operating system. In case of IPv6 all addresses are publicly routable.

![image](https://github.com/MrR0807/Notes/assets/24605837/c0fac242-8fbd-4041-989f-580b722e2ee7)

![image](https://github.com/MrR0807/Notes/assets/24605837/57fe6f2d-65ae-4dd2-9c25-8b180670c689)

### Bastion Host

* An instance in a public subnet.
* Incoming management connections arrive there. Then access internal VPC resources.
* Bastions are used as entry points for private-only VPCs or management point.

## Stateful vs Stateless Firewalls

Stateless needs both ingress and egress rules. As well as allowing for full range of ephemeral ports. Stateful firewalls automatically allow egress/ingress.

## Network Access Control Lists (NACLs)

* NACL are associated with subnets. Every subnet has an associated network ACL. It filters data as it crosses the boundary of that subnet.
* **NOTE!** Connections within same subnet are not affected by NACL.
* Each NACL contains two rule sets - inbound and outbound.
* NACL are stateless.
* Rules are processed in **order, from lowest rule number first**. Once a match occurs, processing stops. `*` is an implicit DENY if nothing else matches.
* If you have an allow rule and deny rule which match the same traffic, but if the deny rule comes first, then the allow rule might never be processed.

![image](https://github.com/MrR0807/Notes/assets/24605837/a36ef7cc-2b4c-4d82-ab96-2b12145d324c)

A VPC is created with default NACL. Inbound and outbound rules have the implict deny and allow all rule. Because the allow has a rule number, it is evaluated first, hence all traffic is allowed.

![image](https://github.com/MrR0807/Notes/assets/24605837/3eb959d2-3d4b-440e-929d-cff125b0289f)

Custom NACLs can be created for a specific VPC and are initially associated with NO subnets. They have explicit deny on everything.

### Summary

* Stateless.
* Only impacts data crossing subnet boundary.
* Work only with IPs/CIDR, ports and protocols - no logical resources.
* Network ACLs are unique in terms of being able to explicitly deny certain IP ranges.
* NACLs cannot be assigned to AWS resources - only subnets.
* Used together with Security Groups to add explicit Deny (Bad IPs/Nets).
* Each subnet can have only one NACL (default or custom).
* A single NACL can be associated with **many subnets**.

## Security Groups (SG)

* Stateful - detect response traffic automatically.
* Allowed (IN or OUT) request = allowed response.
* **There is no explicit deny**. Only allow or implicit deny. Hence cannot be used to block specific bad actors.
* Supports IP/CIDR and logical resources, including other security groups and itself.
* SG are not attached to instances, but to ENIs (elastic network interface). **EXAM NOTE!** SG are attached to network interfaces.

A network interface, often abbreviated as NIC (Network Interface Card) or simply as "interface," is a hardware component or software abstraction that enables a device to connect to a network. It serves as the intermediary between the device's operating system and the physical network medium (such as Ethernet, Wi-Fi, or cellular networks).
In the context of computer networking, a network interface can refer to:
* Physical Network Interface: This is a physical component of a device, such as a network adapter or network interface card (NIC). It could be an Ethernet port, a Wi-Fi adapter, or a cellular modem. Physical network interfaces typically have unique hardware addresses known as MAC (Media Access Control) addresses.
* Virtual Network Interface: In virtualized environments or software-defined networking, a virtual network interface can be created to provide network connectivity to virtual machines or containers. These virtual interfaces often emulate the behavior of physical network interfaces and can be configured with their own IP addresses, MAC addresses, and network settings.

Elastic Network Interface (ENI) is a **virtual network interface** that you can attach to an instance in a Virtual Private Cloud (VPC). It provides networking capabilities to the instance, enabling it to communicate with other resources in the VPC, the internet, and other AWS services.

For example, say we have two subnets - web and app. Web is public, while app is private. Bob calls an application which is hosted in web subnet. Web subnet configures SG to accept inbound traffic from 0.0.0.0/0 to 443 porrt. Web application in turn calls app application in app subnet. In this case, app subnet would have to define a SG which would allow a trafic from web subnet. It could do so via IP/CIDR ranges. But SG functionality allows to reference another SG, which is attached to web subnet. This way, any machine which has attached web SG, can call app subnet applications. 

![image](https://github.com/MrR0807/Notes/assets/24605837/db1d9016-2630-4757-b3e7-f98e94f6b921)

Logical references allow self referencing. When you self reference in SG it allows all applications using said SG to communicate between themselves.

![image](https://github.com/MrR0807/Notes/assets/24605837/fa6dcb96-00d6-4931-80ce-106af0cc8076)

## Network Address Translation (NAT) & NAT Gateway - PART1

* A set of processes - remapping SRC (source) or DST (destination) IPs.
* Internet Gateway provides a static NAT. Allocate public IP addresses to certain services and does converstion between private and public IP addresses.
* IP masquerading - hiding CIDR Blocks behind one IP.
* Gives private CIDR range **outgoing** internet access. Outgoing is crucial, because many internap IPs are using one IP, which does not work for incoming traffic.
* Historically there are two ways to provide NAT functionality. EC2 could be configured to use NAT and NAT as a service.
* If it wasn't in AWS, NAT gateway is how your internet router at home works. It provides one public address for many IPs inside your home network.
* NAT Gateways need to run from a public subnet.
* They utilise Elastic IPs. The one service that does.
* AZ resilient services.
* For every AZ you need one NAT gateway and one route table to point to it.
* **EXAM NOTE!** Priced a base price and then for processed data.

As per below, we can configure App subnet default route to point to NAT Gateway, which is hosted in Web subnet, which has assigned public IP address (but it is not really public address as per IGW section, just a record in IGW table that NAT gateway IP is associated with public IP). In Web subnet, default route is Internet Gateway.

If you want to give **private instances**, outgoing access to the internet and the AWS public zone services such as S3, then you need **both the NAT gateway and Internet Gateway**. If you have public instances, then only IGW is required.

![image](https://github.com/MrR0807/Notes/assets/24605837/0a80634d-3032-4161-a006-61fcb21c4305)

## Network Address Translation (NAT) & NAT Gateway - PART2

![image](https://github.com/MrR0807/Notes/assets/24605837/c5c3ba1e-2dd5-4064-8b16-87ac9c617422)

**EXAM NOTE!** You cannot associate Security Group with NAT gateway.

With IPv6 NAT isn't required. **NAT Gateways don't work with IPv6**.

# ELASTIC COMPUTE CLOUD (EC2) BASICS

## Virtualization 101

* First iteration of hypervisor was making binary transformations coming from guest OS to hardware via software. It was super slow.
* Second iteration was Para-Virtualization. Still same approach, but changes were made to OS (e.g. Linux) and OS is aware that it is virtualized. There are areas in OS where instead of privileges calls, they are made to be user calls to hardware, which instead of calling the hardware, they call hypervisor. 
* Third iteration was Hardware Assisted Virtualization. The hardware itself has become virtualization aware. The CPU contains specific instructions and capabilities so thatt the hypervisor can directly control and configure this support. What this means is when guest OS attempt to run any privileged instructions they're trapped by the CPU, which knows to expect them from these guest OS. However, these instructions are still directed to a hypervisor, but just this time by hardware. The downside that it requires a lot of CPU cycles and for intensive IO operations, for example, accessing network card still involves software.
* Final iteration is where a hardware device themselves become virtualization aware. Such as network cards. This process is called **SR-IOV** - single route IO virtualization. It allows for a single physical card to present itself as several mini cards. Because this is supported at hardware, they are fully unique almost standalone cards. **This means that no translation needs to happen**. In EC2 this is called **Enhanced Networking**.

## EC2 Architecture and Resilience

* EC2 instances are virtual machines.
* EC2 instances run on EC2 Hosts.
* EC2 hosts are either shared or dedicated.
* AZ resilient.

EC2 hosts have local hardware - CPU, Memory, local storage (Instance Store - temporary storage), storage networking, data networking. When instances are provisioned into a specific subnet within a VPC, whats actually happening is that a primary elastic network interface, is provisioned in a subnet, which maps to the physical hardware on the EC2 host. Instances can have multiple network interfaces, even in different subnets as long as they are in the same Availability Zone.

EC2 can mount block store (EBS). **EBS also runs in same AZ**. You can't access them cross zone.

**NOTE!** If you restart an instance it will stay on particular EC2 host. If you stop and start - it might move to a different EC2 host.

You cannot connect network interfaces or EBS storage located in one AZ to an EC2 instance, located in another.

![image](https://github.com/MrR0807/Notes/assets/24605837/ef2b13e3-3988-45b7-b674-23194bdf1811)

EC2 good for:
* Traditional OS+Application Compute.
* Long Running Compute.
* Server style applications (burst or steady-state).

In general always choose for EC2, unless you need specific need.

## EC2 Instance Types - PART1

Instances types influence:
* Raw CPU, Memory, Local Storage Capacity & Type.
* Resource Ratios.
* Storage and Data Network Bandwidth.
* System Architecture / Vendor (Intel, AMD etc).

EC2 Categories:
* General Purpose - Default - equal resource ratio.
* Compute Optimized - media processing, scientific modelling, gaming, machine learning.
* Memory Optimized - processing large in-memory datasets, some database workloads.
* Accelerated Computing - Hardware GPU, field programmable gate arrays (FPGAs).
* Storage Optimized - sequential and random IO - scale-out transactional databases, data warehousing, ElasticSearch, analytics workloads.

![image](https://github.com/MrR0807/Notes/assets/24605837/0939744e-6b50-488d-bc3a-17e802c2146b)

## EC2 Instance Types - PART2

| Categories            | Type                 | Details/Notes                                                                                        |
|-----------------------|----------------------|------------------------------------------------------------------------------------------------------|
| General Purpose       | A1, M6g              | Graviton (A1), Graviton 2 (M6g) ARM based processors. Efficient.                                     |
| General Purpose       | T3, T3a              | Burst Pool - Cheaper assuming nominal low levels of usage, with occasional Peaks.                    |
| General Purpose       | M5, M5a, M5n         | Steady state workload alternative to T3/3a - Intel / AMD Architecture.                               |
| Compute Optimized     | C5, C5n              | Media encoding, Scientific Modeling, Gaming Servers, General Machine Learning.                       |
| Memory Optimized      | R5, R5a              | Real time analytics, in-memory caches, certain DB applications.                                      |
| Memory Optimized      | X1, X1e              | Large scale in-memory applications. Lowest $ per GB memory in AWS.                                   |
| Memory Optimized      | High Memory (u-Xtb1) | Highest memory of all AWS instances.                                                                 |
| Memory Optimized      | z1d                  | Large memory and CPU - with directly attached NVMe.                                                  |
| Accelerated Computing | P3                   | GPU instances - parallel processing & machine learning.                                              |
| Accelerated Computing | G4                   | GPU Instances - machine learning inference and graphics intensive.                                   |
| Accelerated Computing | F1                   | Field Programmable Gate Arrays - genomics, financial analysis, big data.                             |
| Accelerated Computing | Inf1                 | Machine Learning - recommendation, forecasting, analysis, voice, conversation.                       |
| Storage Optimized     | I3/I3en              | Local high performance SSD (NVMe) - NoSQL Databases, warehousing, analytics.                         |
| Storage Optimized     | D2                   | Dense Storage (HDD) - data warehousing, Hadoop, DFS, data processing - lowest price disk throughput. |
| Storage Optimized     | H1                   | High Throughput, balance CPU/Memory, HDFS, File systems, Apache Kafka, Big data.                     |

## EC2 SSH vs EC2 Instance Connect

## Storage Refresher

Key Terms:
* Direct (local) attached Storage - Storage on the EC2 Host. Called Instance Store. This is Ephemeral Storage.
* Network attached Storage - Volumes delivered over the networks (EBS). This is persistent storage.

Three main categories of storage:
* Block - volume presented to the OS as a collection of blocks. **Moutable. Bootable**. Blocks have no structure. It is up for OS to create a file system.
* File - presented as a file share. Has structure. **Moutable. Not bootable**.
* Object storage - collection of objects. Not mountable. Not bootable.

Storage Performance:
* IO (block) size.
* IOPS - measures the number of IO operations the storage system can support in a second.
* Throughput - the amount of data that can be transferred in a given second (generally in MB/s).

IO (block) size x IOPS = Throughput

But this not always translates directly. For example, 16KB x 100 IOPS = 1.6 MB/s. But 1MB x 100 IOPS != 100 MB/s. Because sometimes, by increasing block size, IOPS slow down. Also, there are caps on throughput.

## Elastic Block Store (EBS) Service Architecture

* Block Storage - raw disk allocations (volume) - Can be encrypted using KMS.
* When you attach a block storage onto EC2 they can create a file system on this device.
* **AZ resilient**.
* Usually attach to one EC2 instance (or other service) over a storage network.
* Lifecycle not linked to one instance. Persistent.
* Can snapshot into S3. Create volume from snapshot (migrate between AZs).
* There are different physical storage types, different sizes, different performance profiles.
* Billed based on GB/month (and in some cases performance).
* No cross AZ attachements between EBS and services (e.g. EC2).

## EBS Volume Types - General Purpose

### General Purpose SSD - GP2.

An IO credit is 16 KB. IOPS assumes 16 KB. 1 IOPS is 1 IO in 1 second. If you transfer 160KB per second, that is 10 credits.

Volumes can be as small as 1 GB or as large as 16TB.

IO Credit Bucket Capacity is 5.4 million IO Credits and it fills at rate of Baseline Performance.

Baseline Performance - every volume has a baseline performance based on its size with a minimum. So streaming into the bucket at all times is a 100 IO credits per second refill rate. The actual baseline you get in GP2 depends on the volume size. You get three IO credits per second per GB of volume size. A 100 GB volume gets 300 IO credits per second. Anything below 33.33 GB gets a 100 IO Credits minimum. This is only for volumes up to 1TB.

By default, GP2 can burst up to 3000 IOPS.

All volumes get an initial 5.4 million IO credits. Which means 30 minutes of 3000 IOPS (without calculating the fill up rate).

Volumes larger than 1TB are given equal or exceeding the burst rate of 3000. They will always achieve their baseline performance as standard. The maximum IO per second for GP2 is 16000. Any volumes above 5.33 recurring TB in size, gets this.

![image](https://github.com/MrR0807/Notes/assets/24605837/520f075e-eded-46fe-9110-9dffbdae8773)

### GP3

It removes the credit bucket architecture of GP2. Every GP3 regardless of size starts with a standard 3000 IOPS and can transfer 125 MB per second. Volumes can range from 1GB to 16TB. If you need more performance you can pay extra for up to 16000 IOPS or 1000 MB/s. GP3 is 4x faster max throughput vs GP2 - 1000 MB/s vs 250 MB/s. Just be aware that IOPS do not automatically scale with volume size.

## EBS Volume Types - Provisioned IOPS

Provisioned IOPS SSD (io1/2). There are three types of provisioned IOPS SSD:
* io1 - up to 64000 IOPS per volume (4x GP2/3). Up to 1000 MB/s. Volume sizes from 4GB - 16TB. 50 IOPS/GB Max.
* io2 - up to 64000 IOPS per volume (4x GP2/3). Up to 1000 MB/s. Volume sizes from 4GB - 16TB. 500 IOPS/GB Max.
* io2 Block Express - up to 256000 IOPS per volume. Up to 4000 MB/s. Volume sizes from 4GB - 64TB. 1000 IOPS/GB Max.

A common factor among all of them is that **IOPS are configurable independent of the size of the volume (but still have caps per GB as per above)** and they're designed for super high performance situations. 

There is also per instance performance betweent EBS and EC2 which is influenced by:
* The type of volume. Different volumes have different per instance performance.
* The type of the instance.
* The size of the instance. The most modern and the largest instances support the highest levels of performance.

 Furthermore, the maximum per instance will be bigger than one volume can support, hence you can have multiple volumes attached. The instance maximums:
 * io1 - 260000 IOPS & 7500 MB/s.
 * io2 - 160000 IOPS & 4750 MB/s.
 * io2 Block Express - 260000 IOPS & 7500 MB/s.

**EXAM NOTE!** Do not remember the numbers, but have a feel how much they differ from gp2/3 volumes. 

## EBS Volume Types - HDD-Based

There are two types of HDD Based storage in EBS:
* st1 - throughput optimized. 125 GB - 16 TB. Max 500 IOPS (block sizes are 1MB). Max 500 MB/s. 40MB/s/TB base. Good for data warehouses. Data warehouses, log processing.
* sc2 - cold HDD. Max 250 IOPS (1MB). 125 GB - 16 TB. Max 250 MB/s. 12MB/s/TB base. 80 MB/s/TB burst.

HDD is not good for random access, but for sequental. HDD works like GP2 - bucket credit. But credits are per MB/s instead of IOPS.

## Instance Store Volumes - Architecture

* Block storage devices.
* Physically connected to one EC2 host.
* Instances on that host can access them.
* Highest storage performance in AWS. For example using D3 instance (storage optimized) provides 4.6 GB/s throughput. I3 = 16 GB/s.
* Included in instance price.
* **EXAM NOTE!** Attached at launch. You cannot attach them later.
* Data is lost when instances moves (stop, start), resized or hardware failure.

## Choosing between the EC2 Instance Store and EBS

**Remember all of this. Very important**.

* Persistance = EBS (avoid instance store).
* Resilience = EBS (avoid instance store).
* Storage isolated from instace lifecycle = EBS.
* Resilience built into application, then it depends.
* High performance needs - it depends.
* Super high performance needs - instance store.
* Cost - instance store (included into instance price).
* **EXAM NOTE!** If you see a question about cost efficiency and need to use EBS, then choose ST1 and SC1.
* **EXAM NOTE!** If the question mentions throughput or streaming then you should ST1.
* **EXAM NOTE!** If the question mentions boot volume - then you cannot use either ST1 or SC1.
* **EXAM NOTE!** GP2/3 up to 16000 IOPS. IO1/2 up to 64000 IOPS (*256000).
* **EXAM NOTE!** EBS + RAID 0. You can take lots of individual EBS volumes, and you can create a RAID 0 set from those EBS volumes. And that RAID 0 set then gets up to the combined performance of all of the individual volumes. However, this is up to 260000 IOPS, because this is the maximum possible IOPS per instance.

## Snapshots, Restore & Fast Snapshot Restore (FSR)

* Snapshots are incremental volume copies to S3.
* The first is a full copy of data on the volume. EBS performance is not impacted during snapshot.
* Future snapshots are incremental.
* EBS volume can be created (restored) from snapshots.
* When EBS is provisioned without snapshot - it is available immediately. When EBS is restored, restore is happening gradually in the background and it takes some time. If you try to read data that has not been restored, it will pull the data from S3, but that achieves lower level of performance.
* Fast Snapshot Restore - immediately pulls data from S3. You can have up to 50 FSR per region. 1 snapshot restored to 4 different AZ is counted as 4 FSR. This feature has associated cost.
* Costs are GB/month. And snapshot costs are tied to increment size. E.g. initial snapshot is 10 GB, then it is priced as 10 GB. Next incremental snapshot is 2 GB, then snapshot is priced as 2 GB.

## EBS Volumes - PART1

Linux commands to run inside EC2:
* `lsblk` - list all block devices connected to this instance.
* `sudo file -s /dev/xvdf` - check whether there are file systems on this block device. If you see `/dev/xvdf: data` then there isn't any filesystem.
* `sudo mkfs -t xfs /dev/xvdf` - because there is no file system, you have to create one as you can only mount file systems under linux. You can mount a file system to a mount point which is a directory.
* `sudo mkdir /ebstest` - create directory.
* `sudo mount /dev/xvdf /ebstest` - mount to directory the volume.
* `df -k` - shows all file systems on this instance.
* `sudo blkid` - will list unique IDs of all volumes attached to the instance.
* `sudo nano /etc/fstab` - configuration file which describes which volumes are automatically mounted.
  * UUID=<uuid> /ebstest xfs defaults,nofail
* `sudo mount -a` - mounts all the volumes which are defined in fstab file.

## EBS Encryption

EBS uses KMS to encrypt the data. You can use default AWS kms/ebs key or your own. KMS provides DEK keys to EBS and during encryption, the DEK key is stored together. When the volume is first used, either mounted on an EC2 instance by you or when an instance is launched, then EBS asks KMS to decrypt the data encryption key, that's used just for this one volume. That key is loaded into the memory of the EC2 host which will be using it.

If a snapshot is made, then the same encryption key is used. **It doesn't cost anything to use**.

**EXAM NOTES!**
* Accounts can be set to encrypt by default - default KMS Key. Otherwise choose a KMS Key manually each and every time.
* Each volume uses 1 unique DEK. Every time you create a new volume from scratch - it gives you new DEK.
* Snapshots & future volumes use the same DEK.
* Can't change a volume to NOT be encrypted.
* OS isn't aware of the encryption - no performance loss. 

## Network Interfaces, Instance IPs and DNS

EC2 instance always start with one network interface called ENI - elastic network interface. This is primary ENI. Optionally you can attach one or more secondary ENIs. EC2 network interfaces can be in different subnets, but must be in the same AZ. For example if you launch EC2 with security group, that SG is on ENI not on the instance.

Primary Network interface has:
* MAC address.
* Primary IPv4 Private IP.
* 0 or more secondary IPs.
* 0 or 1 public IPv4 address.
* 0 or more IPv6 addresses.
* Security groups.
* You can enable source/destination check. It will discard traffic if source/destination do not match.

Secondary Network Interfaces have the same properties as primary, but additionally you can detach from one EC2 instance and move them to other EC2 instances.

Exploring ENI in detail with an example:

* MAC address.
* Primary IPv4 Private IP - let's assume that this instance receives a primary IP of 10.16.0.10. This is static and doesn't change for the lifetime of the instance. An instance is given a DNS name that's associated with this private address: ip-10-16-0-10.ec2.internal. This IP is only resolvable inside the VPC.
* 0 or more secondary IPs.
* 0 or 1 public IPv4 address - allocated 3.89.7.136 public IP address. It is a dynamic, not fixed. If you stop and start an instance, it's public IP address will change. Also allocated a public DNS name - ec2-3-89-7-136.compute-1.amazonaws.com. What's special about this public DNS name is that inside the VPC, it will resolve to the primary private IPv4 address. This public IPv4 address is not attached to the instance or ENI, but is stored in Internet Gateway. Inside VPC DNS always resolves to private IP, outside VPC - public IP for that instance.
* 1 elastic IP per private IPv4 address - Elastic IPs are allocated to your AWS account. When you allocate an elastic IP, you can associate the elastic IP with a private IP either on the primary interface or a secondary interface. If you do associate it with the primary interface, then as soon as you do that, the normal (non elastic IP version) is removed and replaced by elastic IP. **EXAM NOTE!** If an instance has a non elastic public IP and you assign an elastic IP and then remove it, is there any way to get that original IP back? The answer is no, there is not. It gets a new IP address, but not the same as before.
* 0 or more IPv6 addresses.
* Security groups.
* You can enable source/destination check. It will discard traffic if source/destination do not match.

In short, an instance have one or more network interfaces (a primary and optionally secondaries) and then for each network interface, it can have:
* primary private IP address.
* secondary private IP addresses.
* optionally, one public IPv4 address.
* optionally, one or more elastic IP addresses.

**EXAM NOTES!**
* Secondary ENI + MAC = Licensing. A lot of legacy licesing is per MAC address as it was viewed as static. Because EC2 is a virtualized environment, we can swap and change elastic network interfaces. And so if you provision a secondary elastic network interface on an instance, and use that secondary network interface MAC address for licesing, you can detach that secondary interface and attach to a new instance. Move the license between EC2 instances.
* Different Security Groups for multiple interfaces (because SGs are attached to interfaces). If you need different rules so different security groups, for different IPs then you need multiple elastic network interfaces.
* **OS does not see public IPv4**.
* IPv4 public IPs are dynamic. Stop & Start = Change. To avoid this, you need to allocate elastic IP address.
* Inside VPC, the public DNS resolves to the private IP. Outside the VPC, it will resolve to the public IP address.

## Amazon Machine Images (AMI)

* AMI can be used to launch EC2 instance.
* AMIs can be provided by AWS or Community provided.
* AMIs can be found in Marketplace (can include commercial software).
* AMIs are regional.
* AMIs control permissions. By default an AMI is set so that only your account can use it. You can set an AMI to be public or add specific AWS accounts onto that AMI.
* You can create an AMI from an EC2 instance you want to template.

**EXAM NOTES!**
* AMI = One Region. Only works in that one region. If you create a particular AMI, it can only be used in that region.
* AMI baking - taking an EC2 instance, installing all of the software, doing all the configuration and then baking all of that into an AMI.
* AMI is immutable.
* Can be copied between regions (includes its snapshots).
* AMI permissiosn default to only your account.
* AMI costs per EBS snapshots.

## EC2 Purchase Options

* On-Demand - default. Instances are isolated but multiple customer instances run on shared hardware. Per-second billing while an instance is running. Associated resources such as storage consume capacity, so bill, regardless of instance state.
  * No interruption.
  * No capacity reservation.
  * Predictable pricing.
  * No upfront costs.
  * No discount.
  * Short term workloads.
  * Unknown workloads.
  * Apps which can't be interrupted.
* Spot. AWS selling unused EC2 host capacity for up to 90% discount - the spot price is based on the spare capacity at a given time. Each AWS user can setup their upper limit how much they want to pay for spot instance. If say customer A max is 10$ and customer B - 20$, then when spot instance price changes upwards, then customer A will be removed first from spot instances to make room for e.g. on-demand instaces.
  * Non time critical.
  * Anything which can be rerun.
  * Bursty Capacity needs.
  * Cost sensitive workloads.
  * Anything which is stateless.
* Reserved. 
  * Reservation is purchased for a particular type of instance and locked in AZ or to a region.
  * Unused reservation still billed.
  * Partial coverage of larger instance. For example, you have reserved t3.medium, but provisioned t3.large. You get a discount for t3.large.
  * You can commit to either 1 year or 3 years.
  * You can agree with no-upfront and get a reduced per second fee. You have the ability to pay all upfront - no per second fee. You pay partial upfront and get lower per second fee.
* Dedicated host. You pay for the host and have various instance types run in those hosts.
  * Has a host affinity feature, which means when an instance is stopped it can restart on the same host.
  * **EXAM NOTE!** The true reason to use these hosts are per socket/core licensing requirements.
  * You have to monitor resource consumption, because you might run out of resources.
* Dedicated Instances. Your instance run on EC2 host with other instances of yours. No other customers use same hardware. You don't pay for the host or share the host. Extra charges for instances, but dedicated hardware.

## Reserved Instances - the rest

Scheduled Reserved Instances:
* ideal for long term usage which doesn't run constantly (batch processing daily for 5 hours starting at 23:00).
* You can only use that instance during that time window.
* Does not support all instance types and minimum is 1200 hours per year & 1 year term minimum.

Capacity Reservations. Sometimes AWS might run out of capacity due to failure. There is a priority list which get the compute:
* Reserved instances.
* On demand.
* Spot instances.

Capacity reservation is different from reserved instance purchase. There are two components:
* billing.
* capacity.

![image](https://github.com/MrR0807/Notes/assets/24605837/79bfd9f4-594f-4bdf-9096-0f8f6a73191f)

EC2 savings plan is a feature like reserved instance, but instead of focusing on a particular type of instance in an availability zone or a region, you're making a one or three year commitment to AWS in terms of hourly spending. You might make a commitment that you're going to spend 20 US dollars per hour for one or three years. And in exchange for doing that you get a reduction on the amount that you're paying for resources.
Two plans:
* A reservation of general compute $ amounts ($20 per hour for 3 years)
* A specific EC2 savings plan - flexilibyt on size & OS.
* If you go above your plan - you begin to consume on-demand prices.

## Instance Status Checks & Auto Recovery

Instance status checks. The 2/2 checks are separated into:
* System status check - a failure of the system status check could indicate one of a few major problems. Loss of power, network connection, host issues.
* Instance status check - a failure could indicate a corrupt file system, incorrect networking or OS issues.

You either manually make sure that faulty instances are restarted or let AWS managed it. One of such feature is Auto Recovery.

Auto Recovery moves the instance to a new host, start it up with exactly the same configuration. However, there are limitations to this feature, e.g. cannot recover if EC2 contains instance store. Only works with EBS. Not really recommended.

## Horizontal & Vertical Scaling

Horizontal scaling has downside of sessions. You need to maintain off-host sessions.

## Instance Metadata

* EC2 service provides data to Instances.
* Accessible inside all instances.
* **EXAM NOTE!** Data is accessed via `http://169.254.169.254/latest/meta-data/`
* You can get metadata about instance like host name, events, security groups - all the information about the environment that the instance is in.
* Network metadata - instance cannot see its own public IPv4 for example, the instance metadata can be used by the application running on that instance to get access to that information.
* Authentication metadata.
* **EXAM NOTE!** Metadata services has no authentication or encryption. Anyone who can access instance shell can access the metadata.

# CONTAINERS & ECS

## Introduction to Containers

## ECS - Concepts

* Container Definition - define images and ports.
* A Task Definition is a collection of 1 or more container configurations. Some Tasks may need only one container, while other Tasks may need 2 or more potentially linked containers running concurrently. The Task definition allows you to specify which Docker image to use, which ports to expose, how much CPU and memory to allot, how to collect logs, and define environment variables.
  * Task Definition stores task roles - a task role is an IAM role that a task can assume. When the task assumes that role, it gains temporary credentials, which can be used within the task to interact with AWS resources. **Task roles are the best practice way of giving containers within ECS permissions to access AWS products and services**.
* A Service is used to guarantee that you always have some number of Tasks running at all times. If a Task's container exits due to an error, or the underlying EC2 instance fails and is replaced, the ECS Service will replace the failed Task. This is why we create Clusters so that the Service has plenty of resources in terms of CPU, Memory and Network ports to use. To us it doesn't really matter which instance Tasks run on so long as they run. A Service configuration references a Task definition. A Service is responsible for creating Tasks.

## ECS - Cluster Mode

ECS runs in either of two modes:
* EC2 mode which uses EC2 instances as container hosts.
* Fargate mode - serverless way of running docker containers.

Using EC2 mode we start with the ECS management components so these handle high-level tasks like scheduling, orchestration, cluster management and the placement engine which handles where to run containers. Because EC2 mode runs within VPC it benefits from the multiple AZ setup. You can configure to scale up EC2 machines with Auto Scalling Groups.

In EC2 mode you are responsible for these EC2 instances. ECS provisions these EC2 container hosts, but expecations is that you will manage them. Generally through the ECS tooling. You need to worry about capacity and availability. Even if you don't have running tasks within EC2 hosts, you still pay for those resources, just like normal EC2.

With Fargate you don't manage servers. Important to understand - tasks and services are actually running from the shared infrastructure platform, and then they're injected into your VPC. They're given network interfaces inside a VPC, and it's using these network interfaces in that VPC that you can access them. So if VPC is configured to use public subnets, which automatically allocate an IPv4 address, then tasks and services can be given public IPv4 addressing.

**EXAM NOTE!**

When to use EC2 vs ECS (EC2) vs Fargate:
* If you use containers pick ECS.
* You'd pick EC2 mode for large workloads and you're price conscious (because you get to control all pricing related stuff of EC2 - reserved/spot/reserved).
* You'd pick Fargate for large workloads and you're overhead conscious.
* Small/Burst workloads - Fargate.
* Batch/Periodic workloads - Fargate.

## Elastic Container Registry (ECR)

* Each AWS account has a public and private registry.
* Public registry only Read/Write permissions are required while Read Only are by default.
* Private registry requires permissions for either Read/Write or Read Only.

Features:
* Integrated with IAM.
* Image scanning: offer basic and enhanced (uses inspector and scans for OS issues or software packages).
* Provides Metrics and CloudTrail.
* Provides replication cross-region and cross-account.

## Elastic Kubernetes Service (EKS) 101

* Kubernetes Control Plane scales and runs on multiple AZs.
* EKS integrates with other AWS products e.g. ECR, ELB, IAM, VPC.
* EKS cluster = EKS Control Plane & EKS Nodes are managed by AWS.
* etcd distributed accross multiple AZs.
* Nodes handling modes:
  * Self Managed nodes. These are EC2 instances which you managed.
  * Managed node groups. These are still EC2, but the product handles the provisioning and life cycle management.
  * Fargate pods.

**NOTE!** Keep in mind when deciding between self-managed, managed node groups or fargate is based on your requirements. Limitations are provided [here](https://docs.aws.amazon.com/eks/latest/userguide/eks-compute.html).

![image](https://github.com/MrR0807/Notes/assets/24605837/a66c5ba6-0bb2-4c37-b19a-61aefdedcc02)

# Advanced EC2

## Bootstrapping EC2 using User Data

* Bootstrapping allows EC2 Build Automation. Rarther than relying on your own custom AMI, it allows you to direct an EC2 instance to do something when launched.
* With EC2, bootstrapping is enabled using EC2 user data and this is injected into the instance in the same way that metadata is.
* Accessed via metadata api: `http://169.254.169.254/latest/user-data`.
* Anything in User Data is executed by the instance OS.
* Only once and only on launch.
* If something is not correctly configured via user data, the health checks will pass (unless you delete massive amount of OS). You have to make sure that the state is correct.
* User data is not secure.
* User data is limited to 16 kb in size. For anything more complex than that, you would need to pass in a script which downloads required data.
* User data can be modified - shutdown instance, modify it and restart.

**EXAM NOTE!** It is required to know how fast from boot time to service time (in other words, when the instance is ready to serve). From AMI - in minutes. If you're solely using bootstraping, it might take hours. AMI baking is also measured in minutes. The most optimal way is combining AMI baking and boostraping.

**NOTE!** `/var/log` contains logs for debuging user data failures: `cloud-init-output.log`; `cloud-init.log` . The formal will show you all of the actual commands and outputs of those commands that were executed on this EC2.

## Enhanced Bootstrapping with CFN-INIT

`CloudFormation::Init` is way how you can pass any complex bootstrapping instructions into an EC2 instance. There is much moer powerful way to configure EC2 instance instead of user data - **cfn-init**:
* It is installed on AWS EC2 AMIs.
* A better way to describe it is calling it a configuration management system.
* User Data is procedural, run line by line. `cfn-init` is a desired state.
* `cfn-init` can install packages, update packages, configure groups, users. Download sources and extract them. Create files with certain permissions and ownerships.
* Commands to `cfn-init` are provided with directives via **Metadata** and **`AWS::CloudFormation::Init`** on a CFN resource.

![image](https://github.com/MrR0807/Notes/assets/24605837/2ec8563a-ea91-44bf-8e50-5012aa0565b8)

`cfn-init` communicates with CloudFormation in order to fill in the details about command variables. It also can be updated and `cfn-init` will make sure to reach desired state.

### CreationPolicy and Signals

CloudFormation in a way is dumb. If we'd create EC2 instance using User Data, then there is no clear way to make sure that EC2 is running correctly. That is where `CreationPolicy` works. Using CreationPolicy it goes through the same motions as before, but differently, waits for a signal from EC2 instance before moving it into completed state. `cfn-signal -e $? ...` communicates with CloudFormation and command `-e $?` check previous `cfn-init` command's status and sends it to CloudFormation.

## EC2 Instance Roles & Profile

EC2 instance role allows EC2 service to assume IAM role. In order to deliver those credentials into running application within EC2, there is an intermediate piece of architecture called InstanceProfile. InstanceProfile is a wrapper around an IAM role. The instance profile is the thing that allow the permissions to get inside the instance and it is attached to the EC2 instance.
Inside EC2 instance credentials are delivered via the instance metadata. The upside is that EC2 instance always makes sure that credentials are valid and always renewed.

## SSM Parameter Store

* Storage for configuration & secrets.
* Allows to store 3 different parameter value: String, StringList, SecureString (If you choose the SecureString parameter type when you create your parameter, Systems Manager uses AWS KMS to encrypt the parameter value). Use these values, you can store license codes, database strings, full configs and password.
* You can version.
* Can store plaintext and ciphertext values (integrates with KMS which allows you to encrypt parameters).
* Any changes that occur to any parameters can generate events (SQS).
* Supports hierarchy. For example instead of fetching a single parameter, you can fetch all parameters within that hierarchy.
  * `aws ssm get-parameters-by-path --path /mydb/` would fetch all.
  * `aws ssm get-parameters --names /mydb/dbpassword` would fetch only one.

SSM Parameter Store is a public service.

## System and Application Logging on EC2

* CloudWatch cannot capture data inside an EC2 Instance natively. The inside of an instance is opaque to CloudWatch and CloudWatch Logs by default.
* To provide visibility CloudWatch Agent is required.
* For CloudWatch Agent to function it requires configuration and permissions.
* CloudWatch Agent requires Agent configuration.
* In order to allow CloudWatch Metrics and Logs, IAM role is required to be attached to running EC2 instances, which subsequently gives permissios to anything running inside EC2.
* We can store Agent configuration in Parameter Store.

## EC2 Placement Groups

Normally when you launch EC2 instance its physical location is selected by AWS, placing it on whatever EC2 host makes the most sense.

There are 3 types of placement groups for EC2:
* Cluster - pack instances close together. When you create a cluster placement group, they are generally launched in the same rack, sometimes in the same host. Placement groups allow you to influence placement. All members of cluster group have direct connections to each other - fast bandwidth. They can achieve 10GB/s compared to usual 5GB/s. This has no resilience, because if rack/host fails - all applications fail.
* Spread - keep instances separated. This provides maximum amount of availability and resilience. Can span multiple AZs. Located on separate, isolated infrastructure racks. Limitation - 7 instances per AZ.
* Partition - groups of instances spread apart (for replicated and distributed applications). Have similar architecture to spread placement groups. Can be created accross multiple AZs. You specify partitions per AZ. Max is 7. Different from spread, you can launch as many instances as you need within partition. You can control to which partition you launch instances.

**EXAM NOTE!**
Cluster placement groups:
* You cannot span AZs with cluster placement group. Only one AZ - locked at launch.
* Can span VPC peers - but significantly impact performance.
* Requires a supported instance type.
* Use the same type of instance (not mandatory).
* Launch at the same time (not mandatory, very recommended).
* 10GB/s single stream performance.
* Use cases: performance, fast speeds, low latency.

Spread placement groups:
* Provides infrastructure isolation.
* Each instance runs from a different rack. Each rack has its own network and power source.
* Hard limit 7 instances per AZ.
* Not supported dedicated instances or hosts.
* Use case: small number of critical instances that need to be kept separated from each other.

Partition placement groups:
* 7 partitions per AZ.
* Instances can be placed in a specific partition or auto placed by AWS.
* Great for topology aware applications (HDFS, HBase, Cassandra).

## Dedicated Hosts

* EC2 Host dedicated to you in its entirety.
* Because you pay for a host, there are no charges for running instances.
* You can use on-demand or reserved options (same one or three years term).
* Comes with physical sockets and cores. Beneficial for licenses where costs are associated with sockets and cores.
* For example A1 instance type has 1 socket and 16 cores. You can run 16 medium, 8 large, 4 xlarge, 2 2xlarge and 1 4xlarge instance types. Most of dedicated hosts require you to set the number of instances types you'll run. You can mix and match.
* However, newer type instances which run Nitro Virtualization Platform (e.g. R5) allow you to mix and match different types. Have few medium, have few large etc.

Limitations:
* You cannot use RHEL, SUSE Linux or Windows AMIs.
* Amazon RDS instances not supported.
* You cannot utilise placement groups.
* Hosts can be shared with other organisation accounts.

## Enhanced Networking & EBS Optimized

* Enhanced Networking is a feature which aims to improve the overall performance of EC2 networking. Its a feature which is required for Cluster placement group. 
* Enhanced Networking uses a technique called SR-IOV - single root IO virtualization. In other words, physical network interface inside an EC2 host is aware of virtualization. In non-SR-IOV case, EC2 instances talk with single physical network interface, which means that host has to sit in the middle and controll you has accesses to the network interface. It is done in software, hence consumes CPU and is slower. With SR-IOV you are provided with logical network interfaces which are handle by physical card. Instances are use them as physical network cards. Furthermore, in this scenario network card no CPU overhead.
* Enhanced Networking has no charge.
* Higher IO and Lower Host CPU Usage.
* More Bandwidth.

![image](https://github.com/MrR0807/Notes/assets/24605837/3400b453-a888-416a-83be-e34a586576b0)

EBS Optimized instances. Whether EC2 is EBS optimised depends on an option that sets on a per instance basis. Historically network was shared between data networking and EBS storage networking. This resulted in contention and limited performance for both types of networking. Simply put, an instance being EBS optimized means that some stack optimizations have taken place and dedicated capacity has been provided for the instance for EBS usage. Most instances support and have enabled by default, no extra charge.

## R53 Public Hosted Zones

R53 Hosted Zone:
* Globally resilient service.
* A R53 Hosted Zone is a DNS DB for a domain e.g. animals4life.org.
* There is a monthly fee to host each hosted zone and a fee for the queries made against that hosted zone.
* In summary hosted zones are databases which are referenced via delegation using name server records. A hosted zone, when referenced in this way is authoritative for a domain.

R53 Public Hosted Zone:
* DNS Database (zone file) hosted by R53 (Public Name Servers).
* Accessible from the public internet & VPCs.
* Hosted on 4 R53 Name Servers specific for the zone.
* Inside a public hosted zone, you create resource records, which are the actual items of data, which DNS uses.
* You can use R53 to host zone files for externally registered domains.

![image](https://github.com/MrR0807/Notes/assets/24605837/8b2ca32d-06dd-4643-8d76-cd6dea944cd7)

## R53 Private Hosted Zones

* Works just like public Hosted Zone - its just not public.
* Instead of being public it is associated with VPCs in AWS.
* Only accessible within those VPCs.
* Split-view (overlapping public & private) for Public and Internal use with the same zone name. Split-view DNS is a process wherein the DNS server gives out a different response to the same DNS query, based on where the query came from.

![image](https://github.com/MrR0807/Notes/assets/24605837/5de0b316-98c7-4e6f-957b-3e2818b63a88)

**Split View example**

You have a VPC running on Amazon Workspace. To support some business applications a private hosted zone with some records inside it is hosted. The private hosted zone is associated with VPC1. Split view allows us to create a public part of the Hosted Zone with subset of records which are available.

![image](https://github.com/MrR0807/Notes/assets/24605837/133e40b6-20b8-4a4c-a0f6-93c860c89ed7)

## CNAME vs R53 Alias

The problem if we would only use CNAME. In DNS an A record maps a name to an IP address, e.g. catagram.io -> 1.3.3.7. A CNAME maps a name to another name, e.g. www.catagram.io -> catagram.io. Its a way to create another alternative name for something within DNS.

The problem is that you cannot use CNAME for the apex of a domain, also known as the naked domain. You cannot have a CNAME record for catagram.io poiting to something else. And this is a problem, because many AWS services (e.g. Elastic Load Balancer) don't give you an IP address, but DNS name. With just CNAME pointing catagram.io to ELB would be invalid.

**A naked domain refers to a domain name without any subdomains. For example, "example.com" is a naked domain, while "www.example.com" is not**.

Alias (only in AWS) fixes the problem:
* Alias records map a NAME to an AWS resource.
* Can be used both for naked and normal records.
* For non naked functions like CNAME.
* There is no charge for ALIAS requests pointing at AWS resources.
* **For AWS Services - by default pick ALIAS**.
* You can have a `A` record alias and CNAME record alias. 
* Alias records are always of type A or AAAA.
* Cannot set TTL. Set automatically.

## Simple Routing

Default routing policy. **Only one not supporting health checks**.

![image](https://github.com/MrR0807/Notes/assets/24605837/a7901349-01d7-44e7-a06d-d32bbd2f02bf)

## R53 Health Checks

* Health checks are separate from records, but are used by records.
* Health checkers can check any internet IP, not only within AWS.
* Health checks every 30 sec (every 10 sec costs extra).
* You can have TCP, HTTP/HTTPS and HTTP/HTTPS with String Matching checks. The latter searches for a string in the response body within first 5120 bytes response body.
* There are three types of checks: Endpoint, CloudWatch Alarm (more in depth check using CloudWatch Agent), calculated checks/checks of checks checking multiple different granualar checks.
* Health checkers are distributed throughout the world.

![image](https://github.com/MrR0807/Notes/assets/24605837/fc076072-df67-440a-bda1-62e40be720a0)

## Failover Routing

Failover routing is a R5e routing policy. This routing policy plays upon primary and secondary routing strategy. Primary is being constantly health checked, and if there is failure, secondary target is used. For example primary being EC2 instance while secondary S3 bucket.

![image](https://github.com/MrR0807/Notes/assets/24605837/8792b8f3-7414-4287-905f-be8e8cd92cd9)

## Multi Value Routing

Its a mix of simple and failover policy. With Multi Value routing you can create multiple records with the same name. Each record can have a healthcheck associated.

![image](https://github.com/MrR0807/Notes/assets/24605837/0914a303-9b66-40fe-8f34-6059af08a7eb)

## Weighted Routing

Weighted routing can be used when you're looking for a simple form of load balancing, or when you want to test new versions of software.

![image](https://github.com/MrR0807/Notes/assets/24605837/08063c0b-9535-4f72-b19b-5e7ed6f0a8f9)

## Latency Routing

Use latency-based routing when optimising for performance and user experience.

![image](https://github.com/MrR0807/Notes/assets/24605837/4a7d8e41-218d-44ed-b5f4-7e952dc8129e)

## Geolocation Routing

Similar to latency rounting. Instead of latency, the location of resources and location of customer is used. **Does not return the closest**.

![image](https://github.com/MrR0807/Notes/assets/24605837/7cbca637-3572-4650-aa53-6c6249974f89)

## Geoproximity Routing

Tries to provide records that are as close to your customer as possible. Latency routing calculates the lowest latency. The geoproximity - calculates the distance and answers with a record with the lowest distance.

Geoproximity routing lets R53 route traffic to your resources based on the geographic location of your users and your resources, but you can optionally choose to route more traffic or less traffic to a given resource by specifying a value called bias.

![image](https://github.com/MrR0807/Notes/assets/24605837/850615b4-ac2a-479b-802f-4db4f06f2de6)

## R53 Interoperability

When you register a domain with R53 it actually does two things:
* Domain registrar.
* Domain hosting.

R53 can do either alone - be registrar or do only hosting.

Flow when you register a domain and host on AWS (steps are separated either by domain hosting or domain registrar):
* R53 accepts your money (domain registration fee).
* R53 allocates 4 Name Servers (NS) (domain hosting).
* R53 Creates a zone file (domain hosting) on the above NS.
* R53 communicates with the registry of the Top Level Domain (TLD) (Domain Registrar).

![image](https://github.com/MrR0807/Notes/assets/24605837/0e81a5c9-2047-4e31-9f05-a2a29348bc75)

You can register domain via R53 and host zone files in 3rd party. In that case, you would need to get the details of those servers and pass those details on to R53. R53 then would liase with TLD to set name server recrods within the domain to point the name servers managed by 3rd party.

![image](https://github.com/MrR0807/Notes/assets/24605837/9759ba60-63e7-4d7b-b03e-f57c12e9b0d1)

![image](https://github.com/MrR0807/Notes/assets/24605837/cfdbba59-85c8-4b8f-8ade-57a97d828536)

## Implementing DNSSEC using Route53

![image](https://github.com/MrR0807/Notes/assets/24605837/01974760-5253-41a8-993c-fffb11b888cc)

R53 without DNSSEC. DNSSEC is enabled, then the process starts with KMS:
* This part can be done separately or as part of enabling DNSSEC signing. Either way, asymmetric key is created in KMS. From this key, Key Signing Key is created. The created keys which R53 uses need to be in the us-east-1 region.
* Next R53 creates the Zone Signing Keys internally. They are managed by R53.
* Next R53 adds the Key Signing Key and the Zone Signing Key public parts into a DNSKEY record within the hosted zone. This tells any DNSSEC resolvers which public keys to use to verify the signatures on any other records in this zone.
* Next, the private Key Signing Key is used to sign those DNSKEY records and create the RRSIG DNSKEY record. These signatures mean that any DNSSEC resolver can verify that the DNSKEY records are valid and unchanged.
* R53 has to establish a chain of trust with the parent zone. The parent zone needs to add a DS (delegated signer) record, which is a hash of the public part of the Key Signing Key for this zone.

![image](https://github.com/MrR0807/Notes/assets/24605837/ff6742eb-dca2-4e71-ad89-430941cf97de)

# Relational Database Service (RDS)

## Database Refresher & MODELS

DynamoDB is a wide column store. The only rigid part of such databases are key - each entry has to have a key of the same structure. Data, on the other hand, can have all parameters (ala columns), or mixture, or none.

Document databases - similar to key/value database, but the content of documents is exposed to the database (ala ElasticSearch).

With a wide-column nosql db, every key maps to potentially many columns that can be selected. This can make reads more efficient, since we only need to read the columns that we are interested in. With the key-value nosql db, all the columns would be in the same value field, so everything would have to be read.

## ACID vs BASE

Both are DB transaction models. 
CAP Theorem - Consistency, Availability, Partition Tolerant.
ACID chooses consistency.
BASE chooses availability.

**EXAM NOTE!** if you see ACID mentioned, then it is refering to RDS and has **limitation on scale**. 

**EXAM NOTE!** if you see BASE it means NoSQL and scales very well and has higher performance.

**EXAM NOTE!** if you see NoSQL or ACID mentioned with DynamoDB, then you can assume they are reffering to DynamoDB transactions.

## Databases on EC2

There are two different setups to run database on EC2:
* Database, Application, Webserver are all running on the same EC2 instance.
* Database is running on a separate EC2 from Webserver and Application.

There is cost associated for data transitting between AZ in the same region.

Why you might deploy on EC2:
* Access to the DB Instance OS. Questionable requirement.
* Advance DB Option tuning (DBROOT access). A lot of AWS managed database now allow to tune those parameters. Questionable requirement.
* DB or DB version AWS don't provide.
* Architecture AWS don't provide (replication/resilience).

Negatives:
* Admin overhead - managing EC2 and DBHost.
* Backup / Disaster Management.
* **EC2 is single AZ**.
* Not easily scaling, no serverless.
* Replication.
* Performance - AWS invest time into optimisation and features.

## Relational Database Service (RDS) Architecture

* RDS is not Database as a Service (DBaaS). With RDS you pay for and receive a database server.
* You can have multiple databases on one DB Server (instance).
* Choice of DB Engines: MySQL, MariaDB, PostgreSQL, Oracle, SQL Server.
* Amazon Aurora is a different product.
* No access to OS or SSH access.
* Not a public service like S3 or DynamoDB.
* AWS will pick primary and standby instances in different AZs.
* If you deploy RDS in public subnet, you can configure with public addressing allowing access from the public internet. This is bad practice, but possible.
* Each RDS instance has its own dedicated EBS storage per instance.
* Backups and snapshots are placed in S3. You do not see them.

![image](https://github.com/MrR0807/Notes/assets/24605837/b4fe1dec-ded4-45ac-9d30-221051d3d752)

Costs:
* Instance size and type.
* Multi AZ or not.
* Storage type and amount (per GB).
* Data transferred (from and to the internet and other AWS regions).
* Backups and Snapshots.
* Licensing (if applicable).

## Relational Database Service (RDS) MultiAZ - Instance and Cluster

There are two modes of replications:
* MultiAZ Instance.
* MultiAZ Cluster.

In MultiAZ Instance mode there is a primary instance in AZ-a and replica in AZ-b. The replication is at storage level. This is less efficient than MultiAZ cluster. MultiAZ Instance mode is a synchronous replica. All reads and writes are happening only to primary instance. Backups on the other hand happen on Standby instance. **Because MultiAZ Instance uses DNS change in order to point to standby instance, there can be brief outages due to DNS caching (60sec or 120 sec).

Summary of MultiAZ Instance:
* Data goes to Primary and replicated to standby (synchronous).
* Extra cost for replica.
* **One standby replica only**.
* Replica can't be used for reads or writes. Failover can take from 60-120 seconds due to DNS caching.
* Same region only.
* Backups can be taken from standby replica to improve performance.
* Failover can happen due to different reasons - az outage, primary failure, manual failover, instance type change and software patching.

MultiAZ Cluster.

One writer can replicate to **two** reader instances (this is the main difference between Aurora and MultiAZ cluster). The difference between MultiAZ cluster and MutliAZ instance is that readers are usable. Application support is required in order to distinguish between readers and primary. The data is commited when 1+ reader finished writing. Different from Aurora, each instance has its own local storage. However, like Aurora, you access the cluster using a few endpoint types:
* cluster endpoint - CNAME which points to the writer.
* reader endpoint - directs to any reader (including the writer).
* instance endpoints - point to a specific instance.

![image](https://github.com/MrR0807/Notes/assets/24605837/8781dddb-077f-4fa7-bd55-633ba454848a)

Summary of MultiAZ Cluster:
* 1 writer and 2 reader db instances (different AZs).
* Runs on much faster hardware.
* Any writes are firstly written to local storaged and flushed to EBS.
* Readers can be used to scale for reads.
* Replication is done via transaction logs - more efficient.
* Failover is faster ~35 seconds + transaction log apply.
* Writes are commited only when at least 1 reader has confirmed.

## RDS Automatic Backup, RDS Snapshots and Restore

There are two types of backup like functionalities:
* Automated backups - occur once per day. Uses the same architecture (first one is full, follow up are incremental). Occur during a backup windown, which is defined on the instance. Addition to snapshot, every 5 minutes database transaction logs are written into S3. This creates a five minute Recovery Point Objective. Backups are cleared automatically. Retention period can be set up to 35 days.
* Snapshots - not automatic. You run them explicitly. They are stored in S3, which are managed by AWS. They function like EBS snapshots. First snapshot is the full copy of data stored within. Then onwards - incremental. Snapshots don't expire. **EXAM NOTE** You have to clear them youself. You can run them at any rate you like. More frequent snapshots will be done faster.

Both are stored in S3, but use AWS managed buckets. **EXAM NOTE** You cannot see these buckets via your own S3 console, but only via RDS console.

RDS can replicate backups to another region - both snapshots and transaction logs. Charges apply for the cross-region data copy. **Not default, this has to be configured with automated backups**.

RDS restores:
* During restore, **EXAM NOTE! It creates new RDS instances with new address**. The IP address will be different, just like the endpoint will be different. DNS CNAME will be different.
* Backups are restored from the closest snapshot and transaction logs are replayed to bring DB to desired point in time.
* **Restores aren't fast**.

## RDS Read-Replicas

**EXAM NOTE**. Synchronous means multi AZ and asynchronous means read replicas. RR can be created in the same region as primary or created in other AWS regions. RR matter in two main areas:
* Read performance and read improvements.
* You can have 5 direct read replicas of DB instance.
* Each providing an additional instance of read performance.
* Read Replicas can have their own Read Replicas - but lag becomes a problem.
* Read replicas offer near zero RPO. That is because the data is synced with primary, so you have to just switch.
* They offer near zero RTO.

## RDS Data Security

* SSL/TLS is available for RDS (can be mandatory on a per user basis).
* Encryption at rest is supported in few different ways depending on the database engine:
  * By default it is supported using KMS and EBS encryption. Handled by RDS host and EBS storage. As far as database engine knows its just writing unencrypted data to storage. Using this method all storage, logs, snapshots and replicas are encrypted using the same master key. **Encryption cannot be removed once it is added**.
  * MSSQL and Oracle support TDE (Transparent Data Encryption). Encryption which is supported and handled within database engine. Data is encrypted/decrypted by database engine itself.
  * RDS Oracle supports TDE using CloudHSM. With this process data is even more secure - because CloudHSM is managed by you with no exposure to AWS.

![image](https://github.com/MrR0807/Notes/assets/24605837/2f63dbfe-2428-426c-b167-e1242e732f30)

IAM Authentication for RDS:
* Normally logins to RDS are controlled using local database users. They are not IAM users.
* You can configure RDS to allow IAM user authentication against a database. 

![image](https://github.com/MrR0807/Notes/assets/24605837/ee0b0ab3-b87f-404e-8be6-cdf5402c162f)

**EXAM NOTE** This is only authentication. No authorization support. Authorization is controlled by database users.

## RDS Custom

No real world/niche use. For exam:
* RDS custom fills a gap between running RDS and EC2 running a DB engine.
* RDS is fully managed - OS/Engine access is limited.
* DB on EC2 is self managed - but has overhead.
* RDS custom works for MSSQL and Oracle.
* Can connect using SSH, RDP, Session Manager and actually get access to operating system or database engine.
* When you run AWS RDS all internal (S3, EC2 etc) are hidden. When running RDS custom, those internal parts are visible, because they are running in your account.
* When doing customisation to RDS custom, **you need to pause automation and once you're done - resume automation**.

## Aurora Architecture

* Aurora architecture is very different from RDS.
* Uses as a foundation something called cluster. Cluster made of:
  * Single primary instance.
  * 0 or more replicas.
 * Aurora does not use local storage for the compute instance. Instead Aurora has a shared cluster volume.
 * All storage is based on SSD - high IOPS, low latency.

When primary instance writes data into cluster volume, Aurora synchronously replicates that data across all of storage nodes. This replication happens at the storage level. By default only primary instance can write into storage. Aurora's storage subsystem is much more resilient than that which is used by the normal RDS database engine.

With Aurora you can have 15 replicas.

Billing for storage is very different than normal RDS engine. With Aurora you don't have to allocate the storage that cluster uses. Storage is simply based on what you consume. Upper limit is 128 TB. You are billed for high watermark, meaning if you use 50 GB, you are build for 50 GB. If you removed some data, you are still billed for 50GB or in other words - maximum storage you've consumed in the cluster. If you want to save on those 10GB you need to migrate to new cluster (this is being change and it will not behave like this in the future).

There are several endpoints exposed:
* Cluster Endpoint - always points to the primary instance.
* Reader Endpoint - any replica + primary.

Its easier to scale reads, because it can automatically load balance between read instances unlike RDS.

Aurora costs:
* No free tier option.
* Beyond RDS single AZ Aurora offers better value.
* Compute - hourly charge, per second, 10 minute minimum.
* Storage - GB-Month consuemd, IO cost per request.
* 100% DB Size in backups are included.

Aurora backups:
* Automatic backups and snapshots work the same way as in RDS.
* Restore will create a brand new cluster.
* Backtrack can be used which allow in-place rewinds to a previous point in time (needs to be enabled per cluster basis).
* Fast clones make a new database much faster than copying all the data - copy-on-write.
  * Rapid Provisioning: Fast Clone enables you to provision a new Aurora database by copying only the necessary data pages from the source database. This significantly reduces the time required to create a clone compared to traditional methods.
  * Space Efficiency: The clone shares data with the source database until changes are made to either the source or clone. This means that initially, the clone consumes very little additional storage space beyond what is already used by the source database.

## Aurora Serverless

* Uses concept of ACU - Aurora Capacity Units. Capacity units represent a certain amount of compute and a corresponding amount of memory.
* For a cluster you can set Min and Max ACU (elasticity).
* Cluster can go to 0 and be paused if no activity (only billed for storage).
* Consumption billing per second basis.
* Same resilience as Aurora provisioned (6 copies across AZs).

Aurora provisioned vs Aurora serverless:
* Instead of provisioned servers - serverless has ACUs.
* ACUs are allocated from a warm pool. They are stateless - shared between many AWS customers.
* In an Aurora serverless cluster we have a shared proxy fleet which is managed by AWS. You as a developer interact with this shared proxy fleet, which maps to ACUs.

![image](https://github.com/MrR0807/Notes/assets/24605837/19d56d5a-e7e3-4dbc-afdb-378239357437)

Use cases:
* Infrequently used applications.
* New applications where load is unknown.
* Variable workloads (clear peaks).
* Development and test databases. Because during inactive periods it just scales down.

## Aurora Global Database

Global databases allow you to create global level replication using Aurora from a master region to up ot five secondary AWS regions. Replication is ~1s or less between regions.

![image](https://github.com/MrR0807/Notes/assets/24605837/5a6f97a6-2b20-44a4-9529-9fbf9eabb9fa)

## Multi-master writes

In multi-master mode all instances are capable of both read and write.

The difference between single master is that there is no cluster endpoint to use. An application is responsible for connectin to instances within the cluster. No load balancing accross instances, the application connects to one or all of the instances in the cluster. When one of the read write nodes receive a write opperation - it immediately proposes that data be commited to all of the storage in that cluster. Each node can either accept or decline. It rejects if it conflicts with something that is in flight. The write instance is looking ofr a quorum for data to be inserted. If the change is commited, that change is replicated to other storage nodes in the cluster just like with single-master cluster. Also, that change is also replicated to other compute nodes. This means that other writers can add the updated data to their cache.

DOES NOT EXIST ANYMORE!!!!!!!!! Only was available in MySQL 5.6 which is now deprecated.

## Relational Database Service (RDS) - RDS Proxy

* Opening and Closing Connections consume resources. It takes time and add latency. Especially prevelent in lambda architecture.
* DB proxy - layer sitting between application and database. Instead of application connecting to database directly, it connects to a proxy and the proxy maintains a connection pool. It can multiplex - handle more request that come in than there are connections open.
* DB Proxy is a managed services which run inside a VPC.

![image](https://github.com/MrR0807/Notes/assets/24605837/55c7a3c4-031b-4551-abcd-ca1cdb961be9)

**EXAM NOTE** When to use DB Proxy:
* Too many connections error.
* DB instances using T2/T3 (smaller or burst) instances.
* When using AWS Lambda.
* Long running connections (SAAS apps).
* Resilience to database failure. RDS proxy can reduce the time for failover.

Key Facts:
* Fully managed for RDS/Aurora.
* Auto scalling, highly available.
* Provides connection pooling.
* only accessible from a VPC (not from public internet).
* Accessed via Proxy Endpoint.
* Can enforce SSL/TLS.
* Can reduce failover time by over 60%.
* Abstracts failure away from applications.

## Database Migration Service (DMS)

* A managed database migration service.
* It starts with a replication instance that runs on EC2, you have to define replication task, source and destination endpoints.
* **One endpoint must be on AWS**.
* Database Migration Service jobs can be one of three types:
  * Full load (one off migration of all data). 
  * Full load + CDC for ongoing replication which captures changes.
  * CDC only - transfer the bulk DB data. Sometimes it is more efficient to use native tools (e.g. Oracle has their own export/import tools) and then use DMS simply to replicate the changes.
* DMS does not support any schema conversion, but there is a dedicated AWS tool known as Schema Conversion Tool.

**EXAM NOTE** if you see a DB migration question and one of the endpoints is in AWS, then it is a safe option to choose DMS. If the question talks about no downtime migration - default to DMS.

Schema Conversion Tool:
* SCT is used when converting one database engine to another (when engines are not compatible).
* **SCT is not used between DBs of the same type** (e.g. On-premise MySQL -> RDS MySQL engines are the same).
* Works with OLTP and OLAP (e.g. Teradata, Vertica, Greenplum) databases.

Another example where DMS is used with SCT:
* Doing large migrations of multi TB in size. Moving data over the network takes time and consumes capacity. DMS can utilise snowball products.
* The flow of such migration:
  * Step 1. Use SCT to extract data locally and move to a snowball device.
  * Step 2. Ship the device back to AWS. They load onto an S3 bucket.
  * Step 3. DMS migrates from S3 into the target store.
  * Step 4. CDC can capture changes and via S3 intermediary they are also written to the target database.

# EFS

## EFS Architecture

* EFS is an implementation of NFSv4.
* EFS Filesystems can be mounted in Linux.
* Shared between many EC2 instances.
* Private service. By default it is isolated to the VPC that it is provisioned. Access to EFS file systems is via mount targets which are things inside a VPC.
* Can be accessed from on-premises via VPC peering, VPN connections or AWS Direct Connect (physical private networking connection between a VPC and existing on-premises network).
* The mount targets have IP addresses taken from the IP address range of the subnet.

![image](https://github.com/MrR0807/Notes/assets/24605837/79614670-fbe6-4c3b-9125-413c4fb97d58)

**EXAM NOTE**.
* Linux only.
* Offers two performance modes: General Purpose and Max IO.
* General Purpose is default for 99.9% of uses.
* Max IO scale to higher levels of aggregate throughput and operations per second, but it does have a trade-off of increased latencies. Suits applications that are highly parallel.
* There are two throughput modes: bursting and provisioned. Bursting works like GP2 EBS volumes. It has a burst pool, and throughput scales with the size of the file system. With provisioned you can specify throughput requirements.
* Two storage classes: Standard and Infrequent Access.
* Lifecycle Policies can be used with storage classes to move data.

## AWS Backup

* Fully managed data-protection (backup/restore) service.
* Consolidate management into one place - across accounts (utilises services like Control Tower and Organisations) and regions (able to copy data between regions).
* Supports a wide range of AWS products (EC2, EBS, EFS, RDS, S3).

Main components:
* Backup Plans - frequency, window, lifecycle, vault, region copy. If you enabled continues backups then you can restore a supported service to a particular point in time. You can define life cycles, which can transition backups to cold storage. Backup has to be stored there for a minimum of 90 days.
* Backup Resources - what resources are backed up. Whether you want to backup S3 or RDS.
* Vaults - think of vaults as backup destinations. It is here where all the backup data is stored. You need to configure at least one of these.
* Vault Lock - write-once, read-many, 72 hours cool off, then even AWS can't delete. Any data retention periods that you set still apply - backups can age out, but setting this means it is not possible to bypass or delete anything early.
* On-Demand - manual backups created as needed.
* Some services implement point in time recovery. For example S3 and RDS. This means that you can restore that specific resource to specific date and time within a retention window.

# HA & Scaling

## Regional and Global AWS Architecture

These are global architectural components:
* Global service location and discovery. How does your machine where to point at? E.g. enter netflix.com.
* Content Delivery (CDN) and optimisation.
* Global health checks and failover. Detecting if infrastructure in one location is healthy or not and moving customers to another country as required.

Regional components:
* Regional entry point.
* Regional scaling and regional resilience.
* Application services and components.

Continueing Netflix example, let's say that N.America is primary region and Australia is secondary. Route53 has healthchecks which point to N.America if they are healthy, if not - Australia. Despite that, CDN can be used which is deployed as close to end user as possible.

![image](https://github.com/MrR0807/Notes/assets/24605837/83e8c81d-fa93-4f1b-a44b-59257f8f80a4)

Initially communications from your customers will generally enter at the web tier. Generally this will be a regional based AWS servicel like application load balancer or API gateway. CloudFront can cache objects from S3.

## Evolution of the Elastic Load Balancer

There are three different types of elastic load balancers:
* Load balancers are split between v1 (avoid/migrate) and v2 (prefer).
* Started with Classic Load Balancer - v1 - introduced in 2009.
  * Not really layer 7 LB (can't make decisions based on HTTP protocol features), lacking features, 1 SSL certificate per CLB, more expensive to use.
* Application Load Balancer (ALB) - v2 - HTTP/HTTPS/WebSocket. Truely Layer 7 devices.
* Network Load Balancer (NLB) - v2 - TCP/TLS/UDP. 

## Elastic Load Balancer Architecture - PART1

When you deploy ELB you have to pick AZ, which the load balancer will use. Specifically, you are picking one subnet in two or more AZs. Based on the subnets that you pick inside AZ, when you provision a load balancer, the product places into these subnets, one or more load balancer nodes. What you see as a single load balancer object, is actually made up of multiple nodes and these nodes live within the subnets that you pick.

Another important thing you have to choose **EXAM NOTE** is whether it is internet facing or internal. This choise controls the IP addressing for the load balancer nodes. If you pick internet-facing, then the nodes of that load balancer are given public addresses and private addresses. If you pick internal, then the nodes only have private IP addresses. Otherwise they are the same architecturally.

Public ELB can connect to both private and public EC2 instances. The important part is that if you want LB to be reachable from the public internet, it needs to have public IP/placed inside public subnet. Minimum subnet size is /27 or /28 in order for load balancers to scale. AWS suggests to use /27. **ELB require 8+ free IPs per subnet.**

**ELB is a DNS A Record pointing at 1+ nodes per AZ**.

![image](https://github.com/MrR0807/Notes/assets/24605837/745d522a-b160-4e54-bf74-48e221715fd3)

## Elastic Load Balancer Architecture - PART2

If load balancer would not exist, your user would communicate with specific instance. When that fails, the flow would be disrupted. Same thing with web tier applications communicating on behalf of customer to internal applications.

![image](https://github.com/MrR0807/Notes/assets/24605837/f2f170cb-8e17-42fc-8900-0235adb17c51)

### Cross Zone LB

Initially, LB node could only distribute connections to instances within the same AZ. Consider this architecture where LB is spread between two AZs (a and b). AZ-a contains 4 EC2 instances, while AZ-b only one. In this scenario LB node in AZ-a would distribute load between 4 instances, while AZ-b would distribute only to one. A fix for this was cross-zone load balacing. It simply allows every load balancer node to distribute any connections that it receives equally accross all registered instances in all AZs. Now enabled by default. **EXAM NOTE** often question arises how to distribute load evenly even if instances are deployed unevenly.

![image](https://github.com/MrR0807/Notes/assets/24605837/1deb107b-623b-4d27-aeb7-139de91f6728)

## Application Load balancing (ALB) vs Network Load Balancing (NLB)

Application Load Balancer:
* Layer 7 load balancer. Listens on HTTP and/or HTTPS.
* No other Layer 7 protocols (SMTP, SSH, Custom Gaming protocols). No TCP/UDP/TLS Listeners. Only HTTP or HTTPS.
* It understands layer 7 things like content type, cookies, custom headers, user location and app behaviour.
* HTTP/HTTPS always terminates connection on ALB and a new connection is made from ALB to the application.
* ALB must have SSL certs if HTTPS is used, because they have to initiate SSL connections on behalf of the client to the application.
* ALB are slower than NLB. **EXAM NOTE** If performance is key, then NLB is way to go.
* They can evaluate application health. 

ALB rules:
* Rules direct connections which arrive at a listener.
* Processed in priority order.
* Default rule = catchall is processed last.
* Rule Conditions: host-header, http-header, http-request-method, path-pattern, query-string, source-ip.
* Rules have actions: forward, redirect, fixed-response (e.g. provide same error code), perform certain types of authentications (authenticate-oidc, authenticate-cognito).
* **EXAM NOTE** If you need unbroken connection from client to application you have to use NLB. ALB brakes them.

NLB:
* Layer 4 devices - TCP, TLS, UDP, TCP_UDP.
* No visiblity or understanding of HTTP or HTTPS.
* Cannot interpret headers, cookies, sessions etc.
* They are very, very fast (millions of rps, 25% of ALB latency).
* **EXAM NOTE** Are not web or secure web, don't use HTTP or HTTPS, then you should default to NLB.
* Health checks just check TCP handshake. Not app aware.
* NLBs can have static IPs - useful for whitelisting.
* Forward TCP to instances - unbroken encryption.
* **EXAM NOTE** Used with private link to provide services to other VPCs.

ALB vs NLB:
* Choose NLB if you need unbroken encryption.
* Choose NLB static IP for whitelisting.
* Choose NLB for fastest performance.
* Choose NLB for protocols not requiring HTTP or HTTPS.
* Choose NLB for privatelink.
* Otherwise choose ALB.

## Launch Configuration and Templates

Launch Configurations and Launch Templates at the high level perform the same task. They allow the configuration of EC2 instances to be defined in advance. They let you define:
* AMI, Instance Type, Storage, Key pairs.
* Networking configuration and security groups.
* Userdata and IAM role.
* Both of them are not mutable. Launch Templates have versions.
* **AWS recommends using Launch Templates**.
* Launch Configurations have one use - they are used as part of Auto Scaling Groups.
* Launch Templates can be used for Auto Scaling Groups as well, furthermore they can launch EC2 instances directly from Console/CLI.

## Auto-Scaling Groups

* Auto Scaling Groups do one thing - they provide Automatic Scaling and Self-Healing for EC2.
* Uses Launch Templates or Launch Configurations.
* Has three important values associated with it - minimum, desired and maximum (e.g. `1:2:4`).
* Keep running instances at the Desired capacity.
* Normally Scaling Policies are used together with ASG. Scaling policies can update the desired capacity based on certain criteria for example CPU load.
* ASG runs across one or more subnets in VPC.
* ASG tries to maintain an even number of instances in each subnet.
* There are ways that you can scale Auto Scaling Groups:
  * Manual Scaling - Manually adjust the desired capacity.
  * Scheduled Scaling - Time based adjustments.
  * Dynamic Scaling has three subgroups:
    * Simple - "CPU above 50% +1", "CPU below 50 -1". Not only CPU, but memory, IO etc, lenght of SQS queue.
    * Stepped Scaling - Bigger +/- based on difference. It allows you act depending on how out of normal the metric value is. Maybe at one instance if the CPU usage is above 50%, but if there is a sudden spike of load, say above 80% then add three. Stepped Scaling allows to react quickly the more extreme the changing conditions.
    * Target Tracking - Desired Aggregate CPU = 40%. It allows you to define an ideal percentage of something. Has something like request count per target.
    * Scaling based on SQS - ApproximateNumberOfMessagesVisisble.
* Cooldown periods. Value in seconds and controls how long to wait at the end of a scaling action before doing another.

**ASG + Load Balancers**

Application Load Balancer checks can be much richer. They can monitor the state of HTTP or HTTPS requests. 

ASG Scaling processes:
* Launch and Terminate - Suspend and resume (??).
* AddToLoadBalancer - add to LB on launch.
* AlarmNotification - accept notification from CloudWatch.
* AZRebalance - Balances instances evenly across all of the AZs.
* HealthCheck - controls whether instance health checks are on or off.
* ReplaceUnhealthy - controls whether the ASG terminates unhealthy instances and replaces.
* ScheduledActions - whether the ASG will perform any scheduled actions or not.
* Standby - this allows you to suspend any activities of the ASG on a specific instance. This is really useful if you need to perform maintenance on one or more EC2 instances.

**EXAM NOTES**
* ASG are free.
* Use cooldowns to avoid rapid scaling.
* Think about more, smaller instances - granularity.
* Use with ALBs for elasticity.
* ASG defines when and where, launch templates defines what.

## ASG Scaling Policies

ASG don't need scaling policies - they can have none. When created without any scaling policies, it means that an ASG has static values for min, max and desired capacity.

## ASG Lifecycle Hooks

Lifecycle hooks allow you to configure custom actions on instances during ASG actions. You can define actions during instance launch or Instance termination transitions.

With ASG instances are paused within the flow and they wait:
* Until a timeout (then either Continue or Abandon). By default 3600 seconds.
* After completing a custom process you can resume the ASG scale-in/scale-out process using CompleteLifecycleAction.
* Lifecycle hooks can be integrated with EventBridge or SNS Notifications, which allow your systems to perform event driven processing based on a launch or termination of EC2 instance within an ASG.

![image](https://github.com/MrR0807/Notes/assets/24605837/9ef53554-374b-4dab-a2a6-b6f40c10362d)

## ASG HealthCheck Comparison - EC2 vs ELB

ASG assess the health of instances within that group using health checks. And if an instance fails a health check, then it is replaced within the Auto Scaling group. There are three types of health checks:
* EC2 (Default). Any statys (Stopping, Stopped, Terminated, Shutting Down or Impaired (not 2/2 status)) is viewed as unhealthy. Basically anything that is not in Running state.
* ELB (can be enabled). The instance which is healthy has to be running and passing ELB health checks.
* Custom - instances marked healthy or unhealthy by an external system.
* Health check grace period (default 300s) - delay before starting checks.

## SSL Offload & Session Stickiness

There are three ways how load balancer can handle secure connections:
* bridging (default for Elastic load balancer) - one or more clients makes one or more connections to a load balancer. Load Balancer is configured for HTTPS. Connections are terminated on the ELB and started anew (man in the middle attack). Needs a certificate. If you're in a situation where you have to be very careful where the certificates are stored, then you might have a problem with bridge mode. Compute might be an overhead with high volume applications.
* pass through (Network Load Balancer) - the load balancer just passes that connection along to one of the back end instances. This is only available in network load balancer. Listener is configured for TCP. No encryption or decryption happens on the NLB. The negative is that you cannot perform any load balacing based on HTTP. 
* offload (Elastic Load Balancer) - clients use https, connections are terminated on load balancer, but connects to instances using HTTP. ALB still requires certificate.

**Connection Stickiness** if applications are not using external services like DynamoDB, to handle stickiness, but instead rely on instances, then Elastic Load Balancer's session stickiness feature is required to use. Within an application load balancer, this is enabled on a target group. If enabled, the first time that a user makes request, the load balancer generates a cookie called AWSALB. Valid duration of a cookie is between 1 second and 7 days. With this cookie, LB routes requests to the same instance. This will happen until one of two things happen: instance fails, then user moved to a different instance or session expires then new cookie is provided and new instance is tied to said cookie. 

## Gateway Load Balancer

* Helps you run and scale 3rd party appliances (things like firewalls, intrustion detection and prevention systems).
* Inbound and Outbound traffic (transparent inspection and protection).
* At a high level Gateway Load Balancer has two major components:
  * GWLB endpoints, which run from a VPC and traffic enters/leaves via these endpoints.
  * GWLB balances across multiple backend appliances. These are just normal EC2 instances running security software.
* GWLB has to forward packets in their full format without altering them. These packets have source and destination IP addresses, which might be okay on the original network, but which might not work on the network where the security appliances are hosted from. That is why GWLB uses a protocal called GENEVE - tunneling protocal. A tunnel is created between the gateway load balancer and the backend instance.

Gateway Load Balancer actually powers AWS Network Firewall. You can decide to either bring in your own 3rd party security appliance to AWS and put it behind Gateway Load Balancer or use a managed service (AWS Network Firewall) without having to manage GWLB or firewall appliances yourself.

![image](https://github.com/MrR0807/Notes/assets/24605837/d00f969d-ef6c-4e60-bce8-04746ccecf48)

![image](https://github.com/MrR0807/Notes/assets/24605837/c5e7d38b-f9cb-43cb-98da-b8df88ad9267)

# SERVERLESS AND APPLICATION SERVICES

## Architecture Deep Dive

Nothing new.

### Event-Driven Architecture

Nothing new.

## AWS Lambda

* FaaS - function as a service.
* Functions are loaded and run in a runtime environment (e.g. Python 3.8).
* The environment has a direct memory (indirect CPU) allocation. Memory from 128MB to 10240MB in 1MB step. 1769 MB gives 1 CPU.
* By default Lambda is stateless (there are workarounds).
* Mounts 512MB storage as /tmp. Up to 10240MB.
* **EXAM NOTE**. Can run up to 15 minutes.

Lambda function at its most basic is a deployment package which Lambda executes - you define the language/runtime, provide a deployment package (50MB zipped/250MB unzipped), and you set resources. Whenever Lambda is invoked, what actually happens is the deployment package is downloaded and executed within this runtime environment. 

**EXAM NOTE** if you see Docker mentioned, consider this to mean not Lambda. Docker is anti-pattern for Lambda. However, you can use similar/existing build processes to build Lambda images.

Lambda use cases:
* File processing (S3, S3 events).
* Database Triggers (DynamoDB, Streams).
* Serverless Cron (EventBridge/CloudWatch Events).
* Realtime Stream Data Processing (Kinesis).

**Network**

Lambda has two networking modes:
* Public (default) - they are given public networking which means they can access public AWS services (e.g. SQS, DynamoDB) and public Internet. Offers the best performance, because no customer specific VPC networking is required. Lambda functions have no access to VPC based services unless public IPs are provided and security controls allow external access.
* VPC - Lambda functions running in a VPC obey all VPC networking rules. For example, can freely access other VPC based resources (assuming any network ACLs and security groups allow that access). On the flip side they cannot access anything outside VPC unless networking configuration permits.

![image](https://github.com/MrR0807/Notes/assets/24605837/2ee92376-0188-4eb7-ba0a-5b73e3f8d63f)

Old way how AWS Lambdas were running in VPC mode that they not actually ran in your VPC, but they mount network interfaces within your VPC. This created scalability problems because more and more network interfaces were required and it was slow process. Now AWS analyze all of the functions running in a region in an account and build up a set of unique combinations of security groups and subnets. For every unique one of those, one ENI is required in the VPC. If all your functions used a collection of subnets, but the same security groups, then one network interface would be required per subnet.

![image](https://github.com/MrR0807/Notes/assets/24605837/cfbea771-387e-448c-a991-f8a5fc2b3e03)

![image](https://github.com/MrR0807/Notes/assets/24605837/be1a8663-257d-47a2-a579-f59fb84adeb0)

**Permissions**

In order for Lambda to access any AWS products and services it nees to be provided with an execution role. A role is created which has a trust policy which trusts Lambda and the permissions policy that role has is used to generate temporary credentials that the Lambda function uses to interact with other resources.

Lambda also has resources policies. In many ways is like a bucket policy in S3. It controls who can interact with a specific Lambda function. Its this resource policy which can be used to allow external accounts to invoke a Lambda function or certain services to use a Lambda function such as SNS or S3.

**Logging**

* Lambda uses CloudWatch, CloudWatch Logs and X-Ray.
  * Logs go to CloudWatchLogs.
  * Metrics go to CloudWatch.
  * X-Ray distributed tracing.
* CloudWatch Logs requires permissions via Execution Role.

**Invocation**

Three ways how to invoke Lambda:
* Synchronous - CLI/API invokes a lambda function, passing in data and wait for a resposne. Waits for response. Blocking. Lambda function returns respond with data or fails. A common architecture of synchronous Lambda is via API Gateway, which invokes underneath a lambda function. 
* Asynchronous - typically used when AWS services invoke lambda functions on your behalf. For example we have an S3 bucket with S3 events enabled. For example we upload an image to S3 bucket which creates an event, which creates lambda function. In this instance, S3 is not waiting for application response. Fire and forget. If processing of the event fails, lambda will retry between 0 and 2 times (configurable). Lambda handles the retry logic. Events that failed can be sent to dead letter queue. Furthermore, lambda supports destinations and events can be forwarded to other services (e.g. another lambda, SQS, SNS, EventBridge).
* Event Source mappings - this is typically used on streams or queues which don't support event generation to invoke lambda (Kinesis, DynamoDB streams, SQS). In their inception, these services do not generate events when they themselve receive events, hence there is a hidden component, which sources events from streams/queues. **EXAM NOTE** Event Source Mapping is reading from the source, hence it needs permissions to access it. It uses lambda Execution Role. Lambda can receive more than one event from event source mapping, hence it is important to configure event batch size.

![image](https://github.com/MrR0807/Notes/assets/24605837/9c3a25f3-6c68-47ce-a66c-435b2fcc0f1f)

![image](https://github.com/MrR0807/Notes/assets/24605837/9c89edca-8024-4a6c-976f-ce48bd28ec98)

**Versions**

* Lambda functions have versions - v1, v2, v3.
* A version is the code + the configuration of the lambda function.
* A version is immutable. Has its own ARN.
* You can create aliases.

**Lambda startup times**

An execution context is the environment a lambda function runs in. A cold start is a full creation and configuration including function code download. If there is a small gap between two lambda invocations there is a chance that same execution context will be used without cold start. A Provisioned concurrency can be used to speed up cold start. AWS will create and keep X contexts warm and ready to use in advance.

## CloudWatchEvents and EventBridge

* AWS starting to encourage a migration from CloudWatch Events to EventBridge.
* EventBridge is basically CloudWatch Events v2.
* A single default Event bus for AWS account in CloudWatch Events. You cannot interact with it.
* In EventBridge you can create additional events.
* Rules match incoming events. When a rule is matched - the event is delivered to a target. Alternatively you have scheduled rules which is kinda of pattern matching for certain date and time or ranges of dates and times.
* Routes the events to 1+ targets (could be lambda).

![image](https://github.com/MrR0807/Notes/assets/24605837/3e72bb32-feaf-4a5c-9028-5e6af554119c)

## Serverless Architecture

* Stateless and ephemeral environments.
* Generally everything is event-driven. Nothing is running until it's required.
* FaaS is used where possible for compute functionality.

![image](https://github.com/MrR0807/Notes/assets/24605837/905942c2-d9e5-4dfb-9381-a756e31b1292)

## Simple Notification Service

* Public AWS Service - network connectivity with Public Endpoint. To access it, you need network connectivity with the public AWS endpoints.
* Cordinates the sending and deliver of messages.
* Messages are <= 256KB payloads.
* SNS topics are the base entity - permissions and configurations are on them.
* SNS used across AWS for notifications, e.g. CloudWatch uses it.
* Supports Server Side Encryption.
* SNS topic'as are capable of being used cross accounts. 

## Step Functions

Step functions address some of the limitations of Lambda. Lambda is a function as a service and the best practice is to create functions, which are small, focused and do one thing very well. You should never put a full application inside a Lambda function. Lambda function cannot pass 15 minute max execution time. 

Step functions lets you create what are known as State Machines. Think of a State Machine as a workflow. It has a Start point and an end point. Between those points are states. States are the things which occur inside the state machine. They take in data, modify it and output data. Conceptually, the State Machine is designed to perform an activity or perform a flow, which consists of lots of individual components, and maintain the idea of data between those states. **The maximum duration for State Machine executions within Step Functions is 1 year**.

There are two types of workflows available in step functions:
* Standard - standard is the default and has **one year execution limit**. 
* Express - designed for high volume, event processing workloads such as IOT, streaming data processing and transformation. These can run up to **5 minutes**.

State machines have templates, which you can export and it uses Amazon State Language (ASL) - Json Template. IAM role is used for permissions.

Types of states:
* Succeed and Fail.
* Wait - wait a certain period of time or until a specific date and time.
* Choice - take a different path depending on an input.
* Parallel - allows you to create parallel branches within a state machine.
* Map - map state accepts a list of things, e.g. list of orders. For each iteam in that list perform an action.
* Task - represents a single unit of work performed by a state machine. Task state can be integrated with lots of different services, e.g. Lambda, AWS Batch, Dynamo DB etc.

## API Gateway 101

* API Gateway acts as an endpoint or an entry point for applications looking to talk to your services and architecturally it sits between applications and integrations.
* Highly available and scalable, handles authorisation, throttling, caching, CORS, transformations, OpenAPI spec, direct integration and much more.
* Can connect to services/endpoints in AWS or on-premises.
* HTTP APIs, REST APIs and WebSocket APIs.

![image](https://github.com/MrR0807/Notes/assets/24605837/8738b959-3d8c-4bdf-bac2-b54997fe12f1)

API Gateway supports a range of Authentication methods:
* Integrates with Cognito.
* Lambda based authorization - custom authorization.

API Gateway endpoint types:
* Edge-Optimized - any incoming requests are routed to the nearest CloudFront point of presence.
* Regional - when you have clients in the same region. This is suitable when you have users in the same AWS region as your applications.
* Private - endpoint accessible only within a VPC via interface endpoint. This is how you can deploy completely private APIs.

API Gateway Stages:
* APIs are deployed to stages. Each stage has one deployment, e.g. api.catagram.io/prod, api.catagram.io/dev.
* Stages can enable you for canary deployments. Can be configured so a certain percentage of traffic is sent to the canary. This can be adjusted over time - or the canary can be promoted to make it the new base stage.

**EXAM NOTE**

API Gateway Errors:
* Error codes that are generated by API Gateway are of two categories: 4xx, 5xx.
* 400 - bad request.
* 403 - access denied.
* 404 - not found.
* 429 - api gateway throttle.
* 502 - bad output is returned whatever is backing the backend service.
* 503 - service unavailable.
* 504 - timeout - API has to respond within 29 second limit.

**EXAM NOTE** Caching is configured per stage. TTL default is 300 seconds. Configurable between 0 and 3600 seconds. Can be encrypted. Cache size 500mb - 237GB.

## Simple Queue Service

* Public, Fully Managed, High Available Queues - Standard (order is not guarantee) or FIFO (guarantee an order).
* Messages up to 256KB in size or link to large data.
* **EXAM NOTE** Received messages are **hidden** (VisibilityTimeout). After VisibilityTimeout messages can reappear or explicitly deleted.
* Dead-Letter queues can be used for problem messages.
* Standard = at-least-once, FIFO = exactly-once.
* FIFO performance - 3000 messages per second with batching or up to 300 messages per second without.
* Standard queues are unlimited.
* Billed based on requests - a single request that you make to SQS. With 1 request you can receive from 0 to 10 messages or up to 64KB total.
* Encryption at rest (KMS) and in-transit.
* Queue policy can provide access from external accounts. A queue policy is just a resource policy.
* **EXAM NOTE** FIFO queues to be valid need to have a `.fifo` suffix.

**EXAM NOTE** Fanout architecture is really important for exam.

![image](https://github.com/MrR0807/Notes/assets/24605837/283f5a97-c459-45f0-800a-36251da89a82)

## SQS Delay Queues

Delay queues at high level allow you to postpone the delivery of messages to consumers. Messages added to the queue will be invisible for `DelaySeconds`. DelaySeconds min value is 0 and max is 15 minutes.

## SQS Dead-Letter Queues

`maxReceiveCount` defines how many times message can be received until placed into dead letter queue.

Enqueue timestamp of message is unchanged (it is orginal when message was added to main queue). Retention period of a dead-letter queue is generally longer then the source queues.

Single dead letter queue can be used for multiple sources.

## Kinesis Data Streams

* Kinesis is a scalable streaming service. It is designed to ingest data from lots of devices and applications.
* Producers send data into a kinesis stream.
* It can scale from low levels of data throughput to near infinite data rates.
* Public services and highly available by design.
* Streams store a 24-hour moving window of data by default.
* Storage is included for however much data you ingest during those 24 hours.
* The window can be increased to 365 days.
* Multiple consumers can access data from anywhere in that moving window.
* Shard architecture is used to scale Kinesis. One shard supports 1 MB ingestion and 2 MB Consumption. Shards are automatically scalled depending on the load.

### SQS vs Kinesis

* SQS 1 production group, 1 consumption group. Generally you won't have hundreds or thousands of sensors sending to an SQS queue.
* Generally you'll have one consumer or consumer group - worker tier.
* SQS does not provide persistence and no time window.
* Kinesis designed for huge scale of data.
* Kinesis designed for multiple consumers, each of which might be consuming data at different rates.

## Kinesis Data Firehose

* Fully managed service to load data for data lakes, data stores and analytics services.
* Automatic scaling, fully serverless, resilient.
* Near Real Time delivery (~60 seconds).
* **EXAM NOTE** Kinesis provides real time data, Firehose supports **near real time**.
* Supports transformation of data on the fly (lambda).
* Billing - volume through firehose.
* **EXAM NOTE** Need to know valid destinations for firehose:
  * HTTP endpoints.
  * Splunk.
  * Redshift.
  * ElasticSearch.
  * S3.
* Source - kinesis data streams, or **producers can send data directly into Firehose**.
* Firehose waits for 1MB of data or 60 seconds. These can be adjusted.
* Firehose transforms data via lambda functions.
* All of the above destinations are direct except Redshift. Underneath, it firstly places data into S3 and then uses Redshift copy command.

## Kinesis Data Analytics

* Provides real time processing of data using SQL. Data inputs at one side, queries run against that data in real time and then data is output to destination at the other.
* Ingest data from either Kinesis Data Streams or Kinesis Firehose. Also from S3 as reference data.
* Destinations:
  * Firehose (near real time), hence indirectly also S3, Redshift, Splunk, ElasticSearch.
  * AWS Lambda (real time).
  * Kinesis Data Streams (real time).
* Processing data with Kinesis Analytics application is not cheap.

![image](https://github.com/MrR0807/Notes/assets/24605837/ff346381-f2ee-46dc-9188-e91853c228f3)

Kinesis Analytics Application actually tackles the same old stream enriching problem. Reference table in S3 is just slow moving data which can enrich fast moving data. For example, some kind of game where change events are coming constantly and being enriched by slow moving data like player name etc.

**When to use Kinesis Data Analytics**

* Streaming data needing real-time SQL processing.
* Time-series analytics, e.g. elections/e-sports.
* Real-time dashboards - leaderboards for games.
* Real-time metrics - security and response teams.

## Kinesis Video Streams

* Ingest live video data from producers.
* Producers can be security cameras, smartphones, cars, drones, time-serialised audio, thermal, depth and radar data.
* Consumers can access data frame-by-frame or as needed.
* Can persist and encrypt data (in-transit and at rest).
* **EXAM NOTE** Cannot access data directly via storage. Only via API.
* Integrates with other AWS services e.g. Rekognition and Connect.

## Amazon Cognito - User and Identity Pools

* Cognito has terrible naming.
* Cognito provides two main pieces of functionality:
  * Authentication, Authorization and user management for web/mobile apps. There are two parts of Cognito:
    * User pools - sign in and get a JSON Web Token. JWT can be used for authentication with applications, certain AWS products e.g. API gateway. However, **most AWS services cannot use JWT**. User pool job is to control sign in and deliver a JWT. When thinking about user pools think about database of users which can include external identities. **When you sign in, you get JWT**.
    * Identity pools - allow you to offer access to temporary AWS credentials, which can then be used to access AWS resources. One option is Unauthenticated Identities/Guest Users. Also provides **Federated Identities**. You can swap an external identity (e.g. Google, Facebook, Twitter, User pool) for temporary AWS credetianls. Identity pools work by assuming IAM roles.

### User Pools Architecture

![image](https://github.com/MrR0807/Notes/assets/24605837/42569fc9-f024-47b1-86ab-39c82c3013f6)

### Identity pools

If we want to support five different token providers (e.g. Google, Facebook, Twitter), we need to have five different configurations within Identity pool. In identity pool, there are at least two roles: Authenticated Role and Unauthenticated role.

![image](https://github.com/MrR0807/Notes/assets/24605837/f4c1dc0d-177a-48cd-9ce9-1d00351a2fce)

### Combine both

The benefit is that identity pool can be configured with only a single external identity provider, which is user pool.

![image](https://github.com/MrR0807/Notes/assets/24605837/d635f19f-3c77-4c08-8558-271e14feb702)

### Summary

User pools are for sign in and sign up while identity pools are for swapping identity tokens for from external ID provider for temporary AWS credentials.

## AWS Glue 101

* Glue is serverless ETL.
* There is another ETL called data pipeline and users servers (EMR) to perform the tasks.
* Moves and transforms data between source and destination.
* Crawls data sources and generates the AWS Glue Data catalog.
* Data Source: S3, RDS, DynamoDB, JDBC Compatible stores, Kinesis Data Stream, Apache Kafka.
* Data Targets: S3, RDS, JDBC Databases.

* AWS Glue provides a data catalog. Data catalog is a collection of metadata combined with data management and search tools. In other words - persistent metadata about data sources in region.
* One catalog per region per account.
* Athena, Redshift, EMR and AWS Lake Formation can use data catalog.
* Data is discovered by crawlers. Giving them credentials to access data stores.

![image](https://github.com/user-attachments/assets/640ed7e7-23b8-4645-8acb-df5711050303)

## Amazon MQ 101

Amazon MQ is a managed message broker service for Apache ActiveMQ Classic and RabbitMQ.

* SNS provides (topics) one to many communication channels.
* SQS provides (queues) one to one communication channels.
* Both services are highly scalable and highly available.
* This service is for migrating existing queue systems from on-premise, to AWS.
* It supports JMS API and protocols such as AMQP, MQTT, OpenWire and STOMP.
* Provides queues (one to one) and topics (one to many).
* **EXAM NOTE** Not a public service. Private networking required (VPC based).
* Cannot use with other AWS products.
* **EXAM NOTE** Default position is to use SNS or SQS.
* **EXAM NOTE** Use SNS or SQS if AWS integration is required (logging, service integration).

## Amazon AppFlow

* Fully Managed integration service (middleware).
* Exchange data between applications (connectors) using flows.
* Example - sync data across applications, sync contact records from salesforce to redshift, sync suport tickets from zendesk to s3.
* Uses public endpoints, but works with PrivateLink.

![image](https://github.com/user-attachments/assets/d972908a-4262-4084-ad72-4f75e75fc47a)

# GLOBAL CONTENT DELIVERY AND OPTIMIZATION

## Cloudfront Architecture

CloudFront is content delivery network. It improves content delivery by caching and by using an efficient global network.

CloudFront Terms:
* Origin - the source location of your content.
  * S3 Origin or Custom Origin (anything else which runs a web server and has a publicly routeable IP version 4 address).
  * You have one or more origins.
* Distribution - the "configuration" unit of CloudFront.
* Behavior - a sub "configuration" which contains most of the configuration. It works on a principle of a pattern match. Distribution always has at least one behavior, but in can have many more. **EXAM NOTE** Restrict viewer (viewers must use CloudFront signed URLs or signed cookies to access content) and caching are set in behavior.
* Edge Location - local cache of your data.
* Regional Edge Cache - larger version of an edge location. Provides another layer of caching.
* **EXAM NOTE**. It does not provide write cache. Only read cache.

![image](https://github.com/user-attachments/assets/ef694231-9cad-4083-b1c4-29ee8f15b2a2)

![image](https://github.com/user-attachments/assets/07c201a9-17e9-43f2-862a-949847cbec6f)

## CloudFront (CF) - Behaviours

Nothing interesting.

## CloudFront - TTL and Invalidations

* More frequent cache HITS = lower origin load.
* Default TTL - 24 hours.
* You can set Min TTL and Max TTL.
* **EXAM NOTE** There are several headers you need to remember:
  * Origin Header: Cache-Control max age (seconds) - does the same thing as below. After defined seconds are passed, the object is viewed as expired.
  * Origin Header: Cache-Control s-maxage (seconds) - does the same thing as above. After defined seconds are passed, the object is viewed as expired.
  * Origin Header: Expires (Date and Time) - specific date and time when object will be viewed as expired.
* Custom Origin or S3 (via object metadata).

Cache invlidation performed on a distribution. Applies to all edge locations and takes time.

**EXAM NOTE** Versioned file names are better than trying to invalidate files anytime you need a change. For example, instead of invalidating `test.jpg`, you could create a new file `test_v2.jpg`. It is more cost efficient, it also works in client's browser, and lastly, in logging you can identify which file is actually returned from cache - old or new. With same name, you wouldn't know.

## ACM - AWS Certificate Manager

* ACM can function both as a public Certificate Authority - generating certificates which are trusted or as a private certificate authority.
* With Private Certificate Authority you need to trust your private Certificiate Authority.
* You can generate or import certificates. It can also automatically renew them.
* **EXAM NOTE** If you imported certificates, then you are responsible for renewal.
* Certificates can be deployed out to supported services - e.g. CloudFront and ALBs, but not EC2. **EXAM NOTE** Not all AWS services are supported.
* ACM is a regional service.
* Certs cannot leave the region they are generated or imported in.
* **EXAM NOTE** To use a cert with ALB in ap-southeast-2 you need a cert in ACM in ap-southeast-2. This is critical to understand.
* **EXAM NOTE** Global Services such as CloudFront operate as though within us-east-1. In other words you need to configure certs in us-east-1 for CloudFront.

![image](https://github.com/user-attachments/assets/040b20e1-bfae-4f39-86d4-2f03f9e477cf)

## Cloudfront and SSL/TLS

* Each CloudFront receives a default domain name (CNAME). It looks something like this - `https://d111111111abassf.cloudfront.net`, starts with a random pattern and ends with `cloudfornt.net`.
* SSL Supported by default as long as you use `*.cloudfront.net`.
* You can use alternative domain names (CNAMES) e.g. `cdn.catagram...`.
* In order to provide SSL/TLS for alternative domain name you need a cert which contains that information as well.
* There are two SSL connections happening: Client -> CloudFront and CloudFront -> Origin. **EXAM NOTE** Both of them need a valid, public certificates (and intermediate certs).
* Self-signed certs will not work with CloudFront. They need to be publicly trusted certificates.

CloudFront and SNI
* SNI - server Name Indication. This adds the ability for a client to tell a server which domain name its attempting to access. It occurs within the TLS handshake. This extension to SSL was required in order to host multiple different servers on the same IP. For example, if you'd have two apps running on the same instance, before this extension, only one certificate was possible per IP.
* If you need dedicated IP in CloudFront, because older browsers don't support SNI, then you have to pay extra to AWS. Otherwise, SNI is enabled by default and it is free.

![image](https://github.com/user-attachments/assets/c45985f4-120f-4101-888d-4db62d71ada4)

## Origin Types & Origin Architecture

If you're using S3 origins, then restricting S3 for only CloudFront is supported out of the box. If you use custom origins (e.g. your web server), then you can pass a header (which is configurable in CloudFront) and handle this header in web server (for example if header is present - return results, otherwise - 403).

## Securing CF and S3 using OAI

When thinking about securing CloudFront delivery path, we need to think about three parts:
* Origins (e.g. S3 or Custom).
* CloudFront Network - Edge locations.
* Public Internet - where consumers reside.

Hence, there are three parts to this path:
* From origin to edge.
* Within Edge.
* From Edge to Customer.

The main problem to address is client bypassing the edge and going directly to origin.

When you use S3 website feature, then origin access is custom origin and NOT S3 origin.

What is Origin Access Identity (OAI):
* An OAI is a type of identity. It is not the same as IAM user or an IAM role, but it does share some characteristics of both.
* It can be associated with CloudFront Distributions. When they are accessing an S3 origin, CloudFront Distribution "becomes" that origin access identity.
* This means that OAI can eb used in S3 bucket policies, so either explicit allows or denies.
* Generally the best practice is lock down S3 access to only being accessible via CloudFront. Deny all but one or more OAIs.

![image](https://github.com/user-attachments/assets/f06e2342-08ab-46d8-a36f-96ba92ad37f0)

We cannot use OAI for custom origins. There are two ways to implement secure infrastructure:
* Utilize custom headers. They are injected in edge location and validated in custom origin. Because HTTPS is used then nobody knows those custom headers.
* AWS public services have known IP ranges. If we have IP ranges that are used by CloudFront, then we can use traditional firewall around custom origin.

![image](https://github.com/user-attachments/assets/057f36a5-c613-428b-8984-72a349107840)

## Private Distribution & Behaviours

CloudFront can run in two security modes when it comes to content:
* Public - in this mode any content which is distributed via CloudFront is public. Can be accessed by any viewer.
* Private - wny request made to CloudFront need to be made with a signed cookie or signed URL or they'll be denied.

CloudFront distributions are created with a single behavior. Hence either the whole distribution is public or private.

There are two ways to configure CloudFront private behavior:
* Legacy way - you had to create CloudFront Key by Account Root User. Once the key is added to said account, that account can be added as a trusted signer. **EXAM NOTE** If you see a trusted signer in question, you will know that this is talking about private distribution.
* Recommended way - Create trusted key groups and assign those as signers. The key groups determine which keys can be used to create signed URLs and signed cookies.

In both ways you required a signer. A signer is an entity or entities which can create signed URLs or signed cookies. Once a signer is added to behavior, the behavior is now private and only signed URLs and 

CloudFront Signed URLs vs Cookies:
* Signed URLs provide access to **EXAM NOTE** **one object and one object only**.
* Use URLs if your client doesn't support cookies.
* Cookies provides access to groups of objects. Use for groups of files/all files of a type (e.g. all cat gifs).
* If you want to preserve a custom URL then signed cookies is the only option.

![image](https://github.com/user-attachments/assets/14d88d0c-4417-40d7-9494-a961759bfe10)

## Lambda@Edge

* You can run lightweight Lambda functions at edge locations.
* Lambdas can adjust traffic between the Viewer and Origin.
* Currently, only Node.js and Python are supported as run times.
* Run only in AWS Public Space (Not VPC).
* Layers are not supported.
* Different time and sizes limits than in normal Lambda Function.

Each connection between Viewer and Edge (request), Edge and Origin (request), Origin back to Edge (response), Edge to Viewer (response) can run a lambda function. Limitations on viewer side are: 128MB and 5 seconds. On Origin side - Normal Lambda MB, but 30 seconds run time.

![image](https://github.com/user-attachments/assets/2a6e9455-f8a8-4bba-8c5b-27c0bf49b4c7)

Some use cases:
* A/B testing. function to present two different versions of the image without creating redirects or changing the URL - viewer request.
* Migration between different S3 Origins - origin request.
* Different objects based on device - origin request.
* Content by country - origin request.

## Global Accelerator

**EXAM NOTE** You'll have to determine when to use CloudFront and when to use Global Accelerator.

Global Accelerator starts with two Anycast IP addresses. Anycast IP is a special type address - normal IP addresses are called Unicast IP, in other words they refer to one thing, one network device. Anycast IPs allow a single IP to be in multiple locations. Routing moves traffic to closest location. 

**EXAM NOTE**. You need to understand the architecture. The customer arrives at one of the Global Accelerator edge locations, because they are using one of the Anycast IP addresses. Their connections will be routed to the closest Global Accelerator edge location. Once the traffic enters Global Accelerator edge location, then it will use AWS private networks, which will require less hops and has significantly better performance.

![image](https://github.com/user-attachments/assets/50e86d86-7835-4d12-88f2-52d7ebfee9ba)

|   |	Cloud Front   | 	Global Accelerator   |
|---|---|---|
| Purpose | Shorter Latency, Security, Content Caching | Shorter Latency, High Availability, IP Caching |
| Underlined mechanism | Cache the content to a location near the customer/client   | Find a optimal way to reach to the host from where the content will delivered. Note here content is not being cached like CDN |
| Use case | HTTP, HTTPS based traffic. Suitable for caching static resources like images, videos | Both HTTP & Non HTTP protocols like TCP, UDP for Gaming, Video streaming, IoT Messaging |
| How is it charged | Based on numbers of HTTP requests along with amount of data transferred | Hourly charges along with amount of data transferred |
| Static IP  | You can not assign static IP to Cloud Front distribution node. Your client will see non deterministic end points for your application | Global Accelerator assign fixed number of Anycast static IPs. So the client can determine the possible IP addresses of the end points |

There are several benefits to anycast:
* First, in steady state, users of an anycast service (DNS is an excellent example) will always connect to the 'closest' server.
* Another advantage is ease of configuration management. Rather than having to configure different DNS servers depending on where a server/workstation is deployed (Asia, America, Europe), you have one IP address that is configured in every location.
* Depending on how anycast is implemented, it can also provide a level of high availability. If the advertisement of the anycast route is conditional on some sort of health check (e.g. a DNS query for a well known domain, in this example), then as soon as a server fails its route can be removed.
* A final advantage is that of horizontal scaling; if you find that one server is being overly loaded, simply deploy another one in a location that would allow it to take some proportion of the overloaded server's requests.

# Advanced VPC Networking

## VPC Flow Logs

* Captures packet's metadata - not content.
* Flow logs work by attaching virtual monitors within a VPC and these can be applied at three different levels:
  * At the VPC level (all ENIs) which monitor every network interface in every subnet within VPC.
  * At the subnet level - all ENIs in given subnet.
  * At a specific ENI directly.
* Flow logs are **not realtime**.
* Log destinations: S3 and CloudWatch logs.
* Flow logs can capcture Accepted, Rejected or All metadata on connections.

![image](https://github.com/user-attachments/assets/c425a366-17da-4f2f-98c2-8e4c9fc8c52a)

VPC Flow Log is a collection of rows and each row has the following fields (highlighter are the most useful ones):
* version
* account-id
* interface-id££™¡
* **srcaddr**
* **dstaddr**
* **srcport**
* **dstport**
* **protocol**
* packets
* bytes
* start
* end
* **action**
* log-status

Some traffic does not show up in logs: to and from 169.254.169.254, DHCP, Amazon DNS and Amazon Windows license.

![image](https://github.com/user-attachments/assets/25519744-767d-46ce-ad49-6a866193dd67)

## Egress-Only Internet gateway

Only allows for connections to be initiated from inside VPC to outside.

* With IPv4 addresses are private or public.
* NAT allows private IPs to access public networks. NAT gateway provides private IPv4 IPs with a way to access the public internet or public AWS services. But, and this is important thing, it doesn't allow any connections from the internet to be initiated to the private instance. NAT exists because of limitations of IPv4. It does not work with IPv4.
* All IPv6 IPs are publicly routable.
* Internet Gateway (IPv6) allows all IP IN and OUT.
* This is why Egress Only Internet Gateway exists - to fill the functionality gap within IPv6 services.
* Egress-Only Gateway is exactly the same as normal internet gateway and has same properties - HA by default accross all AZs. Scales as required.
* To enable flow through Egress-Only IGW - add default IPv6 route `::/0` 

## VPC Endpoints (Gateway)

* Provide private access to S3 and DynamoDB. Normally when you want to access AWS public services from within a VPC, usually an IGW.
* Now the way it works is that you create a gateway endpoint, these are created per service, per region. For example, S3 in the us-east-1. You create this gateway for S3 in us-east-1 and you associate it with one or more subnets in a particular VPC. Gateway endpoints doesn't go into VPC subnets. What happens is that when you allocate the gateway endpoint to particular subnets, something called a prefix list is added to the route tables for those subnets. And this prefix list uses the gateway endpoint as a target. A prefix list is just like what you would find on a normal route but it's an object, it's a logical entity which represents these services. So it represents S3 or DynamoDB. Imagine this as a list of IP addresses that those services use, but where the list is kept updated by AWS. This prefix list is added to the route table, the prefix list is used as the destination and the target is the gateway endpoint. This means in this example, that any traffic destined for S3 as it exits the subnet, it goes via the gateway endpoint rather than the internet gateway. **EXAM NOTE** gateway endpoint does not go into a particular subnet or AZ. It is HA across all AZs in a region by default.
* Endpoint policy is used to control what it can access. For example, allow gateway endpoint to connect to a subset of S3 buckets.
* Regional, can't access cross-region services.
* Another use cases is to **prevent leaky buckets**. S3 buckets can be set to private only by allowing access only from a gateway endpoint.
* **EXAM NOTE**. Gateway endpoints are only accessible from inside specific VPC.

Not using Gateway Endpoints. But this creates a security problem, because resources have public internet access, either directly for public resources or via NAT gateway for private only EC2 instances. If you want instances inside VPC to be able to access S3 without the public internet, then it is problematic. This is where VPC endpoints come in.

![image](https://github.com/user-attachments/assets/f3372ef8-c9d0-4fca-bfbe-af8909ed7b94)

![image](https://github.com/user-attachments/assets/2eb55687-0a77-4a9d-b1b2-1ee291c14b90)

## VPC Endpoints (Interface)

* Just like gateway endpoints, interface endpoints provide private access to AWS public services.
* Historically, it used to provide access to all services apart from S3 and DynamoDB. Historically, those services were only available via gateway endpoints. **S3 is now supported**.
* One crucial difference between gateway endpoints and interface endpoints is that interface endpoints are not HA by default. Interfaces are added to specific VPCs. One subnet means one AZ.
* For HA, add one interface endpoint to one subnet per AZ. If you have 2 AZ then you need 2 interfaces.
* Network access controlled via Security Groups. You don't have this capability with gateway endpoints.
* Endpoint policies also work with interface endpoints.
* Only support TCP and only IPv4.
* Behind the scenes, interface endpoints use **PrivateLink**. It is a product which allows external services to be injected into your VPC, either from AWS or from third parties. Injecting means giving network interface inside your VPC to a service.

Interface endpoints don't work the same way as internet gateway. Gateway endpoints use a prefix list, while interface endpoints primarily use DNS. Interface endpoints just network interfaces inside your VPC. They have a private IP within the range, which subnet uses.

* The way that this works is when you create an interface endpoint in a particular region for a particular service, you get a new DNS name for that service. An endpoint specific DNS name. And that name can be used to directly access the service via the interface endpoint.
* This is an example of DNS name you might get for the SNS service inside us-east-1 region: `vpc-123-xyz.sns.us-east-1.vpce.amazonaws.com`. This name resolves to the private IP address of the interface endpoint and if you update your applications to use this endpoint-specific DNS name, then you can directly use it to access the service via the interface endpoint and not require public IP addressing.

There are multiple DNS names for specific interface endpoint:
* Regional DNS name - which is one single DNS name that works whatever AZ you're using to access the interface endpoint. Its good for simplicity and for HA.
* Each interface in each AZ gets a zonal DNS, which resolves to that one specific interface in that one specific availability zone.
* Applications can optionally use thse or use PrivateDNS.
* PrivateDNS overrides the default DNS for services. PrivateDNS associates a route 53 private hosted zone with your VPC. This privated hosted zone carries a replacement DNS record for the default service endpoint DNS name. 

For example, in normal scenario, if you'd like to reach SNS service, you'd have to resolve a given DNS name, then go through router and IGW to SNS service.

![image](https://github.com/user-attachments/assets/3f79f32a-2dd8-4137-929b-fe72648a9a75)

Interface endpoints architecture without private DNS:

![image](https://github.com/user-attachments/assets/464afb3d-d7f2-4344-8ee1-769ec525e70a)

Interface endpoints architecture with private DNS:

![image](https://github.com/user-attachments/assets/e793b23f-bdfb-490a-b544-5a7806f23256)

The difference between with private DNS and without:
* Interface endpoint resolves to private IP address of interface endpoint. Which then directs traffic to SNS.
* With private DNS, the record is overriden, which means that applications do not need to direct to interface endpoint, but can utilise the same, public DNS name for SNS, however it is overriden in DNS service to point to interface.

## VPC Peering

* VPC peering is a service that lets you create a private and encrypted network link between two VPCs. One peering connection links **two and only two VPCs**.
* Works between VPCs in the same region or cross region. VPCs can be in same account or different.
* (optional) Public Hostnames resolve to private IPs. This means you can use the same DNS names to locate services whether they're in peered VPCs or not. If a VPC peer exists between one VPC and another and this option is enabled, then if you attempt to resolve the public DNS hostname of an EC2 instance it will resolve to the private IP address of that EC2 instance.
* If your VPCs are in the **same region** then they can reference each other's security groups via SG IDs.
* VPC peering does not support transitivie peering. If you have VPC A peered to VPC B, while VPC B is peered to VPC C, that does not mean that VPC A is peered to VPC C.
* To fully configure connectivity between those VPCs, you need to configre routing. Route tables with routes on them pointing at the remote VPC IP address range and using VPC peering connection gateway object as the target. SGs and NACLs have to be configured accordingly.
* **EXAM NOTE!** VPC peering connections cannot be creted where there is an overlap in the VPC CIDRs. Never use the same ranges in multiple VPCs.

![image](https://github.com/user-attachments/assets/0cb90588-6bc9-4bb0-8d97-3188f4969f32)

# HYBRID ENVIRONMENTS AND MIGRATION

## Border Gateway Protocol (BGP) 101

BGP is a routing protocol. AWS Direct Connect and Dynamic VPNs both utilize BGP.

* BGP are made up of lots of autonomous systems (AS). AS could be a large network, could be a collection of routers but in either case they're controlled by one single entity.
* Each AS is alocated a number by IANA - ASN (autonomous system numbers). The range is from 0 to 65535. 64512 - 65534 are private.
* ASNs are the way BGP identifies different entities within the network, different peers. It's the way that BGP can distinguish between your network, or your ASN and my network.
* BGP is design to be reliable and distributed. **It operates over TCP, using port 179**. It is not automatic. You have to manually create a peering relationship between two different Autonomous systems.
* Given autonomous system will learn about networks from any of the peering relationships that it has. Anything that it learns will communicate out to any of its other peers. And that is how the internet is working. All the major networks are busy exchanging routing and topology information.
* BGP is a **path-vector** protocol. It changes the **best path to a destination between peers**. The path is called **ASPATH** (autonomous system path).
* iBGP = internal BGP - routing within an AS.
* eBGP = external BGP - routing between AS.

![image](https://github.com/user-attachments/assets/77ea47a9-014b-4f6b-befd-1bae7693476c)

![image](https://github.com/user-attachments/assets/78758e12-2d1d-4eab-bbf6-f9265b459aa2)

## IPSec VPN Fundamentals

* IPSEC is a group of protocols.
* Their aim is to set up secure networking tunnels accross insecure networks.
* IPSEC provides authentication, so that only peers which are known to each other are connected.
* Any traffic which is carried by IPSEC protocols is encrypted. 

![image](https://github.com/user-attachments/assets/7c30cd8d-2ccd-470b-bab4-1cc743113e75)

IPSEC has two main phases:
* IKE Phase 1 (Slow and heavy) - IKE stands for internet key exchange, as the name suggest is a protocol, for how keys are exchanged. In this context within a VPN.
  * Authenticate - pre-shared key (password) / certificate.
  * Using asymmetric encryption to agree on, and create a shared symmetric key.
  * IKE SA (security association) phase 1 tunnel. Heavy work of moving encryption keys is done.
* IKE Phase 2 (Fast and Aglie)
  * Uses the keys agreed in phase 1.
  * Agree encryption method, and keys used for bulk data transfer.
  * Create IPSEC SA - phase 2 tunnel (architecturally running over phase 1).

There are two phases, because phase 2 can be torn down and recreated later on, when no more "interesting traffic" is observed.

![image](https://github.com/user-attachments/assets/8177e587-1b18-48ed-8ce3-7feed54dd1f0)

![image](https://github.com/user-attachments/assets/007d6889-35ec-4b98-a281-34c047c58cf0)

There are two types of VPNs (the difference is how they match interesting traffic):
* Policy based VPN - rules are created which match traffic. Based on these rules, the traffic is sent over a pair of SA.
* Route based VPN - target matching based on prefix. For example, send traffic for 192.168.0.0/24 over this VPN. With this type of VPN you hava a single pair of security associations for each network prefix.

Route based VPN has one phase one tunnel and one phase two tunnel based on routes.

![image](https://github.com/user-attachments/assets/6cd30d06-636d-45d1-a433-74075ad54d3f)

## AWS Site-to-Site VPN

VPNs are the quickest way to establish a network link between an AWS environment and something that is not AWS (this might be on premise, another cloud environment or a data center).

* A site-to-site VPN is a logical connection between a VPC and on-premise network encrypted using IPSec, running over the **public internet**.
* An exception is when you're running a VPN over the Direct Connect. 
* Site-to-site can be Higly Available (HA) if you design and implement them correctly.
* Quick to provision (less than an hour).
* There are few components required in creating a VPN connection:
  * VPC.
  * Virtual Private Gateway (VGW).
  * Customer Gateway (CGW). This can refer to two different things. It's often used to refer to both the logical piece of configuration within AWS and the thing that configuration represents - a physical, on-premise router which the VPN connects to.
  * VPN connection between the VGW and CGW.
 
When setting up a VPN, we need to gather these data points:
* IP range of VPC inside AWS.
* IP range of customer's on-premise network.
* IP address of the physical router on the customer premise.

Once we have all this information, we can create a virtual private gateway and attached to our VPC. In customer environment we create a logical representation of customer's router - Customer Gateway.

There are two kinds of VPNs - static and dynamic. You need to link it to a virtual private gateway.

![image](https://github.com/user-attachments/assets/135049d2-d4e4-4b08-ad52-52e593772b9d)

This design (picture above) is not fully HA. This is due to customer's router. If it fails, then everything fails. To move to a HA you need to add additional router on customer's side.

![image](https://github.com/user-attachments/assets/b210e115-dd67-4d8b-b86a-bb7031cd4fe6)

### Static vs Dynamic VPN (BGP)

Dynamic VPN uses a protocal called BGP - border gateway protocal. **If you customer router does not support BGP, you cannot use Dynamic VPN**. The benefits of using static VPN is simplicity and it works almost anywhere with any combination of routers, because it only needs IPSec. However, you are restricted on load balancing and multi connection failover.

If you need any advanced high availability, if you need to use multiple connections, if the VPNs need to work with Direct Connect, then you need to use dynamic VPN. With dynamic VPN you can add routes statically or dynamically (enable route propogation).

VPN considerations:
* **EXAM NOTE!** There is a speed cap for VPNs - 1.25 GBps.
* The cap is the same for virtual private gateway as a whole (all VPN conenctions connecting to virtual private gateway). It is also 1.25 GBps.
* VPN connection transits over the public internet - which adds latency, is inconsistent.
* Cost - AWS hourly cost, GB out cost, data cap.
* **EXAM NOTE!** VPNs are very quick to setup.
* Can be used as a backup for Direct Connect (DX).
* Can be used with Direct Connect (DX).

## Direct Connect (DX) Concepts

* A Direct Connect is a physical connection (1, 10 or 100 GBps) in a AWS region.
* A direct connect is between Business Premise -> DX Location -> AWS region.
* When you order direct connect what you are order is actually a port allocation at a DX location. They provide a port and authorization for you to connect to that port. It is up to you to connect to this directly.
* The port has two costs - hourly cost based on the DX location and outbound data transfer. Inbound data transfer is free of charge.
* Provisioning time - it takes time for AWS to allocate a port. Connecting physical cabel to DX location might take weeks or months. No resilience (if the cabel is cut it is cut).
* Low and consistent latency + high speeds.
* DX can access AWS Private Services (VPCs) and AWS Public Services.

![image](https://github.com/user-attachments/assets/da050535-4303-4df2-b575-4004bb46482b)

## Direct Connect (DX) Resilience

![image](https://github.com/user-attachments/assets/463b9e3f-834f-4f01-9e17-d2757eb3cf23)

![image](https://github.com/user-attachments/assets/6739bcbd-6eb1-4cf6-9df4-389300feaf4a)

![image](https://github.com/user-attachments/assets/bcf54d9f-fd55-4e77-a221-ccbde5a6fbfb)

![image](https://github.com/user-attachments/assets/598a3f60-c84f-4239-8476-4cbe224abdb6)

## Direct Connect (DX) - Public VIF + VPN (Encryption)

VIF - virtual interface.

Using a VPN gives you an encrypted and authenticated tunnel. This is true whether you use public internet or run VPN over direct connect. However, by running VPN over DX you get low latency and consistent latency.
* Using private VIF provides access to private IPs only.
* Public VIF provides access to public zones, meaning public IP addresses owned by AWS.

When using VIFs focus on what you're trying to access whether use private or public VIFs.

*UNFINNISHED*

## Transit Gateway

## Storage Gateway - Volume

## Storage Gateway - Tape (VTL)

## Storage Gateway - File

## Directory Service

## DataSync

## FSx for Windows Servers

## FSx For Lustre

## AWS Transfer Family

# SECURITY, DEPLOYMENT & OPERATIONS

## AWS Secrets Manager

It is often confused with SSM Parameter store. Inside the parameter store, you can create secure strings, which allow you to store passwords. So, when should you use parameter store vs secret manager?
* AWS Secrets Manager shares functionality wiht Parameter Store.
* AWS Secret Manager is designed specifically for secrets (e.g. passwords, API keys etc). **EXAM NOTE!** When you see passwords or API keys, you should default to Secret Manager.
* Usable via console, CLI, API and SDK.
* Automatic rotation. Done using lambdas.
* Direct integration with some AWS products like RDS. Not only is the secret rotated in Secret Manager, but also change on the RDS side as well. Keeping them in sync.
* **EXAM NOTE!** If you see any mention of rotating secrets and more specifically rotating secrets with RDS then it's almost always Secret Manager as an answer.

Down below path could be completely covered by Parameter Store.

![image](https://github.com/user-attachments/assets/df43c716-ff7e-4769-9b49-8b6b82ab45bf)

However, rotation and direct integration with products like RDS is not supported in Parameter Store.

![image](https://github.com/user-attachments/assets/3384b6c1-20d4-4bd4-930f-8d2fc6a48bb3)

## Application Layer (L7) Firewall

Normal firewalls (layer 3/4/5):
* Layer 3/4 see packets, segments, ip addresses and ports. It sees two flows of communications - requests coming from the client and responses coming from the server. These flows are perceived as two separate, non related flows.
* Adding layer 5 provides sessions. This allows to see the previous example as one flow.
* Adding layer 7 provides additional level of analysis on headers, data, hosts etc.
* Layer 7 firewalls can support only HTTP, or SMTP, or both etc. Depends on the software in the firewall.

![image](https://github.com/user-attachments/assets/875a1610-76f6-4cd3-ad35-c4a43f9138f4)

![image](https://github.com/user-attachments/assets/16a5de6c-e62d-4989-961a-d1b1567c32e7)

## Web Application Firewall (WAF), WEBACLs, Rule Groups and Rules

WAF is layer 7 firewall. Provided example is fairly complex, however WAF architectures range from fairly simple to such as this:

![image](https://github.com/user-attachments/assets/6ed2ccf1-7c36-4a37-af4f-3ac056d4db27)

Web Access Control List (WEBACL) - the main unit of configuration within WAF.
* A web ACL is what controls if traffic is allowed or blocked.
* A starting point in WEBACL is a default action. An actions which would allow or block if no other rule matches.
* A web ACL is created for either **CloudFront** or **Regional Service** such as ALB, API GW, AppSync.
* If you create a WEBACL for regional service, then you have to pick a region.
* WEBACLs don't do anything. You have to add rule groups or rules. **They are processed in order**.
* Rule groups have limits how much CPU can be dedicated to rules. Rules use CPU.
* Web ACL capacity units (WCU) - default 1500. WCU are indication of the complexity of rules. Default WCU can be increased with a support ticket.
* One WEBACL can be associated with many resources, but resource can have only one WEBACL.

Rule groups
* Rule Groups contain rules.
* They don't have default actions. That is defined when groups or rules are added to WEBACLs.
* Rule groups are **managed** (AWS or MarketPlace), **Yours**, **Service** owned (e.g. shield or firewall manager).
* Rule groups can be referenced by multiple WEBACLs.
* WEBACL can reference one or more rule groups.

WAF Rules
* Rule's structure:
  * type - the type of rule determines at a high level how it works.
  * statement - the statement consists of one or more things that match traffic or not.
  * action - what WAF does if a match occurs.
* Rules are one of two types - regular and rate-based.
  * Regular rules are design to match if something occurs (e.g. allow SSH connections from a certain IP address).
  * Rate-based are design to match if something occurs at a certain rate.
* Statement of the rule defines what the rule checks for. For regular rules, think of this as a what. What does the rule match against (e.g. contains a certain header). For rate based rules, you're going to apply a rate limit.
* You can match against things like origin country, IP address, header, cookies, query parameter, URI path, query string, body (**EXAM NOTE** first 8192 bytes only), HTTP method.
* You can have one statement or multiple. You can combine multiple with AND, OR, NOT.
* Actions: Allow (only for regular rules), Block, Count, Captcha, Custom Response (prefixed with `x-amzn-waf-`), Label.
* Labels can be refered by other rules within a single web ACL. Labels don't persist outside of that.

Pricing
* WEBACL - monthly (5$/month).
* Rule on WEBACL - each role per month 1$.
* Requests per WEBACL - monthly 0.60$ / 1 million req.

## AWS Shield (DDoS Protection)

AWS shield actually comes in two forms:
* AWS Shield standard - free. 
* AWS Shield advanced - costs money.
* AWS Shield protects against three levels of attacks: 1) Network Volumetric Attack (L3) - Saturate Capacity; 2) Network Protocol Attacks (L4) - TCP SYN Flood; 3) Application Layer Attacks (L7) - e.g. web request floods.

Shield Standard:
* Free.
* Protection at the perimeter. Or at the region/VPC or the AWS edge.
* Protects against common L3 and L4 layer attacks.
* Best protection if you use R53, CloudFront, AWS Global Accelerator.

Shield Advanced:
* 3k per month (per ORG), 1 year lock-in + data (OUT) monthly. **Not per account, but per organisation**.
* Protects CloudFront, R53, Global Accelerator, Anything Associated with EIPs (e.g. EC2), ALBs, CLBs, NLBs.
* Not automatic - **you need to explicitly enabled** in Shield Advanced or AWS Firewall Manager Shield Advanced policy.
* Cost protection (e.g. EC2 scaling) for unmitigated attacks. In other words, if it should be mitigated by Shield Advanced and wasn't - the cost is removed.
* Proactive Management the Shield Response Team contacts you directly when availability of your application is affected due to possible attack.
* WAF integration - includes basic AWS WAF fees for web ACLs, rules and web requests.
* Shield Advanced uses the web application firewall to implement its protection against layer-seven attacks (L7 protection).
* Real time visibility of DDoS events and attacks.
* Health-based detection - uses R53 health checks to implement applications specific health checks.
* Protection groups - you can group resources under protection groups and manage this way instead of per resource.

## CloudHSM

































