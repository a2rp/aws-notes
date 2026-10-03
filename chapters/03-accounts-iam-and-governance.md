[Back to notes index](../README.md)

| [Previous: Global infrastructure and core services](02-global-infrastructure-and-core-services.md) | [Notes index](../README.md) | [Next: AWS CLI, SDKs, and CloudShell](04-aws-cli-sdk-and-cloudshell.md) |
| --- | --- | --- |
# 3. Accounts, IAM, and governance

AWS Identity and Access Management (IAM) controls which principals can make requests and which actions those requests may perform. Good access design starts with identities, resource ownership, and the smallest permission set that allows the work to succeed.

## Protect the account root user

The root user is created with the AWS account and has broad authority. Do not use it for daily work. Protect it with a strong unique password and multi-factor authentication. Keep recovery details current and use it only for tasks that specifically require root access. Do not create root access keys.

For human access, use the organization identity provider and AWS IAM Identity Center where available. This provides centrally managed sign-in and temporary role credentials. A workload should use an IAM role rather than a person's credentials or a long-lived access key.

## Principals and policies

A principal is an identity making an AWS request. Principals include IAM roles, IAM users, the account root user, and federated identities. A policy is a JSON document that describes allowed or denied actions and the resources and conditions where those rules apply.

An identity-based policy is attached to an identity. A resource-based policy is attached to a resource such as an S3 bucket. A role trust policy decides which principal may assume the role. A permissions boundary limits the maximum permissions an identity policy can grant, but does not grant access by itself.

An AWS Organizations service control policy sets the maximum available permissions for accounts in an organization. It does not grant permissions. A request must still be allowed by an applicable identity or resource policy, and it must not be denied by a higher-level control.

## How policy evaluation works

The practical default is deny. A request succeeds only when the applicable policies allow it and no applicable explicit deny blocks it. More than one policy can participate in a request, including identity policies, resource policies, permissions boundaries, session policies, and organization policies.

When access fails, check the complete request context:

1. Which principal is making the request?
2. Which action and resource are involved?
3. Is the resource in a different account or Region?
4. Which identity and resource policies apply?
5. Does a condition key, boundary, session policy, or organization policy restrict it?
6. Is there an explicit deny or a missing service role trust relationship?

Do not fix an access error by attaching administrator access without understanding the missing permission.

## Least-privilege example

This identity policy allows listing one bucket and reading objects in that bucket. Replace the example bucket name with the resource the workload owns. It does not allow writes or access to other buckets.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListOneBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::example-study-bucket"
    },
    {
      "Sid": "ReadObjectsInOneBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-study-bucket/*"
    }
  ]
}
```

The bucket and object ARNs are different resource types. Actions should be matched to the resource format expected by the service. Add conditions when they express a real boundary, and test the policy before relying on it.

## Let a workload assume a role

An EC2 instance, Lambda function, or other AWS service should receive temporary credentials through an attached role. The role trust policy names the service that may assume it. A separate permissions policy describes what the role can do.

A Lambda execution role uses a trust relationship like this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "lambda.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

This trust document only allows Lambda to assume the role. It does not grant access to S3, logs, databases, or any other service. Add a separate, scoped permission policy for each required task.

## Organize accounts and environments

AWS Organizations can group accounts under organizational units and apply organization-wide controls. Separate accounts can isolate production, development, security logging, and shared infrastructure. Account boundaries help limit the impact of mistakes and separate billing and permissions.

Use tags and naming conventions to record ownership, environment, application, and cost center. Tags are not an access control unless policies explicitly use them. Keep an inventory of privileged roles and review who can assume them.

## Inspect and test access

Use the CLI to confirm the active caller and to inspect a policy before a change. Policy simulation can help test a principal, action, and resource, but it does not replace a real integration test because not every service-specific condition is represented in the same way.

```bash
aws sts get-caller-identity
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam::123456789012:role/ExampleReadRole --action-names s3:GetObject --resource-arns arn:aws:s3:::example-study-bucket/example.txt
```

Do not use an account ID or resource name from a real production environment in public notes. Use clearly fictional examples.

## Key points

- Do not use the root user for everyday work or create root access keys.
- Prefer federated human sign-in and temporary credentials over long-lived access keys.
- A role trust policy controls who can assume a role. Permission policies control what it can do.
- An organization policy or permissions boundary limits permissions but does not grant them.
- Explicit deny overrides an allow. Diagnose the principal, action, resource, and conditions before changing access.

## Practice

1. Draw a policy boundary for a developer who can deploy only to a development account.
2. Review the S3 policy and explain why it has two different resource ARNs.
3. Create a fictional trust policy for one AWS service, then write a separate permission policy for a single task.
4. Explain the difference between a resource policy, a role trust policy, and an identity permission policy.

## References

- [IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [AWS Organizations service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [IAM roles for AWS services](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
