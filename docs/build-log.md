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
