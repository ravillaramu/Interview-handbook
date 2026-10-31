## AWS Scenario-Based Questions and Answers

### 1) An EC2-hosted application becomes unreachable after a deployment. How do you investigate?

```text
Answer:
- Confirm the instance state and system status checks in EC2.
- Check the load balancer target health, listener rules, and recent deployment events.
- Verify the instance security group allows the required inbound traffic only from the expected source, such as the load balancer.
- Check route tables, network ACLs, DNS, and whether the application is listening on the expected interface and port.
- Review application, system, and cloud-init logs; compare the deployment with the last known-good release.
- Roll back if customer impact is ongoing, then determine the root cause and add a deployment health gate.

Useful checks include `systemctl status <service>`, `journalctl -u <service>`, and the EC2 console's status checks.
```

### 2) An application on a private EC2 instance cannot reach an external package repository. What do you check?

```text
Answer:
- Verify the private subnet's route table points internet-bound traffic to a NAT gateway or another approved egress path.
- Confirm the NAT gateway is in a public subnet whose route points to an internet gateway, and that it is available.
- Check security group egress rules, network ACL rules in both directions, DNS resolution, and proxy requirements.
- Test name resolution and connectivity from the instance, and inspect VPC Flow Logs for rejected traffic.
- Prefer VPC endpoints for supported AWS services such as S3 or Systems Manager when that meets the requirement.
```

### 3) A service intermittently returns 5xx errors behind an Application Load Balancer. How do you troubleshoot?

```text
Answer:
- Use ALB access logs, CloudWatch metrics, and target group health checks to determine whether the load balancer or targets return the errors.
- Compare `HTTPCode_ELB_5XX` with `HTTPCode_Target_5XX`, target response time, and healthy host count.
- Check application logs, resource saturation, connection pools, timeout alignment, and recent deployments.
- Verify health-check path, port, expected response codes, and deregistration delay.
- Mitigate by rolling back or removing unhealthy targets, then fix the underlying issue and test failover.
```

### 4) A workload needs access to S3. How do you grant access securely?

```text
Answer:
- Assign an IAM role to the compute workload rather than storing long-lived access keys.
- Grant only the required actions and bucket or object resources; use conditions where appropriate.
- Check both identity policies and the bucket policy, along with any permissions boundary, SCP, or KMS key policy that may affect access.
- Use CloudTrail data events where required for object-level audit and monitor denied requests.
- Validate access with the workload's role and remove unnecessary permissions.
```

### 5) An S3 bucket must hold sensitive data but still be usable by an application. What controls do you apply?

```text
Answer:
- Block public access at the account and bucket levels unless a reviewed use case requires otherwise.
- Enable server-side encryption, preferably with a managed KMS key when key-level access control or audit is needed.
- Restrict access with least-privilege IAM and bucket policies, and require TLS in transit.
- Enable versioning and an appropriate lifecycle/retention policy; define recovery and deletion protections to meet the data requirements.
- Monitor policy changes and access through CloudTrail and AWS Config, and test backups or recovery procedures.
```

### 6) Traffic spikes cause an application to become slow. How would you scale and protect it?

```text
Answer:
- Use CloudWatch and load-balancer metrics to determine whether the bottleneck is compute, database, network, or a downstream dependency.
- Scale stateless application instances horizontally with an Auto Scaling group and an ALB; use suitable target-tracking metrics and health checks.
- Check database connection capacity and query performance before simply adding application instances.
- Add caching or queue-based buffering where the application architecture supports it.
- Set sensible scaling limits and alarms, load-test changes, and verify that scaling down does not interrupt in-flight work.
```

### 7) An IAM user or workload receives `AccessDenied` even though a policy appears to allow the action. What do you inspect?

```text
Answer:
- Confirm which principal and role session made the request, the exact action, resource ARN, and request context.
- Check identity-based and resource-based policies, permissions boundaries, session policies, SCPs, and explicit denies.
- For KMS or S3, inspect the relevant key or bucket policy as well.
- Use IAM Policy Simulator as a diagnostic aid and inspect CloudTrail for the denied request.
- Apply the smallest policy correction and retest with the same principal; do not solve it by granting administrator access.
```

