[Back to notes index](../README.md)

| [Previous: Infrastructure as code with CloudFormation and CDK](13-infrastructure-as-code-cloudformation-and-cdk.md) | [Notes index](../README.md) | [Next: Reliability, backups, and disaster recovery](15-reliability-backups-and-disaster-recovery.md) |
| --- | --- | --- |
# 14. Security, encryption, and secrets

Security is a set of controls across identity, network, application, data, and operations. No single AWS service makes a workload secure. Start with the data and threat model, then layer preventive, detective, and recovery controls.

## Identity and access

Use short-lived identities and least privilege. Separate human access from workload access. A workload should use a role associated with its compute service. Review trust policies and permission policies independently. Remove unused credentials and roles, and investigate unexpected access paths.

Use IAM Access Analyzer to identify supported resources shared with external principals and to review policies. Use account-level controls and organizational policies to establish boundaries. A deny guardrail should be tested against the actual workload so it does not block required AWS service operations.

## Encryption with KMS

AWS Key Management Service (KMS) manages cryptographic keys used by AWS services and applications. Many services provide encryption at rest by default with service-managed keys. Customer-managed KMS keys provide more control over key policy, rotation settings, grants, and audit events, but require additional ownership and cost consideration.

Envelope encryption protects data by using a data key to encrypt the content and a KMS key to protect the data key. The application or service must be authorized to use the relevant key. Access to an encrypted object may therefore require both service-level permissions and permission to use the encryption key.

Key policies are a primary control for KMS keys. A key can become unusable if its policy or grants remove required access. Plan key administrators separately from key users. Understand what happens to stored data before scheduling a key deletion.

Encryption in transit uses TLS to protect data while it moves between clients and services. Validate certificates and prefer current protocol configurations. Do not confuse encryption with authorization: an encrypted request can still be made by an identity that should not have access.

## Secrets management

Secrets include database passwords, API tokens, signing keys, and private certificates. Do not commit them to source control, bake them into container images, write them to logs, or pass them through unprotected deployment files.

AWS Secrets Manager is designed for secret lifecycle and supports rotation patterns. AWS Systems Manager Parameter Store can hold configuration and SecureString values protected with KMS. Choose based on the required rotation, access, change history, and application retrieval behavior.

Grant each workload access to only the specific secret it needs. Cache retrieved values where appropriate to reduce request volume, but plan how rotation and revocation take effect. Treat any cached secret as sensitive data.

## A narrow secret-read policy

This example allows a role to read one named secret and use one customer-managed key. Replace the fictional ARNs with exact resources. The key policy must also permit the intended use, and the application should not log the returned value.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOneApplicationSecret",
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:reading-api/db-abc123"
    },
    {
      "Sid": "DecryptWithApplicationKey",
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "arn:aws:kms:ap-south-1:123456789012:key/11111111-2222-3333-4444-555555555555"
    }
  ]
}
```

The permissions shown are only one side of authorization. The service resource policy, key policy, role trust, and organization controls can further restrict the request.

## Retrieve a secret without logging it

The application uses its role and the SDK provider chain. The example parses a JSON secret but does not print it. In a production application, use the supported caching approach for the runtime and keep the secret out of exception messages.

```python
import json
import boto3

secrets = boto3.client("secretsmanager", region_name="ap-south-1")
response = secrets.get_secret_value(SecretId="reading-api/db")
database_settings = json.loads(response["SecretString"])
```

Do not use a public notes repository to store real secret values. Example secret identifiers above are fictional.

## Detect and respond

CloudTrail records API activity. GuardDuty analyzes supported data sources for suspicious activity. Security Hub can help centralize security findings. AWS Config can identify resources that do not match selected rules. These services provide signals, but a team needs ownership, response steps, and a way to verify remediation.

Protect log and backup destinations from unauthorized changes. Restrict public access to object storage. For private service-to-service traffic, use security groups and private endpoints where appropriate. Review cross-account access as carefully as in-account access.

## Security review checklist

- Is the caller authenticated with a managed identity and temporary credentials?
- Does the role have only the actions and resources required by the workload?
- Are data stores private and encrypted with suitable key ownership?
- Are secrets retrieved at runtime and excluded from source, images, and logs?
- Are API, object, and network access paths monitored?
- Can the team detect, investigate, and recover from a security event?

## Key points

- Encryption protects data confidentiality but does not replace access control.
- KMS key policy, IAM policy, grants, and service permissions can all affect access.
- Secrets need restricted retrieval, rotation planning, and careful logging.
- Detective services produce findings that need an owner and response process.
- Protect backups and audit logs from the same identities that can change production data.

## Practice

1. Draw the permission path for a Lambda role to read a database secret encrypted with a customer-managed key.
2. Compare when you would use Secrets Manager with when you would use Parameter Store SecureString.
3. Identify how a public storage resource, a leaked access key, and an unpatched instance would each be detected.
4. Review a workload and list one preventive, one detective, and one recovery control for its sensitive data.

## References

- [AWS KMS concepts](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html)
- [Secrets Manager best practices](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html)
- [Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
- [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
