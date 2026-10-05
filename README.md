# AWS Zen VPC Project

## Overview

This project demonstrates the design and deployment of a custom Virtual Private Cloud (VPC) in Amazon Web Services (AWS).

The environment contains both a **public subnet** and a **private subnet**, with a Linux EC2 instance deployed in each subnet.

The public EC2 instance is accessible from the internet using SSH and can communicate with the private EC2 instance. The private EC2 instance is not directly accessible from the internet but can access the internet through a NAT Gateway.

This project was created to gain practical experience with AWS networking concepts including VPCs, subnets, route tables, Internet Gateways, NAT Gateways, security groups, and EC2 networking.

---

# Architecture

## Network Design

| Resource | Configuration |
|---|---|
| VPC | `zen-vpc` |
| VPC CIDR | `10.0.0.0/16` |
| Public Subnet | `10.0.10.0/24` |
| Private Subnet | `10.0.20.0/24` |
| Public EC2 | Linux |
| Private EC2 | Linux |
| Internet Gateway | Attached to VPC |
| NAT Gateway | Located in Public Subnet |

---

## Architecture Diagram

```text
                              Internet
                                  |
                                  |
                         Internet Gateway
                                  |
                                  |
                    +---------------------------+
                    |                           |
                    |      zen-vpc              |
                    |      10.0.0.0/16          |
                    |                           |
                    |                           |
        +----------------------+     +----------------------+
        |                      |     |                      |
        |   Public Subnet      |     |   Private Subnet     |
        |   10.0.10.0/24       |     |   10.0.20.0/24      |
        |                      |     |                      |
        |   Linux EC2          |<--->|   Linux EC2          |
        |   Public Server      |     |   Private Server     |
        |                      |     |                      |
        |   NAT Gateway        |<----|   Outbound Traffic   |
        |                      |     |                      |
        +----------------------+     +----------------------+
                    |
                    |
             Internet Gateway
                    |
                 Internet
```

A graphical AWS architecture diagram can also be added here.

Example:

```markdown
![AWS VPC Architecture](diagrams/aws-vpc-architecture.png)
```

---

# Project Objectives

The main objectives of this project are:

- Create a custom AWS VPC.
- Create separate public and private subnets.
- Deploy Linux EC2 instances into both subnets.
- Allow the public EC2 instance to communicate with the internet.
- Allow SSH access to the public EC2 instance.
- Allow ICMP traffic to the public EC2 instance for testing.
- Allow the public and private EC2 instances to communicate with each other.
- Prevent direct internet access to the private EC2 instance.
- Allow the private EC2 instance to access the internet using a NAT Gateway.
- Configure route tables for public and private network traffic.
- Configure AWS Security Groups to control access.

---

# Step 1 - Create the VPC

A custom VPC named:

```text
zen-vpc
```

was created using the following CIDR block:

```text
10.0.0.0/16
```

This address range provides enough private IP addresses to create multiple subnets inside the VPC.

---

# Step 2 - Create the Subnets

Two subnets were created inside `zen-vpc`.

## Public Subnet

```text
Name: Public Subnet
CIDR: 10.0.10.0/24
```

The public subnet contains resources that need connectivity to the internet.

The public Linux EC2 instance and NAT Gateway are deployed in this subnet.

## Private Subnet

```text
Name: Private Subnet
CIDR: 10.0.20.0/24
```

The private subnet is used for resources that should not be directly accessible from the internet.

The private Linux EC2 instance is deployed in this subnet.

---

# Step 3 - Create an Internet Gateway

An Internet Gateway was created and attached to `zen-vpc`.

The Internet Gateway allows resources inside the public subnet to communicate with the internet.

Traffic flow:

```text
Public EC2
     |
Public Route Table
     |
Internet Gateway
     |
Internet
```

---

# Step 4 - Configure the Public Route Table

A route table was created for the public subnet.

The following internet route was added:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

The public subnet was then associated with this route table.

This allows resources in the public subnet that have public IP addresses to communicate with the internet.

---

# Step 5 - Create the NAT Gateway

A NAT Gateway was deployed inside the **public subnet**.

The NAT Gateway was assigned an Elastic IP address.

The NAT Gateway allows instances inside the private subnet to initiate outbound connections to the internet without allowing unsolicited inbound connections from the internet.