### 8) Your team needs reliable access to private EC2 instances without opening SSH to the internet. What do you recommend?

```text
Answer:
- Use AWS Systems Manager Session Manager with instance profiles, managed-node prerequisites, and controlled IAM permissions.
- Provide private connectivity to Systems Manager through NAT or interface VPC endpoints.
- Use centralized session logging to CloudWatch Logs or S3 where required.
- Remove public IPs and broad inbound SSH rules; use time-bound break-glass access only if operationally necessary.
- Test access, audit trails, and recovery procedures before removing the existing access path.
```

### 9) AWS costs increased sharply this month. How do you find and address the cause?

```text
Answer:
- Use Cost Explorer and cost allocation tags to identify which service, account, region, or workload changed.
- Compare usage and rates with the prior period; inspect recent deployments, traffic, storage growth, and data transfer.
- Check for unattached volumes, idle load balancers, oversized instances, excessive NAT/data-transfer costs, and forgotten test resources.
- Right-size or schedule non-production resources where safe, and evaluate Savings Plans or Reserved Instances for steady usage.
- Add budgets and anomaly alerts, then verify savings without violating availability or performance targets.
- Review Unused Resources: Regularly audit resources (like EC2 instances, EBS volumes, and Elastic IPs) and terminate or downsize those that are not in use.
```

### 10) A production AWS change caused an outage. How do you restore service and prevent recurrence?

```text
Answer:
- Declare the incident, communicate impact, and stop or roll back the risky change using the established deployment procedure.
- Use CloudTrail, CloudWatch, deployment records, and AWS Config history to identify what changed and when.
- Validate recovery using service health checks and customer-facing signals, not just instance state.
- Preserve relevant logs and write a blameless post-incident review with corrective actions and owners.
- Reduce recurrence with infrastructure as code, peer review, staged rollout, automated policy checks, and tested rollback or recovery plans.
```

### 11) You need to monitor the performance of your AWS resources. What AWS services would you use for monitoring and logging, and what metrics would you track?

```text
Answer:
- Amazon CloudWatch: To monitor AWS resources and applications in real time. You can track metrics like CPU utilization, memory usage, disk I/O, and network traffic.
- AWS CloudTrail: To log API calls and monitor account activity across AWS infrastructure, providing governance and compliance information.
- AWS X-Ray: To analyze and debug distributed applications, visualizing traces to identify performance bottlenecks.
Amazon GuardDuty: To continuously monitor for malicious or unauthorized behavior and provide security alerts.
```

### 12) Your organization wants to ensure compliance and security for its AWS resources. What AWS services would you recommend for auditing and governance?

```text
Answer:
- AWS CloudTrail: To enable logging of all API calls across your AWS account, providing a complete history of account activity.
- AWS Config: To track resource configurations and compliance against predefined rules, helping to identify non-compliant resources.
- AWS Security Hub: To provide a comprehensive view of your security posture across AWS accounts, aggregating findings from various AWS security services.
- Amazon GuardDuty: For continuous monitoring of malicious activity and unauthorized behavior in your AWS env
```

### 13) You need to provide temporary access to an EC2 instance for a developer without sharing permanent SSH keys. How can you do this securely?

### 14) You need a durable and cost-effective solution for archiving infrequently accessed data for compliance purposes. What S3 storage class would you choose?

### 15) You need to replicate data between two different S3 buckets in different AWS regions for disaster recovery. How would you set this up?

### 16) You want to inspect all network traffic going in and out of your VPC for security purposes. What AWS service could you use?

### 17) You suspect that an unauthorized user has gained access to your AWS environment. What steps would you take to investigate and remediate this?

### 18) You need to provide secure access to your internal web applications running on EC2 instances to users outside your corporate network without exposing them directly to the internet. How would you achieve this?

### 19) You want to monitor the health and performance of your hybrid cloud environment (both on-premises and AWS resources) from a single pane of glass. What AWS services could you leverage?

### 20) You need to share resources (e.g., VPCs, subnets) between different AWS accounts securely. How you will do that?

### 21) You need to enforce tagging policies across all your AWS accounts for cost tracking and resource management. How can you achieve this?
