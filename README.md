# AWS Highly Available Web Architecture

## Project Goal

Build a highly available web architecture on AWS using:

- EC2
- Application Load Balancer (ALB)
- Auto Scaling Group (ASG)
- Multiple Availability Zones
- CloudWatch concepts
- Infrastructure automation with Launch Templates

This project focused on learning:

- high availability architecture
- traffic distribution
- self-healing systems
- infrastructure automation
- operational thinking
- AWS networking fundamentals

---

# Architecture

Browser  
→ Application Load Balancer (ALB)  
→ Target Group  
→ Auto Scaling Group (ASG)  
→ EC2 instances across multiple AZs

---

# AWS Services Used

- Amazon EC2
- Application Load Balancer
- Auto Scaling Group
- Launch Templates
- Security Groups
- Default VPC
- Multi-AZ subnets

---

# Key Concepts Learned

## High Availability

The application remained available even after terminating an EC2 instance because:

- the ALB routed traffic only to healthy targets
- the ASG automatically launched a replacement instance
- instances were distributed across multiple Availability Zones

---

## Launch Templates

Launch Templates allow infrastructure to become reproducible instead of manually configured.

The template defined:

- AMI
- instance type
- security groups
- startup automation
- user data

---

## User Data Automation

EC2 user data was used to:

- install Apache
- start the web server
- automatically generate a webpage during boot

This demonstrated infrastructure bootstrapping and server automation.

---

## Security Concepts

Security Groups were used as virtual firewalls.

Key lessons:

- avoid exposing SSH to the internet
- allow only required traffic
- reduce attack surface
- apply least privilege networking principles

---

## Operational Lessons

This project demonstrated:

- self-healing infrastructure
- health checks
- load balancing
- distributed traffic flow
- failure recovery
- disposable infrastructure mindset

---

# Failure Test Performed

One EC2 instance was intentionally terminated.

Observed behavior:

1. ALB stopped routing traffic to unhealthy instance
2. ASG detected reduced capacity
3. ASG launched replacement instance
4. New instance became healthy
5. ALB resumed traffic distribution

This validated the self-healing HA architecture design.
