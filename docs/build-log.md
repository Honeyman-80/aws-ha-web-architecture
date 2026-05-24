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
