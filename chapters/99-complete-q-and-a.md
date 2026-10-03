[Back to notes index](../README.md)

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | Next: End |
| --- | --- | --- |

# 99. Complete questions and answers

These questions review the core AWS concepts in the notes. Use them to explain a design decision in your own words, not only to recall a service name.

## 1. Cloud computing and AWS basics

**1. What does cloud computing change for an engineering team?**

It lets a team request, change, and release infrastructure through provider services instead of owning all physical capacity. The team still designs, secures, monitors, and pays for the workload.

**2. What is the shared responsibility model?**

AWS secures the underlying cloud infrastructure, while the customer secures the workload and configuration in the cloud. The exact boundary depends on the service model.

**3. What does a managed service take over?**

It operates some infrastructure tasks such as host maintenance. The customer still owns the data, permissions, workload configuration, and recovery needs that remain in the service boundary.

**4. What is an AWS account?**

It is an administrative and billing boundary that contains AWS resources. It is separate from an IAM identity, which requests access to those resources.

**5. Why confirm the active caller before changing infrastructure?**

A profile name can point to an unexpected account or role. Checking the caller ARN helps prevent an operation from reaching the wrong environment.

## 2. Global infrastructure and core services

**6. What is a Region?**

A Region is a geographic area where AWS operates services. Most resources are created in a selected Region and do not automatically exist elsewhere.

**7. What is an Availability Zone?**

It is an isolated location within a Region. A subnet belongs to one zone, and a service must be configured across zones to use their separate failure domains.

**8. How does an edge location differ from an Availability Zone?**

An edge location supports services such as content delivery closer to users. It is not the same as placing an application server in another Availability Zone.

**9. What should guide Region selection?**

User latency, data location, service availability, organizational approval, and price should all be checked.

**10. When is a second Region useful?**

It can support disaster recovery or global access when a requirement calls for it. It adds data replication, identity, monitoring, DNS, cost, and testing work.

## 3. Accounts, IAM, and governance

**11. Why should the root user not be used for daily work?**

It has broad authority and is difficult to constrain. Protect it with MFA, avoid root access keys, and use managed identities for normal tasks.

**12. How does an identity policy differ from a resource policy?**

An identity policy is attached to a principal. A resource policy is attached to a resource and can describe which principals may access it.

**13. What does a role trust policy control?**

It defines which principals may assume the role. It does not specify what the role can do after it is assumed.

**14. Do a permissions boundary or service control policy grant access?**

No. They limit the maximum permissions available. An applicable identity or resource policy must still allow the request.

**15. What happens when an explicit deny applies?**

The request is denied even if another applicable policy allows it. Diagnose all relevant policies and conditions before changing permissions.

## 4. AWS CLI, SDKs, and CloudShell

**16. What is a CLI profile?**

It is a named collection of credential-provider and configuration settings, often including an account role, Region, and output format.

**17. Why prefer temporary credentials?**

They expire and reduce the risk of a long-lived secret remaining usable after exposure. Federated access and attached roles can provide them without storing keys in source code.

**18. What is an SDK credential provider chain?**

It is the ordered set of credential sources an SDK checks, such as environment variables, shared configuration, or workload roles.

**19. Does CloudShell make a command safe?**

No. It is a shell environment that can access AWS services. A command can still create, change, or delete resources and incur cost.

**20. What should be checked before a change command?**

Confirm the caller, account, Region, resource target, permissions, expected effect, cost, and cleanup plan.

## 5. VPC networking

**21. What is the difference between a VPC and a subnet?**

A VPC is a regional network boundary with address ranges and routing. A subnet is a smaller range located in one Availability Zone.

**22. What makes a subnet public?**

Its route table sends internet-bound traffic to an Internet Gateway. A resource also needs public addressing and an allowed security group path to be reachable.

**23. How do security groups differ from network ACLs?**

Security groups are stateful and allow rules. Network ACLs are stateless subnet filters with ordered allow and deny rules.

**24. When might a VPC endpoint replace NAT traffic?**

For supported AWS services, a private endpoint may let resources reach the service without using a NAT Gateway. Endpoint type, policy, reachability, and price still need review.

