# Architecture Notes

# High-Level Traffic Flow

Browser  
→ Application Load Balancer (ALB)  
→ Target Group  
→ EC2 instances  
→ Web server response

The ALB distributes incoming traffic across multiple EC2 instances running in different Availability Zones.

This improves:
- availability
- fault tolerance
- traffic distribution

---

# Auto Scaling Group (ASG)

The Auto Scaling Group maintains the desired number of EC2 instances.

Configuration used:

- Desired capacity: 2
- Minimum capacity: 2
- Maximum capacity: 4

Current behavior:
- maintain 2 healthy instances
- automatically replace failed instances

---

# Launch Template

The Launch Template acts as a reusable server blueprint.

It defines:
- Amazon Linux 2023 AMI
- instance type
- security group
- startup automation
- user data script

This allows infrastructure to become reproducible instead of manually configured.

---

# User Data Automation

User data automatically:
- installs Apache
- starts the web server
- generates a webpage during boot

This demonstrates infrastructure bootstrapping.

---

# Multi-AZ Design

Instances were deployed across multiple Availability Zones using multiple subnets in the default VPC.

This protects against:
- single instance failure
- single AZ failure

---

# Failure Recovery Test

An EC2 instance was intentionally terminated.

Observed behavior:

1. ALB stopped routing traffic to unhealthy instance
2. ASG detected reduced capacity
3. ASG launched replacement instance
4. New instance passed health checks
5. ALB resumed routing traffic

This demonstrated a self-healing infrastructure pattern.

---

# Security Concepts

Security Groups acted as virtual firewalls.

Important security decisions:
- allowed HTTP traffic only
- avoided exposing SSH publicly
- applied least privilege networking concepts

---

# Important Mental Models Learned

- servers are disposable
- architecture is the system
- infrastructure should be reproducible
- load balancers distribute traffic
- ASGs maintain healthy capacity
- health checks drive recovery behavior

## Public Entry Layer / Private Compute Layer

The Application Load Balancer remains in public subnets because it receives traffic from the internet.

The EC2 instances now run in private subnets. They do not have public IPv4 addresses and cannot be reached directly from the internet.

Traffic flow:

Internet → Internet-facing ALB → Private EC2 instances

Outbound-only internet access from private instances is provided through a NAT Gateway.

This improves security because the only public entry point is the ALB, while compute resources remain isolated in private subnets.

## Security Group Separation Pattern

The architecture was updated to separate ALB and EC2 security responsibilities.

ALB security group:
- Allows HTTP 80 from the internet (0.0.0.0/0)

EC2 instance security group:
- Allows HTTP 80 only from the ALB security group
- Does not allow direct HTTP access from the internet

Traffic flow:

Internet → ALB Security Group → ALB → EC2 Security Group → EC2

This reduces attack surface and ensures backend instances only accept traffic from the load balancer.

## Systems Manager Access

Session Manager was tested for private EC2 operational access.

The account has Systems Manager Default Host Management Configuration enabled, which allows EC2 instances to be managed by Systems Manager without manually attaching an SSM instance profile role to each instance.

This allows private instances to be accessed through Session Manager without:
- public IP addresses
- inbound SSH access
- bastion hosts

Operational pattern:

Admin → AWS Systems Manager Session Manager → Private EC2

## Private Subnet Network ACL

A custom Network ACL was added to the private subnets.

The private NACL allows:
- inbound HTTP 80 from the VPC CIDR
- outbound ephemeral response ports 1024-65535 back to the VPC CIDR

This demonstrates that Network ACLs are stateless, so both request and response traffic must be explicitly allowed.

Traffic path:

Internet → ALB → Private Subnet NACL → EC2 Security Group → EC2
