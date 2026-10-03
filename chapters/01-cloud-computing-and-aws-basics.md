[Back to notes index](../README.md)

| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: Global infrastructure and core services](02-global-infrastructure-and-core-services.md) |
| --- | --- | --- |
# 1. Cloud computing and AWS basics

These notes introduce the cloud model and the responsibilities that come with using AWS. The goal is to understand the vocabulary before choosing a service or creating resources.

## What cloud computing means

Cloud computing provides computing resources through a network on demand. Instead of buying and maintaining every server, a team can request capacity from a provider, use it, and change or release it as needs change. AWS offers resources through a web console, command-line tools, SDKs, and service APIs.

The cloud changes how capacity is obtained. It does not remove the need to design systems, protect data, monitor failures, or understand costs. A managed service can reduce maintenance work, but the application still needs correct access rules, safe configuration, and a recovery plan.

Common benefits include:

- Provisioning infrastructure in minutes rather than waiting for hardware.
- Increasing or reducing capacity as demand changes.
- Using managed services for routine infrastructure tasks.
- Deploying resources in more than one fault domain.
- Paying for measured usage instead of owning all possible peak capacity.

These benefits depend on good design. Unused instances, oversized databases, unnecessary data transfer, and forgotten storage still cost money.

## Service models

The service model describes how much of the stack the provider operates.

| Model | AWS example | What the customer still manages |
| --- | --- | --- |
| Infrastructure as a Service | Amazon EC2 virtual machines | Guest operating system, patches, application, and data |
| Managed platform or service | Amazon RDS database | Data, access, schema, and workload configuration |
| Serverless service | AWS Lambda | Function code, event handling, permissions, and data |

The labels are useful, but real services sit on a spectrum. Amazon RDS manages database hosts and much of the maintenance. The customer still chooses an engine and configuration, controls who can connect, protects credentials, and plans backups.

## Shared responsibility

AWS is responsible for security of the cloud, including the physical facilities, hardware, and underlying service infrastructure. The customer is responsible for security in the cloud. The exact customer responsibilities depend on the service selected.

For an EC2 instance, the customer manages the guest operating system and application. For a managed database, AWS handles more of the host maintenance, while the customer still owns database access and data protection. With every service, the customer must understand the service boundary and configure the controls that remain theirs.

A practical review asks:

1. Which parts of the service does AWS operate?
2. Which identities can change or read this resource?
3. Where does the data travel and where is it stored?
4. What logs show access and configuration changes?
5. How would the workload recover after a failure?

## An AWS account and its resources

An AWS account is an administrative and billing boundary. It contains resources such as buckets, virtual machines, functions, and databases. An account is not the same thing as an IAM identity. IAM users, roles, and federated identities request access to resources through policies.

A Region is a geographic area where AWS operates services. Availability Zones are separate locations within a Region. Many workloads use more than one Availability Zone so a local infrastructure problem does not take down every copy. The next chapter describes this layout in detail.

A resource usually has an ARN, or Amazon Resource Name, that identifies it in policies and API calls. Names that look similar do not imply that resources share permissions or data.

## Inspect an account without creating resources

After configuring the AWS CLI with an approved sign-in method, use read-only commands to confirm which identity and Region are active. These commands do not create AWS resources.

```bash
aws sts get-caller-identity
aws configure list
aws ec2 describe-regions --query 'Regions[].RegionName' --output table
```

The first command returns the account ID and the caller ARN. Check these before running a command that changes infrastructure. The second shows which credential and Region settings the CLI resolved. The third lists Regions that the current identity can see.

Do not paste access keys, session tokens, account-specific secrets, or private identifiers into notes or public repositories. Prefer an organization-approved identity flow and short-lived credentials.

## A first design exercise

Consider a small website with a static front end, an API, and user records. Write down which AWS service category might serve each need, without creating anything:

- Static files need durable object storage and a delivery path.
- The API needs compute that can receive requests and scale within its limits.
- User records need a data model chosen for the access patterns.
- Each component needs an identity with only the permissions it requires.
- Logs and metrics need to make errors and unusual traffic visible.
- The design needs a budget and an owner who can remove test resources.

The important first step is to describe the workload and constraints. Service names come after the requirements.

## Key points

- Cloud computing changes how infrastructure is provisioned and billed, but does not remove engineering responsibility.
- AWS and the customer share security responsibilities. The boundary changes by service.
- An AWS account is an administrative boundary. IAM identities and policies determine access.
- Confirm the active AWS identity and Region before changing resources.
- Check cost, access, data handling, and cleanup before creating an experiment.

## Practice

1. Explain the difference between a Region, an Availability Zone, and an AWS account.
2. Compare what you manage on EC2 with what you manage when using RDS.
3. Run the three read-only CLI commands after configuring access. Record which profile and Region are active, but do not publish the account ID.
4. Sketch the website example as components and list one security concern for each component.

## References

- [AWS Cloud overview](https://aws.amazon.com/what-is-aws/)
- [AWS shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [Getting started with the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