**25. Why use a source security group rule for application-to-database traffic?**

It limits database access to interfaces associated with the application group instead of a broad IP range.

## 6. EC2 and Auto Scaling

**26. What does an AMI provide?**

It provides the operating system and initial software used to launch an EC2 instance.

**27. How should an EC2 workload access AWS APIs?**

Use an instance profile with a scoped IAM role so the workload receives temporary credentials. Do not store access keys on the instance.

**28. How do EBS and instance store differ?**

EBS is persistent block storage that can outlive an instance depending on configuration. Instance store is temporary storage associated with the host.

**29. What does an Auto Scaling group maintain?**

It maintains a desired number of instances across configured subnets and can replace unhealthy capacity or adjust it using scaling policies.

**30. Does stopping an instance end every cost?**

No. Attached EBS volumes, snapshots, addresses, and related services can continue to incur charges.

## 7. S3 storage

**31. What identifies an S3 object?**

A bucket and an object key identify it. A prefix that looks like a directory is part of the key name.

**32. Why keep Block Public Access enabled?**

It helps prevent accidental public exposure. Public access should be an explicit architecture decision with an approved access path.

**33. What does versioning help recover?**

It can preserve earlier object versions after overwrites and can help recover from some accidental changes. Old versions need storage and retention management.

**34. What is a presigned URL?**

It is a time-limited URL that grants a specific operation on an object to whoever holds the link. Treat it as a temporary secret.

**35. What does an S3 lifecycle rule do?**

It transitions or expires objects based on age or prefix. The rule must match retention and recovery requirements before it is enabled.

## 8. Databases with RDS and DynamoDB

**36. When is RDS a natural fit?**

When relational structure, SQL queries, joins, constraints, and transactional behavior are central to the workload.

**37. Why design DynamoDB keys around access patterns?**

The partition and sort keys determine how items can be queried efficiently. They should represent the reads and writes the application needs.

**38. Why can a scan be a poor request path?**

It reads across the table and can consume capacity as data grows. A query that targets a partition key is usually more focused.

**39. How does Multi-AZ differ from a read replica?**

Multi-AZ is an availability deployment with failover behavior. A read replica is primarily a separate read or replication copy and is not a backup.

**40. Why test a database restore?**

A backup is useful only if data can be restored, validated, and brought back within the required recovery time.

## 9. Serverless with Lambda and API Gateway

**41. Why should a Lambda handler be stateless?**

Execution environments may be reused or replaced. Durable user data belongs in a persistent data service, not in process memory.

**42. What does the Lambda execution role control?**

It controls the AWS API permissions used by the function code, such as writing to a log group or reading a specific object.

**43. Why make an event handler idempotent?**

Event delivery can repeat after retries or failures. Idempotency prevents a repeated event from applying the same side effect more than once.

**44. How does API Gateway invoke permission differ from the function role?**

The function role grants the code access to AWS services. A resource-based permission separately allows API Gateway to invoke the function.

**45. What affects Lambda cost and capacity?**

Request volume, duration, memory allocation, concurrency, data transfer, logging, and connected services all matter.

## 10. Containers with ECR, ECS, and Fargate

**46. Why track an image digest?**

A digest identifies exact image content. A tag can be moved to a different image unless the repository prevents overwrites.

**47. How does an ECS task differ from an ECS service?**

A task is a running copy of a task definition. A service maintains the desired number of tasks and manages deployments and replacement.

**48. How do the ECS execution role and task role differ?**

The execution role supports startup tasks such as pulling images and sending logs. The task role is available to the application inside the container.

**49. What does Fargate manage?**

It manages the compute host capacity for supported containers. The customer still defines the task, roles, networking, scaling, and application behavior.

**50. What does a load balancer health check do?**

It checks whether a target can serve traffic and helps avoid routing requests to unhealthy tasks. The endpoint should reflect actual readiness.

## 11. Messaging with SQS, SNS, and EventBridge

**51. How do SQS, SNS, and EventBridge differ?**

SQS buffers work for consumers, SNS fans out messages to subscribers, and EventBridge routes structured events by rules.

**52. What does an SQS visibility timeout do?**