For example, the private EC2 instance can use the NAT Gateway to:

- Download Linux updates.
- Install packages.
- Access software repositories.
- Connect to public internet services.

Traffic flow:

```text
Private EC2
     |
Private Route Table
     |
NAT Gateway
     |
Internet Gateway
     |
Internet
```

The NAT Gateway must be located in the public subnet because it requires access to the Internet Gateway.

---

# Step 6 - Configure the Private Route Table

A separate route table was created for the private subnet.

The following route was configured:

```text
Destination: 0.0.0.0/0
Target: NAT Gateway
```

The private subnet was associated with this route table.

This allows the private EC2 instance to access the internet while remaining inaccessible directly from the internet.

---

# Step 7 - Deploy the Public Linux EC2 Instance

A Linux EC2 instance was created inside:

```text
Public Subnet
10.0.10.0/24
```

The instance is configured with a public IPv4 address so that it can be accessed from outside the VPC.

The public EC2 instance acts as the externally accessible server in this project.

---

# Step 8 - Configure the Public EC2 Security Group

The Security Group for the public EC2 instance allows selected inbound traffic.

## Inbound Rules

### SSH

```text
Protocol: TCP
Port: 22
Source: My IP
```

SSH is restricted to the administrator's public IP address rather than allowing SSH access from the entire internet.

### ICMP

```text
Protocol: ICMP
Source: My IP
```

ICMP is enabled for network testing using the `ping` command.

For a production environment, security rules should always follow the principle of least privilege.

---

# Step 9 - Deploy the Private Linux EC2 Instance

A second Linux EC2 instance was created inside:

```text
Private Subnet
10.0.20.0/24
```

The private EC2 instance does not require a public IP address.

It cannot be directly accessed from the internet.

Instead, administration can be performed through the public EC2 instance, which can act as a jump host or bastion host.

Example connection path:

```text
Administrator
      |
      | SSH
      v
Public EC2
      |
      | SSH
      v
Private EC2
```

---

# Step 10 - Configure the Private EC2 Security Group

The private EC2 Security Group can be configured to allow traffic only from the public EC2 Security Group.

For example:

```text
SSH
TCP Port 22
Source: Public EC2 Security Group
```

ICMP can also be allowed from the public EC2 Security Group for connectivity testing.

Using Security Group references instead of IP addresses makes the configuration more secure and easier to manage.

---

# Communication Between the Servers

Because both instances exist inside the same VPC, AWS automatically provides local routing between the subnets.

The VPC route tables contain a local route similar to:

```text
10.0.0.0/16 -> local
```

This allows resources inside:

```text
10.0.10.0/24
```

and:

```text
10.0.20.0/24
```

to communicate with each other as long as their Security Groups and Network ACLs allow the traffic.

---

# Testing Connectivity

## Test 1 - SSH to Public EC2

From a local computer:

```bash
ssh -i your-key.pem ec2-user@PUBLIC-IP
```

---

## Test 2 - Ping Public EC2

From the local computer:

```bash
ping PUBLIC-IP
```

This verifies ICMP connectivity to the public server.

---

## Test 3 - Test Public to Private Communication

After connecting to the public EC2 instance:

```bash
ping PRIVATE-IP
```

This verifies communication between the public and private subnets.

---

## Test 4 - SSH from Public EC2 to Private EC2

The public EC2 instance can be used as a jump host to reach the private server.

Example:

```bash
ssh ubuntu@PRIVATE-IP
```

SSH access must first be permitted by the private EC2 Security Group.

---

## Test 5 - Test Internet Access From Private EC2

After connecting to the private EC2 instance:

```bash
ping 8.8.8.8
```

or:

```bash
curl https://example.com
```

Successful communication confirms that the private instance is reaching the internet through:

```text
Private EC2
     ↓
Private Route Table
     ↓
NAT Gateway
     ↓
Internet Gateway
     ↓
Internet
```

---

# Route Table Summary

## Public Route Table

```text
Destination        Target

10.0.0.0/16        Local
0.0.0.0/0          Internet Gateway
```

Associated subnet:

```text
10.0.10.0/24
```

---

## Private Route Table

```text
Destination        Target

10.0.0.0/16        Local
0.0.0.0/0          NAT Gateway
```

