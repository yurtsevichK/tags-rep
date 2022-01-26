# VPC - A virtual network dedicated to your AWS account. 


## Table of Content:

- [Your Goals](#your-goals)
- [VPC Overview](#vpc-overview)
- [Amazon Virtual Private Cloud](#amazon-virtual-private-cloud-amazon-vpc)
- [Elastic Network Interfaces](#elastic-network-interfaces-enis)
- [Elastic IP Addresses](#elastic-ip-addresses-eips)
- [VPC Security](#vpc-security)
- [NAT Instances and NAT Gateways](#network-address-translation-nat-instances-and-nat-gateways)
- [VPC Peering](#vpc-peering)
- [VPC Flow Logs](#vpc-flow-logs)
- [More details](#more-details)

## Your Goals:

- To get familiar with AWS Network Architecture components: VPC, Subnets, Private/Public subnets layout, Nat Gateways, Routing Tables, Elastic Network Interfaces(ENI) 
- Explain the difference between Default VPC / Custom VPC, Public/Private Subnets, their usage pros and cons 
- To know how to establish Cross VPC connectivity: VPC Peering, Transit Gateway 
- To know how to secure your Networks (NACL and Security Groups, the difference between them) 
- To get familiar with AWS DNS Service (AmazonProvidedDNS), DHCPs Option Sets 
- Understand Route53: Public/Private Zones, Types of Records (including CNAME and Aliases) 
- Learn about AWS CloudFront(Content Delivery Network)

## VPC Overview 

![](https://blog.gelin.ru/2018/06/01%20VPC,%20subnets.png)

Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the Amazon Web Services (AWS) Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways 

### Concepts for VPCs: 

**Virtual private cloud (VPC)** — A virtual network dedicated to your AWS account. 

**Subnet** — A range of IP addresses in your VPC. 

**Route table** — A set of rules, called routes, that are used to determine where network traffic is directed to. 

**Internet gateway** — A gateway that you attach to your VPC to enable communication between resources in your VPC and the internet. 

**VPC endpoint** — Enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection. Instances in your VPC do not require public IP addresses to communicate with resources in the service. Traffic between your VPC and the other service does not leave the Amazon network.  

**CIDR block** — Classless Inter-Domain Routing. An internet protocol address allocation and route aggregation methodology. For more information, see Classless Inter-Domain Routing in [Wikipedia](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). 

### Use cases / Considerations

What can we do with a VPC? 

- Launch instances into a subnet of your choose  
- Assign custom IP address ranges in each subnet  
- Configure route tables between subnets 
- Create Internet gateway and attach it to our VPC  
- Much better security control over your AWS resources  
- Instance security groups  
- Subnet network access control lists (ACLs) 


## Governance 

Monitoring tools/service: 
- [VPC Flow logs](#vpc-flow-logs)
- [CloudWatch](https://aws.amazon.com/ru/blogs/mt/monitor-network-throughput-of-interface-vpc-endpoints-using-amazon-cloudwatch/) 

## Cautions 

You can create 5 VPCs per Region. The quota for internet gateways per Region is directly correlated to this one. Increasing this quota increases the quota on internet gateways per Region by the same amount. You can have 100s of VPCs per Region for your needs even though the default quota is 5 VPCs per Region.  

You can request an increase for these quotas. For some of these quotas, you can view your current quota using the Limits page of the [Amazon EC2 console](https://console.aws.amazon.com/ec2/v2/home?#Limits:)). 

## Pricing considerations 

https://aws.amazon.com/vpc/pricing/ 
 

# Amazon Virtual Private Cloud (Amazon VPC)

Figure below illustrates an Amazon VPC with an address space of `10.0.0.0/16`, two subnets with different address ranges (`10.0.0.0/24` and `10.0.1.0/24`) placed in different Availability Zones, and a route table with the local route specified.

![](images/2.1.jpg)

An Amazon VPC consists of the following components:
- Subnets
- Route tables
- Dynamic Host Configuration Protocol (DHCP) option sets
- Security groups
- Network Access Control Lists (ACLs)

An Amazon VPC has the following optional components:
- Internet Gateways (IGWs)
- Elastic IP (EIP) addresses
- Elastic Network Interfaces (ENIs)
- Endpoints
- Peering
- Network Address Translation (NATs) instances and NAT gateways
- Virtual Private Gateway (VPG), Customer Gateways (CGWs), and Virtual Private Networks (VPNs)

Figure below illustrates an Amazon VPC with an address space of `10.0.0.0/16`, one subnet with
an address range of `10.0.0.0/24`, a route table, an attached IGW (Internet Gateway), and a single Amazon EC2 instance with a private IP address and an EIP address. 

The route table contains two routes: the local route that permits inter-VPC communication and a route that sends all non-local traffic to the IGW (igw-id). Note that the Amazon EC2 instance has a public IP address (EIP, 198.51.100.2); this instance can be accessed from the Internet, and traffic may originate and
return to this instance.

![](images/2.2.jpg)

## More Complex Set up:

![](images/2.3.png)

## Elastic Network Interfaces (ENIs)

![](images/2.4.png)

An Elastic Network Interface (ENI) is a virtual network interface that you can attach to an instance in an Amazon VPC. ENIs are only available within an Amazon VPC, and they are associated with a subnet upon creation. They can have one public IP address and multiple private IP addresses. If there are multiple private IP addresses, one of them is primary. Assigning a second network interface to an instance via an ENI allows it to be dual-homed (have network presence in different subnets). An ENI created independently of a particular instance persists regardless of the lifetime of any instance to which it is attached; if an underlying instance fails, the IP address may be preserved by attaching the ENI to a replacement instance.

ENIs allow you to create a management network, use network and security appliances in your Amazon VPC, create dual-homed instances with workloads/roles on distinct subnets, or create a low-budget, high-availability solution.

## IPv6
You can optionally associate an IPv6 CIDR block with your VPC and subnets, and assign IPv6 addresses from that block to the resources in your VPC. IPv6 addresses are public and reachable over the internet. Your VPC can operate in dual-stack mode: your resources can communicate over IPv4, or IPv6, or both. IPv4 and IPv6 addresses are independent of each other; you must configure routing and security in your VPC separately for IPv4 and IPv6. 

#### Docs:

[IP Addressing in your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html)

## Elastic IP Addresses (EIPs)

![](images/2.5.png)

AWS maintains a pool of public IP addresses in each region and makes them available for you to associate to resources within your Amazon VPCs. An Elastic IP Addresses (EIP) is a static, public IP address in the pool for the region that you can allocate to your account (pull from the pool) and release (return to the pool). EIPs allow you to maintain a set of IP addresses that remain fixed while the underlying infrastructure may change over time. Here are the important points to understand about EIPs:

- You must first allocate an EIP for use within a VPC and then assign it to an instance.
- EIPs are specific to a region (that is, an EIP in one region cannot be assigned to an instance within an Amazon VPC in a different region).
- There is a one-to-one relationship between network interfaces and EIPs.
- You can move EIPs from one instance to another, either in the same Amazon VPC or a different Amazon VPC within the same region.
- EIPs remain associated with your AWS account until you explicitly release them.
- There are charges for EIPs allocated to your account, even when they are not associated with a resource.

## VPC Security 

Amazon Virtual Private Cloud provides features that you can use to increase and monitor the security for your virtual private cloud (VPC): 

- A **security group** acts as a virtual firewall for your instance to control inbound and outbound traffic. When you launch an instance in a VPC, you can assign up to five security groups to the instance. Security groups act at the instance level, not the subnet level. Therefore, each instance in a subnet in your VPC can be assigned to a different set of security groups. For each security group, you add rules that control the inbound traffic to instances, and a separate set of rules that control the outbound traffic.. Security Groups are Stateful if you send a request from your instance, the response traffic for that request is allowed to flow in regardless of inbound security group rules. Responses to allowed inbound traffic are allowed to flow out, regardless of outbound rules.;  

- A **network access control list (ACL)** is an optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more subnets. You might set up network ACLs with rules similar to your security groups in order to add an additional layer of security to your VPC.  Network Access Control Lists are Stateless, which means that responses to allowed inbound traffic are subject to the rules for outbound traffic (and vice versa).  


The following diagram illustrates the layers of security provided by security groups and network ACLs. For example, traffic from an internet gateway is routed to the appropriate subnet using the routes in the routing table. The rules of the network ACL that is associated with the subnet control which traffic is allowed to the subnet. The rules of the security group that is associated with an instance control which traffic is allowed to the instance:

![](images/2.6.jpg)

### Difference between Security Group and NACL:

| Security Group | NACL |
|---|---|
| Operates at the instance level (first layer of defense) | Operates at the subnet level (second layer of defense) |
| Supports "allow" rules only | Supports allow rules and deny rules |
| Stateful: Return traffic is automatically allowed, regardless of any rules | Stateless: Return traffic must be explicitly allowed by rules. |
| AWS evaluates all rules before deciding whether to allow traffic | AWS processes rules in number order when deciding whether to allow traffic. |
| Applied selectively to individual instances | Automatically applied to all instances in the associated subnets; this is a backup layer of defense, so you don’t have to rely on someone specifying the security group |

### Cautions 

- Network ACLs per VPC - `2000`. You can associate one network ACL to one or more subnets in a VPC. This quota is not the same as the number of rules per network ACL. 
- Rules per network ACL - `20`. This is the one-way quota for a single network ACL. This quota is enforced separately for IPv4 rules and IPv6 rules; for example, you can have 20 ingress rules for IPv4 traffic and 20 ingress rules for IPv6 traffic. This quota includes the default deny rules (rule number `32767` for IPv4 and `32768` for IPv6, or an asterisk * in the Amazon VPC console). 
- This quota can be increased up to a maximum of `40`; however, network performance might be impacted due to the increased workload to process the additional rules 
- VPC security groups per Region – `2500`. This quota applies to individual AWS account VPCs and shared VPCs. If you increase this quota to more than `5000` security groups in a Region, we recommend that you paginate calls to describe your security groups for better performance. 
- Inbound or outbound rules per security group – `60`.  You can have 60 inbound and 60 outbound rules per security group (making a total of 120 rules). This quota is enforced separately for IPv4 rules and IPv6 rules; for example, a security group can have 60 inbound rules for IPv4 traffic and 60 inbound rules for IPv6 traffic. A rule that references a security group or AWS-managed prefix list ID counts as one rule for IPv4 and one rule for IPv6. 
- A quota change applies to both inbound and outbound rules. This quota multiplied by the quota for security groups per network interface cannot exceed 1000. For example, if you increase this quota to 100, we decrease the quota for your number of security groups per network interface to `10`. 
- If you reference a customer-managed prefix list in a security group rule, the maximum number of entries for the prefix lists equals the same number of security group rules. 
- Security groups per network interface – `5`. The maximum is `16`. This quota is enforced separately for IPv4 rules and IPv6 rules. The quota for security groups per network interface multiplied by the quota for rules per security group cannot exceed `1000`. For example, if you increase this quota to `10`, we decrease the quota for your number of rules per security group to `100`.


## Network Address Translation (NAT) Instances and NAT Gateways

![](images/2.7.png)

By default, any instance that you launch into a private subnet in an Amazon VPC is not able to communicate with the Internet through the IGW. This is problematic if the instances within private subnets need direct access to the Internet from the Amazon VPC in order to apply security updates, download patches, or update application software. AWS provides NAT instances and NAT gateways to allow instances deployed in private subnets to gain Internet access. For common use cases, we recommend that you use a NAT gateway instead of a NAT instance. The NAT gateway provides better availability and higher bandwidth, and requires
less administrative effort than NAT instances. 

### NAT Instance

A network address translation (NAT) instance is an Amazon Linux Amazon Machine Image (AMI) that is designed to accept traffic from instances within a private subnet, translate the source IP address to the public IP address of the NAT instance, and forward the traffic to the IGW. In addition, the NAT instance maintains the state of the forwarded traffic in order to return response traffic from the Internet to the proper instance in the private subnet. These instances have the string `amzn-ami-vpc-nat` in their names, which is searchable in the Amazon EC2 console.

To allow instances within a private subnet to access Internet resources through the IGW via a NAT instance, you must do the following:

- Create a security group for the NAT with outbound rules that specify the needed Internet resources by port, protocol, and IP address.
- Launch an Amazon Linux NAT AMI as an instance in a public subnet and associate it with the NAT security group.
- Disable the Source/Destination Check attribute of the NAT.
- Configure the route table associated with a private subnet to direct Internet-bound traffic to the NAT instance (for example, `i-1a2b3c4d`).
- Allocate an EIP and associate it with the NAT instance.

This configuration allows instances in private subnets to send outbound Internet communication, but it prevents the instances from receiving inbound traffic initiated by someone on the Internet.

### NAT Gateway

A NAT gateway is an Amazon managed resource that is designed to operate just like a NAT instance, but it is simpler to manage and highly available within an Availability Zone. 

To allow instances within a private subnet to access Internet resources through the IGW via a NAT gateway, you must do the following:

- Configure the route table associated with the private subnet to direct Internet-bound traffic to the NAT gateway (for example, `nat-1a2b3c4d`).
- Allocate an EIP and associate it with the NAT gateway.

Like a NAT instance, this managed service allows outbound Internet communication and prevents the instances from receiving inbound traffic initiated by someone on the Internet.

> To create an Availability Zone-independent architecture, create a NAT gateway in each Availability Zone and configure your routing to ensure that resources use the NAT gateway in the same Availability Zone.

#### Docs:

[Comparing NAT Gateway and NAT Instance](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html)

## VPC Peering

![](images/2.7.png)

An Amazon VPC peering connection is a networking connection between two Amazon VPCs that enables instances in either Amazon VPC to communicate with each other as if they are within the same network. You can create an Amazon VPC peering connection between your own Amazon VPCs or with an Amazon VPC in another AWS account within a single region. A peering connection is neither a gateway nor an Amazon VPN connection and does not introduce a single point of failure for communication. 

Peering connections are created through a request/accept protocol. The owner of the requesting Amazon VPC sends a request to peer to the owner of the peer Amazon VPC. If the peer Amazon VPC is within the same account, it is identified by its VPC ID. If the peer VPC is within a different account, it is identified by Account ID and VPC ID. The owner of the peer Amazon VPC has one week to accept or reject the request to peer with the requesting Amazon VPC before the peering request expires.

An Amazon VPC may have multiple peering connections, and peering is a one-to-one
relationship between Amazon VPCs, meaning two Amazon VPCs cannot have two peering
agreements between them. Also, peering connections do not support transitive routing.

### VPC peering connections do not support transitive routing:

![](images/2.9.jpg)

In this figure, VPC A has two peering connections with two different VPCs: VPC B and VPC C. Therefore, VPC A can communicate directly with VPCs B and C. Because peering connections do not support transitive routing, VPC A cannot be a transit point for traffic between VPCs B and C. In order for VPCs B and C to communicate with each other, a peering connection must be explicitly created between them.

Here are the important points to understand about peering:

- You cannot create a peering connection between Amazon VPCs that have matching or overlapping CIDR blocks.
- Amazon VPC peering connections do not support transitive routing.
- You cannot have more than one peering connection between the same two Amazon VPCs at the same time.


## VPC Flow Logs 

![](images/2.10.png)

VPC Flow Logs is a feature that enables you to capture information about the IP traffic going to and from network interfaces in your VPC. Flow log data can be published to Amazon CloudWatch Logs or Amazon S3. After you've created a flow log, you can retrieve and view its data in the chosen destination.  
 
### Use cases / Considerations 

Flow logs can help you with a number of tasks, such as: 
- Diagnosing overly restrictive security group rules 
- Monitoring the traffic that is reaching your instance 
- Determining the direction of the traffic to and from the network interfaces 

Flow log data is collected outside of the path of your network traffic, and therefore does not affect network throughput or latency. You can create or delete flow logs without any risk of impact to network performance. 

### Cautions 

To use flow logs, you need to be aware of the following limitations: 

- You cannot enable flow logs for network interfaces that are in the EC2-Classic platform. This includes EC2-Classic instances that have been linked to a VPC through ClassicLink.  

- You can't enable flow logs for VPCs that are peered with your VPC unless the peer VPC is in your account.  

- After you've created a flow log, you cannot change its configuration or the flow log record format. For example, you can't associate a different IAM role with the flow log, or add or remove fields in the flow log record. Instead, you can delete the flow log and create a new one with the required configuration.  

- If your network interface has multiple IPv4 addresses and traffic is sent to a secondary private IPv4 address, the flow log displays the primary private IPv4 address in the dstaddr field. To capture the original destination IP address, create a flow log with the pkt-dstaddr field.  

- If traffic is sent to a network interface and the destination is not any of the network interface's IP addresses, the flow log displays the primary private IPv4 address in the dstaddr field. To capture the original destination IP address, create a flow log with the pkt-dstaddr field.  

- If traffic is sent from a network interface and the source is not any of the network interface's IP addresses, the flow log displays the primary private IPv4 address in the srcaddr field. To capture the original source IP address, create a flow log with the pkt-srcaddr field.  

- If traffic is sent to or sent by a network interface, the srcaddr and dstaddr fields in the flow log always display the primary private IPv4 address, regardless of the packet source or destination. To capture the packet source or destination, create a flow log with the pkt-srcaddr and pkt-dstaddr fields.  

- When your network interface is attached to a Nitro-based instance, the aggregation interval is always 1 minute or less, regardless of the specified maximum aggregation interval.  

Flow logs do not capture all IP traffic. The following types of traffic are not logged:  

- Traffic generated by instances when they contact the Amazon DNS server. If you use your own DNS server, then all traffic to that DNS server is logged.  
- Traffic generated by a Windows instance for Amazon Windows license activation.  
- Traffic to and from 169.254.169.254 for instance metadata.  
- Traffic to and from 169.254.169.123 for the Amazon Time Sync Service.  
- DHCP traffic. 
- Traffic to the reserved IP address for the default VPC router. 
- Traffic between an endpoint network interface and a Network Load Balancer network interface. 

### Pricing considerations 

Data ingestion and archival charges for vended logs apply when you publish flow logs to CloudWatch Logs or to Amazon S3. For more information and examples, see [Amazon CloudWatch Pricing](https://aws.amazon.com/ru/cloudwatch/pricing/). 


## More details:

### Videos:

- [Amazon Virtual Private Cloud (VPC) | AWS Tutorial For Beginners](https://www.youtube.com/embed/fpxDGU2KdkA)
- [AWS VPC & Subnets | Amazon Web Services BASICS](https://www.youtube.com/watch?v=bGDMeD6kOz0)
- [AWS Networking Fundamentals](https://www.youtube.com/watch?v=hiKPPy584Mg)
- [AWS VPC | re:Invent 2015](https://www.youtube.com/watch?v=3qln2u1Vr2E)
- [VPC Flow Logs | What is Flow Logs?](https://www.youtube.com/watch?v=_A5L4jT-K9I)
- [Setup VPC Flow Logs To CloudWatch Log Group](https://www.youtube.com/watch?v=aczp7gCZZlQ)
- [VPC - Lecture](https://www.youtube.com/watch?v=zPjVWiPpT-8)
- [VPC - Practice](https://www.youtube.com/watch?v=sA9We2Gl4eI)
- [VPC - Bastion Host](https://www.youtube.com/watch?v=vTER05sRObI)
- [VPC - Peering, VPN, FlowLogs](https://www.youtube.com/watch?v=dlIvxxydShU)
- [AWS-VPC Demo, Public & Private Subnets, Route Tables, Internet & NAT Gateways](https://www.youtube.com/watch?v=tD9vDv0uyI8&ab_channel=KnowledgeIndiaAWSAzureTutorials)

### Documentation:

- [What is VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Private Addresses RFC1918](https://tools.ietf.org/html/rfc1918)
- [Prefixes](https://www.ripe.net/about-us/press-centre/understanding-ip-addressing)
- [VPC Limits](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)
- [Publishing flow logs to CloudWatch Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-cwl.html)
- [Publishing flow logs to Amazon S3](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-s3.html)
- [Troubleshooting VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-troubleshooting.html)
- [VPC Scenarios](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenarios.html)
- [VPC FAQ](https://aws.amazon.com/ru/vpc/faqs/?nc=sn&loc=5)
- [Linux bastion host](https://aws.amazon.com/ru/quickstart/architecture/linux-bastion/)

### Recommended Trainings:

- [AWS: Networking](https://learn.epam.com/detailsPage?id=2699c11b-d1c8-455a-bf5e-1af537ba363c&source=EXTERNAL_COURSE)
- [AWS Certified Solutions Architect - Associate (SAA-C02): 3 Virtual Private Cloud](https://learn.epam.com/detailsPage?id=b29ebc21-f056-4436-a748-8b7ce12efbd1&source=EXTERNAL_COURSE)
- [AWS Certified Solutions Architect - Associate (SAA-C02): 6 Auto Scaling and Virtual Network Services](https://learn.epam.com/detailsPage?id=93b5715a-7725-4456-8034-a897287b5787&source=EXTERNAL_COURSE)
