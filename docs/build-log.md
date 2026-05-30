## Migrated EC2 Instances to Private Subnets

- Created two private subnets across separate Availability Zones
- Created a private route table
- Associated private subnets with the private route table
- Created a public NAT Gateway for outbound internet access
- Added private route table default route to NAT Gateway
- Updated the Auto Scaling Group to launch instances into private subnets
- Replaced old public instances one at a time
- Verified new instances have no public IPv4 addresses
- Verified target group health remained healthy
- Verified the web application remains accessible through the ALB

Architecture pattern:

Internet → Public ALB → Private EC2 instances

## Separated ALB and EC2 Security Groups

- Created dedicated ALB security group
- Attached ALB security group to the Application Load Balancer
- Updated EC2 security group rules
- Removed direct HTTP access from the internet to EC2 instances
- Allowed HTTP traffic only from the ALB security group
- Verified the application remained accessible through the ALB

Security pattern:

Internet → ALB → Private EC2

## Added Private Subnet Network ACL

- Created custom private subnet NACL
- Associated it with both private web subnets
- Added inbound HTTP rule for ALB-to-EC2 traffic
- Added outbound ephemeral port rule for EC2-to-ALB response traffic
- Verified the website still works through the ALB

Key lesson:

Security Groups are stateful.
Network ACLs are stateless.

## Added Private RDS Database Tier

- Created dedicated DB subnets across multiple AZs
- Created DB route table
- Created RDS DB subnet group
- Created dedicated RDS security group
- Deployed private MySQL RDS instance
- Disabled public database access
- Connected from private EC2 instance to private RDS instance using Session Manager
- Verified database connectivity and initial database creation

Key lesson:

Databases should remain private and only be reachable from trusted application tiers.

## Converted Static Web Tier into Dynamic Flask Application

- Updated Launch Template user data
- Replaced static Apache HTML page with Python Flask application
- Added dynamic RDS-backed visit counter
- Verified application inserts and retrieves data from MySQL RDS
- Verified multiple EC2 instances share centralized database state
- Verified ALB distributes traffic across instances

Architecture pattern:

Browser → ALB → Flask EC2 instances → RDS MySQL

Key lesson:

Application instances are ephemeral.
Persistent shared state should live in managed database services like RDS.

## Added Operational Health Checks and Observability

- Added dedicated `/health` endpoint for ALB health checks
- Updated target group health check path from `/` to `/health`
- Prevented ALB health checks from modifying application data
- Created EC2 IAM role for observability
- Installed CloudWatch Agent through Launch Template user data
- Added memory and disk metrics to CloudWatch
- Verified CloudWatch Agent metrics from both EC2 instances

Key lesson:

Operational endpoints should be separated from business application logic.

## Added AWS WAF Protection Layer

A Web ACL was deployed and attached to the Application Load Balancer.

Architecture update:

Internet → WAF → ALB → Private EC2 → Private RDS

### IP Allowlist

Created an IP-based allow rule for the administrator IP address.

Purpose:

* Prevent accidental lockout during testing
* Allow administrative access regardless of other WAF rules

Key lesson:

* WAF evaluates rules in priority order
* Allow rules stop further evaluation

### Geographic Restriction

Created a geo-based rule that blocks requests originating outside:

* Canada
* United States

Implementation:

* Block action
* NOT statement enabled
* Countries selected: Canada and United States

Logic:

If request does NOT originate from Canada or the United States → Block

### Rate Limiting

Created a rate-based rule:

* Limit: 20 requests
* Evaluation window: 5 minutes
* Action: Block

Testing:

* Repeated page refreshes triggered WAF protection
* Browser received 403 Forbidden response
* WAF metrics showed blocked requests

Results:

* Total requests: 30
* Allowed requests: 27
* Blocked requests: 3

### SQL Injection Protection

Added AWS Managed Rules SQLi Rule Set.

Testing:

* Submitted SQL injection test pattern in URL query string
* WAF detected request and returned 403 Forbidden

Key lesson:

* WAF blocks malicious requests before they reach the application

### Health Endpoint Protection

Created custom URI path rule:

* URI path = /health
* Action = Block

Rule ordering allows administrator access while preventing public access to operational endpoints.

### Key Lessons

* WAF protects Layer 7 HTTP/HTTPS traffic
* Security Groups protect Layer 3/4 network traffic
* Rule order is critical
* Allow rules stop evaluation
* Managed rule groups provide protection against common attack patterns
* Rate limiting helps mitigate abusive traffic
* WAF can block malicious requests before they reach application servers
