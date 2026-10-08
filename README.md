# AWS Secure Cloud Architecture

A cloud security architecture project built on Amazon Web Services to redesign a vulnerable on-premises environment following a ransomware incident.

The solution focused on network segmentation, controlled administrative access, private application and database tiers, resilience, scaling, and reducing lateral movement through layered AWS security controls.

**Technologies:** AWS, VPC, EC2, RDS MySQL, Application Load Balancer, Auto Scaling, S3, Security Groups, Bastion Host, NAT Gateway, Internet Gateway, Route Tables

## Project Overview

The project was based around a fictional company scenario called **WinLocal Giveaways**.

The company had suffered a ransomware attack that exposed weaknesses in its flat on-premises network. The goal was to design and implement a proof-of-concept AWS architecture that improved security, resilience, availability, and scalability.

The environment was designed around:

- One AWS VPC
- Two Availability Zones
- Public subnets
- Private application subnets
- Private database subnets
- Application Load Balancer
- EC2 web servers
- Bastion host
- Amazon RDS MySQL
- Auto Scaling
- S3 failover page
- Layered security groups

## Architecture

The VPC used a segmented architecture across two Availability Zones.

Public subnets contained internet-facing services such as the Application Load Balancer and bastion host.

Private application subnets contained EC2 web servers with no direct public exposure.

Private database subnets contained Amazon RDS, isolated from direct internet access.

This design reduced the risk of lateral movement compared with the original flat network.

![AWS VPC resource map](screenshots/vpc-resource-map.png)

## Bastion Host

Administrative access to private EC2 instances was routed through a bastion host located in the public subnet.

The bastion security group only allowed SSH access from authorised administrative addresses, creating a controlled entry point into the private network.

![Bastion security group rules](screenshots/bastion-security-group.png)

SSH access to the private web server was performed through the bastion host rather than exposing the private instance directly to the internet.

![SSH through bastion to private web server](screenshots/ssh-through-bastion.png)

## Private Web Server

The web server ran on Amazon Linux 2023 inside a private application subnet.

It had no public IP address and received user traffic through the Application Load Balancer.

This reduced direct exposure while still allowing the application to remain externally accessible.

## Amazon RDS

Amazon RDS MySQL was used to store application data.

The database was placed inside the private database tier with public accessibility disabled.

Only the web server security group was permitted to connect to the database on MySQL port 3306.

![RDS private connectivity](screenshots/rds-private-connectivity.png)

## Application Load Balancer

An Application Load Balancer was deployed across the public-facing layer.

It distributed incoming requests across healthy application servers and provided a more resilient replacement for the company's unreliable existing load balancer.

![Application Load Balancer](screenshots/alb-active.png)

## Auto Scaling

An Auto Scaling Group was configured to maintain one web server under normal conditions and scale to a second instance when CPU utilisation crossed the configured threshold.

The group used private application subnets across both Availability Zones.

![Auto Scaling Group](screenshots/auto-scaling-group.png)

## Security Group Chain

The architecture used separate security groups for each layer.

### SG-ALB
Attached to the Application Load Balancer and allowed HTTP traffic from external users.

### SG-Bastion
Attached to the bastion host and allowed SSH access only from authorised administrator IP addresses.

### SG-Web
Attached to the private web servers.

It allowed:

- HTTP traffic from the ALB
- SSH traffic from the bastion host

No direct internet access was permitted.

![Web server security group](screenshots/sg-web.png)

### SG-RDS
Attached to the RDS database.

It allowed MySQL traffic on port 3306 only from the web server security group.

![RDS security group](screenshots/sg-rds.png)

This security-group chain limited communication between tiers and reduced the potential for lateral movement.

## S3 Failover Page

A static maintenance page was hosted using Amazon S3 to provide users with a fallback page if the main application became unavailable.

This provided a lightweight and low-cost recovery option without requiring another EC2 instance.

## Security Concepts Demonstrated

- Network segmentation
- Defence in depth
- Least privilege
- Bastion-host architecture
- Private application tiers
- Private database tiers
- Controlled east-west traffic
- Reduced attack surface
- Security-group chaining
- Resilience across Availability Zones
- Load balancing
- Automatic scaling
- Backup/failover design

## Challenges Addressed

The architecture was designed to address several issues from the original environment:

- unreliable load balancing
- database storage failures
- traffic spikes
- lack of resilience
- ransomware propagation through a flat network
- excessive direct exposure between systems

The redesign used segmentation, controlled communication between tiers, managed database services, and automated scaling to reduce those risks.

## Production Improvements

Some services were identified as future production improvements rather than part of the proof of concept.

These included:

- AWS WAF
- Amazon CloudWatch
- Route 53 DNS failover
- second NAT Gateway
- Multi-AZ RDS standby

## Sustainability Considerations

The design also considered cost and resource efficiency.

Examples included:

- using appropriately sized EC2 and RDS instances
- scaling capacity only when needed
- using S3 static hosting for the failover page instead of another server

## Technologies Used

- Amazon Web Services
- Amazon VPC
- Amazon EC2
- Amazon RDS MySQL
- Application Load Balancer
- Auto Scaling
- Amazon S3
- Security Groups
- Bastion Host
- NAT Gateway
- Internet Gateway
- Route Tables
- Amazon Linux 2023

## Project Documentation

The full project report is available in the `Docs` directory.

Additional screenshots of the AWS architecture and security controls are available in the `screenshots` directory.

## What I Learned

This project provided hands-on experience with:

- designing segmented AWS networks
- implementing public and private subnet architectures
- reducing lateral movement through security groups
- securing administrative access with a bastion host
- isolating databases from public access
- configuring load balancing
- using Auto Scaling for resilience and demand handling
- designing cloud environments around security and availability requirements