Associated subnet:

```text
10.0.20.0/24
```

---

# Traffic Flow

## Internet to Public EC2

```text
Internet
   ↓
Internet Gateway
   ↓
Public Route Table
   ↓
Public Subnet
   ↓
Public EC2
```

---

## Private EC2 to Internet

```text
Private EC2
   ↓
Private Subnet
   ↓
Private Route Table
   ↓
NAT Gateway
   ↓
Internet Gateway
   ↓
Internet
```

---

## Public EC2 to Private EC2

```text
Public EC2
   ↓
VPC Local Routing
   ↓
Private EC2
```

---

# Security Considerations

This project follows several basic AWS networking security practices:

- The private EC2 instance does not have a public IP address.
- The private EC2 instance is not directly accessible from the internet.
- SSH access to the public EC2 instance should be restricted to a trusted IP address.
- Security Groups control communication between EC2 instances.
- The NAT Gateway allows outbound internet access without exposing the private server.
- Public and private workloads are separated into different subnets.

In a production environment, additional security controls could include:

- AWS Systems Manager Session Manager instead of SSH.
- Network ACL configuration.
- AWS CloudTrail.
- Amazon GuardDuty.
- VPC Flow Logs.
- AWS WAF.
- Centralized logging and monitoring.

---

# AWS Services Used

This project uses the following AWS services and components:

- Amazon VPC
- Amazon EC2
- Public Subnets
- Private Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Elastic IP
- Security Groups

---

# Skills Demonstrated

This project demonstrates practical knowledge of:

- AWS networking.
- VPC design.
- IPv4 CIDR addressing.
- Subnetting.
- Public and private subnet architecture.
- EC2 networking.
- Linux administration.
- Routing.
- Internet Gateways.
- NAT Gateways.
- Security Groups.
- SSH.
- Network troubleshooting.
- Cloud security fundamentals.

---

# Project Structure

```text
aws-zen-vpc/
│
├── README.md
│
├── diagrams/
│   └── aws-vpc-architecture.png
│
├── screenshots/
│   ├── vpc.png
│   ├── subnets.png
│   ├── route-tables.png
│   ├── internet-gateway.png
│   ├── nat-gateway.png
│   └── ec2-instances.png
│
└── docs/
    └── setup-guide.md
```

---

# Future Improvements

Future versions of this project could include:

- Rebuilding the environment using AWS CloudFormation.
- Rebuilding the environment using Terraform.
- Adding multiple Availability Zones.
- Deploying an Application Load Balancer.
- Adding Auto Scaling.
- Creating dedicated web and application tiers.
- Adding an RDS database in a private subnet.
- Implementing VPC Flow Logs.
- Using AWS Systems Manager instead of SSH.
- Adding monitoring with Amazon CloudWatch.
- Hosting a web server on the public EC2 instance.
- Creating the entire environment using Infrastructure as Code.

---

# What I Learned

Through this project, I gained hands-on experience designing and configuring AWS network infrastructure.

I learned how public and private subnets differ, how AWS route tables control traffic, how an Internet Gateway provides public internet connectivity, and why a NAT Gateway must be placed inside a public subnet.

I also learned how Security Groups control communication between EC2 instances and how AWS automatically provides local routing between subnets inside the same VPC.

This project helped strengthen my understanding of AWS networking concepts and how cloud infrastructure can be designed securely using public and private network segments.

---

# Author

**Mind of Zen**

AWS Networking / Cloud Infrastructure Practice Project


FINAL CONFIGURATION


Component	                        Configuration
VPC	                                10.0.0.0/16
Public subnet	                    10.0.10.0/24
Private subnet	                    10.0.20.0/24
Internet Gateway	                Attached to zen-vpc
Public route	                    0.0.0.0/0 → IGW
Private route	                    0.0.0.0/0 → NAT Gateway
Public EC2	Public IP =             Yes
Private EC2	Public IP =             No
Public EC2 Internet	                ✅
Private EC2 Internet outbound	    ✅ via NAT
Internet → Public EC2 SSH	        ✅ from your IP
Internet → Public EC2 ICMP	        ✅
Internet → Private EC2	            ❌
Public EC2 → Private EC2	        ✅
Private EC2 → Public EC2	        ✅ subject to SG rules