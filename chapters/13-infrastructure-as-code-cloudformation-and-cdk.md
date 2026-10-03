[Back to notes index](../README.md)

| [Previous: Observability with CloudWatch, CloudTrail, and Config](12-observability-cloudwatch-cloudtrail-and-config.md) | [Notes index](../README.md) | [Next: Security, encryption, and secrets](14-security-encryption-and-secrets.md) |
| --- | --- | --- |
# 13. Infrastructure as code with CloudFormation and CDK

Infrastructure as code (IaC) describes cloud resources in version-controlled files. A reviewed template can be deployed consistently, compared before changes, and used to explain how a system is assembled. IaC does not make a change automatically safe; the plan, permissions, and resource lifecycle still need review.

## CloudFormation templates and stacks

CloudFormation templates describe resources and their relationships. A stack is a deployed collection of resources managed from a template. Parameters provide values at deployment time, outputs expose selected values, and intrinsic functions connect resources without hard-coding generated identifiers.

A template should define only the resources it owns. Use parameters for environment-specific values and outputs for useful references. Avoid placing passwords, access keys, or secret values directly in a template or parameter file. Use a managed secret store and a supported dynamic reference where appropriate.

Resource dependencies can be inferred from references. Explicit dependencies should be used only when an ordering relationship is not visible from a reference. Keep templates small enough to review, and separate resources when their lifecycle or ownership differs.

## Review before deployment

A change set shows the operations CloudFormation expects to perform. Review replacements and deletions carefully, especially for databases, buckets, encryption keys, and network resources. A resource replacement can interrupt service or remove access even when a template update looks small.

Drift detection compares a deployed stack with its template to identify supported out-of-band changes. Drift detection is useful, but it does not prove every service property is tracked or that the deployed application works.

Use deletion and update-replacement policies for resources that contain data when the recovery requirement calls for it. A retained resource can remain billable and may need manual ownership after stack removal. Retention avoids deletion but can leave orphaned resources, so include a cleanup plan.

## CloudFormation example

This template creates a private S3 bucket with versioning and server-side encryption. It does not assign a public bucket policy. The retention settings protect the bucket from stack deletion or replacement, which means it can remain after the stack is removed and continue to store billable data.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Private versioned bucket for a study workload
Resources:
  StudyBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain
    Properties:
      VersioningConfiguration:
        Status: Enabled
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        IgnorePublicAcls: true
        BlockPublicPolicy: true
        RestrictPublicBuckets: true
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
Outputs:
  BucketName:
    Value: !Ref StudyBucket
```

## AWS CDK

The AWS Cloud Development Kit (CDK) lets a team describe infrastructure using a programming language and constructs. The CDK synthesizes CloudFormation templates. The deployed resources are still managed through CloudFormation stacks.

A small CDK v2 example creates a retained bucket with public access blocked and versioning enabled:

```typescript
import { RemovalPolicy, Stack, StackProps } from "aws-cdk-lib";
import { Construct } from "constructs";
import * as s3 from "aws-cdk-lib/aws-s3";

export class StorageStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, "StudyBucket", {
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      encryption: s3.BucketEncryption.S3_MANAGED,
      versioned: true,
      removalPolicy: RemovalPolicy.RETAIN,
    });
  }
}
```

CDK applications still need dependency management, environment configuration, and review. CDK bootstrap may create supporting resources and roles in an account and Region. Check the target account, Region, synthesized template, and diff before deployment.

## Useful commands

Template validation checks basic template structure. It does not prove that the deployment will succeed or that the permissions are safe.

```bash
aws cloudformation validate-template --template-body file://storage.yaml
aws cloudformation describe-stacks --stack-name StudyStorage --region ap-south-1
cdk synth
cdk diff
```

A deploy command changes infrastructure. Use a sandbox first, review the generated change set, and verify the resulting resources. Keep the template and application code versioned together when they are part of the same release.

## State, ownership, and secrets

CloudFormation records stack state, but application data and external resources can have separate lifecycles. Document which stack owns each resource. Avoid making manual console edits to stack-owned resources because they can create drift.

Do not commit secret values in templates, context files, environment files, or synthesized output. Restrict access to deployment roles and the artifact bucket. Separate preview permissions from deployment permissions where possible.

## Key points

- IaC describes desired resources and their relationships in reviewable files.
- A stack is a managed collection of resources. References connect generated identifiers.
- Change sets and diffs help review changes, but do not replace operational checks.
- Retention policies can protect data and also leave billable orphaned resources.
- CDK synthesizes CloudFormation. Review the synthesized template and target environment.

## Practice

1. Add a parameter for the Region-specific resource setting in the S3 example and explain which values should remain fixed.
2. Identify which changes in a hypothetical database update could replace a resource.
3. Run template validation and a CDK diff in a sandbox project without deploying resources.
4. Write down the owner and cleanup process for a retained bucket after its stack is deleted.

## References

- [CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [CloudFormation change sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
- [CloudFormation drift detection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
- [AWS CDK Developer Guide](https://docs.aws.amazon.com/cdk/v2/guide/home.html)
- [CloudFormation resource deletion policy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html)