It temporarily hides a received message while the consumer processes it. If it is not deleted before timeout, it can be delivered again.

**53. What is a dead-letter queue for?**

It retains messages that exceed a receive or delivery policy so an operator can investigate and decide whether to redrive them.

**54. What does FIFO ordering guarantee?**

It preserves order within a configured message group. It does not make every downstream side effect exactly once.

**55. How can a consumer avoid duplicate work?**

Use a stable message or event identifier as an idempotency key and record successful processing before applying the same side effect again.

## 12. Observability with CloudWatch, CloudTrail, and Config

**56. How do CloudWatch, CloudTrail, and Config differ?**

CloudWatch covers metrics, logs, and alarms. CloudTrail records AWS API activity. Config records resource configuration and evaluates selected rules.

**57. Are all S3 object operations necessarily in a CloudTrail trail?**

No. Data events often need explicit selection. Check the trail configuration and event volume for the resource.

**58. Why set log retention?**

It controls how long events remain stored, which affects cost, privacy, and operational access.

**59. What makes an alarm actionable?**

It has a meaningful signal and threshold, a clear owner, a notification path, and a response guide.

**60. Why avoid logging secrets?**

Logs are copied, queried, and retained by operational systems. A leaked token or password can grant access to systems beyond the original application.

## 13. Infrastructure as code with CloudFormation and CDK

**61. What is a CloudFormation stack?**

It is a deployed collection of resources managed from a template. References connect resources and their generated identifiers.

**62. What does a change set show?**

It previews expected stack operations such as additions, updates, replacements, and deletions before the change is applied.

**63. What is infrastructure drift?**

It is a difference between the deployed resource configuration and the configuration described by its template.

**64. How does CDK relate to CloudFormation?**

CDK code synthesizes a CloudFormation template. The resulting resources are deployed and managed as CloudFormation stacks.

**65. What can a retain deletion policy cause?**

It can preserve data when a stack is removed, but the resource can remain billable and require separate ownership and cleanup.

## 14. Security, encryption, and secrets

**66. Does encryption replace access control?**

No. Encryption protects data confidentiality, while identity and resource policies decide which requests are allowed.

**67. What is envelope encryption?**

A data key encrypts the content and a KMS key protects the data key. Access requires the relevant key and service permissions.

**68. When might Secrets Manager be preferred to Parameter Store?**

It is designed for secret lifecycle and rotation patterns. Parameter Store can hold configuration and protected SecureString values. Choose based on the required behavior.

**69. Why are KMS key policies important?**

They control use and administration of the key. A restrictive policy can make encrypted data inaccessible even when a user has service-level permissions.

**70. Do GuardDuty or Security Hub remediate every finding automatically?**

No. They provide detection and aggregation capabilities. Teams still need owners, response steps, and verification.

## 15. Reliability, backups, and disaster recovery

**71. What does RPO measure?**

It is the maximum amount of recent data loss the business can tolerate after a recovery event.

**72. What does RTO measure?**

It is the maximum acceptable time to restore a service after an outage.

**73. How do backups differ from replication?**

Replication keeps copies synchronized or near-synchronized, while backups preserve recoverable points in time. Replication can copy corruption or deletion.

**74. Why bound retries and add jitter?**

Bounds prevent endless work, and jitter spreads repeated requests so many clients do not overload a recovering dependency at once.

**75. What is a recovery test expected to prove?**

It should show that data and infrastructure can be restored, the application can use them, and measured recovery meets the objectives.

## 16. Cost management and architecture review

**76. Does an AWS Budget automatically stop spending?**

No. It can notify owners about actual or forecasted cost. It does not automatically impose a hard spending limit.

**77. Why use cost allocation tags?**

They associate resource usage with owners, applications, environments, or cost centers. Tags must be consistently applied and activated for billing reports.

**78. What is right sizing?**

It is choosing service capacity from measured workload demand while retaining the required performance and recovery margin.

**79. What areas does the Well-Architected Framework review?**

Operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability.

**80. What should an architecture decision record include?**

The requirement, selected approach, alternatives considered, trade-offs, and the condition that would make the team revisit the decision.
