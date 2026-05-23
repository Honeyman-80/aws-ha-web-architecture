# Operations Notes

## Dynamic Scaling

A target tracking scaling policy was added to the Auto Scaling Group.

Policy behavior:

- Metric: CPU utilization
- Target value: 70%
- Minimum capacity: 2
- Desired capacity: 2
- Maximum capacity: 4

The system was tested by generating CPU load on an EC2 instance.

Observed behavior:

1. CPU increased on one instance
2. CloudWatch detected high CPU
3. AWS-managed target tracking alarm entered ALARM state
4. Auto Scaling Group increased desired capacity from 2 to 3
5. A new EC2 instance launched
6. The new instance registered with the target group

## Scale In

After stopping CPU stress, the Auto Scaling Group eventually scaled back in.

Important lesson:

- scale out is usually faster
- scale in is usually more conservative
- AWS avoids rapid scale-in/scale-out flapping

## CloudWatch Alarm + SNS Notification

A human-facing CloudWatch alarm was created for high CPU.

Alarm flow:

CloudWatch metric  
→ CloudWatch alarm  
→ SNS topic  
→ email/SMS notification

This demonstrated the difference between:

- automation alarms used by scaling policies
- human alerting alarms used for operational awareness

## Target Tracking Observation

During testing, the AWS-managed target tracking alarm showed behavior that was not immediately obvious from a simple manual CPU average.

Important lesson:

- always inspect actual CloudWatch alarms
- do not assume simplified mental models explain all managed service behavior
- validate scaling behavior using activity history, alarm graphs, and metrics

## Networking Notes

The default VPC contained multiple subnets across multiple Availability Zones.

The default route table included:

- 172.31.0.0/16 → local
- 0.0.0.0/0 → Internet Gateway

Important networking lessons:

- VPCs are region-wide
- subnets are AZ-specific
- public subnet behavior depends on route tables
- Internet Gateways provide internet connectivity
- Security Groups control allowed traffic
- Route Tables control where traffic goes

## Security Notes

The current architecture uses public EC2 instances for learning.

This is acceptable for a lab, but a more secure architecture would use:

Internet  
→ public ALB  
→ private EC2 instances  
→ private database

Future improvement:

- remove public SSH
- use SSM Session Manager
- move EC2 instances into private subnets
