[Back to notes index](../README.md)

| [Previous: Accounts, IAM, and governance](03-accounts-iam-and-governance.md) | [Notes index](../README.md) | [Next: VPC networking](05-vpc-networking.md) |
| --- | --- | --- |
# 4. AWS CLI, SDKs, and CloudShell

The AWS Command Line Interface (CLI) and software development kits (SDKs) call AWS service APIs. They use credentials, a Region, and a request to perform an action. The same permissions apply whether a request comes from the console, the CLI, or application code.

## Install and sign in

Use AWS CLI version 2 from the official installation instructions or use AWS CloudShell where it is available. For organization accounts, configure IAM Identity Center access with the CLI setup wizard. Do not use root credentials or commit access keys to a repository.

The SSO configuration associates a local profile with an account and role. The login command obtains short-lived credentials for that profile. Those credentials expire and can be refreshed through the sign-in flow.

```bash
aws configure sso
aws sso login --profile study-dev
aws sts get-caller-identity --profile study-dev
```

Before a command that changes infrastructure, confirm the account and role from the caller ARN. A familiar profile name is not proof that it points to the intended account.

## Profiles, Regions, and output

A profile is a named set of CLI settings and credential-provider configuration. Keep separate profiles for separate environments rather than editing one profile back and forth. The Region can be set in a profile or supplied on a command. An explicit command option is easy to see during review.

```bash
aws configure list-profiles
aws configure list --profile study-dev
aws ec2 describe-instances --profile study-dev --region ap-south-1 --no-cli-pager
```

The shared configuration file usually lives at ~/.aws/config. The credentials file may contain long-lived keys if they were configured. Avoid long-lived keys where federation or an attached role is available. Protect local files and do not copy them into a project.

For scripts, set a profile and Region explicitly or use a controlled execution environment. The following is a shell example for a one-command profile selection:

```bash
AWS_PROFILE=study-dev AWS_REGION=ap-south-1 python inspect_account.py
```

On Windows PowerShell, environment variables use a different syntax. Set them only for the current process when needed, and clear them after the task.

## The SDK credential provider chain

An SDK searches supported credential sources in a defined order. Sources can include environment variables, shared configuration, web identity, container credentials, or an instance profile. The exact chain depends on the SDK and runtime. For AWS-hosted workloads, use the attached role so the SDK can obtain rotating temporary credentials automatically.

Do not put a key and secret in source code. A small Python example can create a client without passing credentials. Boto3 resolves them from the configured provider chain.

```python
import boto3

session = boto3.Session(profile_name="study-dev", region_name="ap-south-1")
sts = session.client("sts")
identity = sts.get_caller_identity()

print(identity["Arn"])
```

This code requires the boto3 package and a configured profile. It only requests caller identity. It does not print the account ID or any credential secret.

## CloudShell

CloudShell is a browser-based shell associated with the signed-in AWS console session. It can provide a ready-to-use CLI environment and selected console credentials. CloudShell is useful for short administration tasks, but files in its environment should not be treated as the only copy of important work. Check Region, permissions, and session identity before use.

A CloudShell session can have persistent storage and may incur service-specific costs if it creates AWS resources. The shell itself does not make a command safe. Review the command and its target before running it.

## Read and change operations

Commands such as describe, get, and list commonly read service state. Commands such as create, delete, put, update, and terminate can change or remove resources. AWS CLI options are service-specific, so a dry-run flag is not available for every command.

Use these habits for change operations:

1. Confirm the active identity and Region.
2. Read the command help and identify whether it creates cost or changes access.
3. Use a non-production account for practice.
4. Review the exact resource name and policy before execution.
5. Capture the result and verify the intended state.
6. Remove temporary resources and check billing afterwards.

Use pagination deliberately. Some commands return a page of results, and the CLI may automatically fetch additional pages. Avoid disabling pagination unless you understand how the output is being limited.

## Useful CLI patterns

The query option selects fields from a response. Choose JSON for scripts that need structured output and a table for quick inspection. Avoid storing large or sensitive output in terminal logs.

```bash
aws s3api list-buckets --query 'Buckets[].Name' --output table
aws ec2 describe-instances --query 'Reservations[].Instances[].{Id:InstanceId,State:State.Name}' --output table
aws help
aws s3api help
```

## Key points

- Use the official CLI v2 installation and an organization-approved sign-in method.
- Profiles keep environment settings separate. Confirm the caller identity before a change.
- SDKs should resolve credentials from a provider chain, not from source-code literals.
- Attached roles provide temporary credentials to AWS-hosted workloads.
- Treat CloudShell as a command-line environment, not as a safety boundary.

## Practice

1. Configure a named profile through IAM Identity Center and confirm its caller ARN.
2. Use a read-only command to inspect a service in an explicit Region.
3. Explain which credential source a local SDK session will use and how a deployed workload will authenticate.
4. Review a command with a create or delete operation. Identify its account, Region, resource, and possible cost before running it in a sandbox.

## References

- [AWS CLI getting started](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
- [AWS CLI SSO configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
- [AWS CLI configuration and credential files](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
- [Boto3 credentials](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html)
- [AWS CloudShell User Guide](https://docs.aws.amazon.com/cloudshell/latest/userguide/welcome.html)
