## Operational Recovery, Observability, and Health Check Improvements

The architecture was tested using a full operational shutdown and recovery workflow to better understand infrastructure dependencies, cost optimization, and recovery procedures.

### Cost Optimization Shutdown Procedure

To reduce AWS lab costs during inactive periods:

* Auto Scaling Group desired capacity was reduced to 0
* EC2 instances were terminated automatically by the ASG
* The RDS database was temporarily stopped
* The Application Load Balancer (ALB) was deleted
* NAT Gateways were deleted

Infrastructure configuration components were preserved, including:

* VPC
* subnets
* route tables
* security groups
* Network ACLs
* Launch Templates
* Auto Scaling Group
* Target Groups
* IAM roles
* Systems Manager Parameter Store configuration

### Recovery Procedure

The environment was later rebuilt using the following recovery process:

1. Start the RDS database
2. Recreate a public NAT Gateway
3. Update private route table default route:

   * `0.0.0.0/0 → NAT Gateway`
4. Recreate the internet-facing ALB in public subnets
5. Reattach the existing target group
6. Restore Auto Scaling Group desired capacity

This demonstrated that cloud infrastructure can often be reconstructed from configuration and architecture definitions rather than relying on individual servers.

### Troubleshooting and Dependency Analysis

Several operational troubleshooting scenarios were encountered during recovery.

Observed issues included:

* unhealthy ALB targets
* request timeouts
* broken outbound connectivity
* incorrect launch template versions
* missing application behavior
* HTTPS vs HTTP listener mismatches

Troubleshooting followed a layered approach:

Browser
→ ALB listener
→ Target Group health
→ EC2 application
→ RDS connectivity
→ NAT routing
→ Security Groups
→ Network ACLs

Important lesson:

Troubleshooting distributed systems is often about identifying which layer in the request path is failing.

### Flask Application Migration

The web tier was upgraded from a static Apache page to a dynamic Python Flask application.

The application now:

* runs automatically through EC2 user data
* connects to MySQL RDS
* inserts visit records into the database
* retrieves shared visit counts
* displays instance identity information

Architecture flow:

Browser
→ ALB
→ Flask EC2 instances
→ RDS MySQL database

This demonstrated that application instances are ephemeral while persistent shared state should live in managed database services.

### Dedicated Health Check Endpoint

Originally, the ALB health check path used `/`.

This unintentionally caused ALB health checks to:

* execute application logic
* write records into the database
* increase the visit counter

A dedicated operational endpoint was added:

* `/health`

The `/health` endpoint only returns:

* HTTP 200 OK

Target group health checks were updated to use `/health` instead of `/`.

Important lesson:

Operational health checks should remain lightweight and separate from business application behavior.

### CloudWatch Agent and Observability

An EC2 IAM role was created to support observability and Systems Manager integration.

Attached policies included:

* AmazonSSMManagedInstanceCore
* CloudWatchAgentServerPolicy

CloudWatch Agent was installed automatically through Launch Template user data.

The agent configuration was stored centrally in Systems Manager Parameter Store.

Additional metrics collected:

* memory utilization
* disk utilization

Metrics were successfully verified in CloudWatch under the `CWAgent` namespace.

Important lesson:

Default EC2 metrics provide limited visibility. Production systems often require additional observability agents and centralized operational telemetry.
