[Back to notes index](../README.md)

| [Previous: Cost management and architecture review](16-cost-management-and-architecture-review.md) | [Notes index](../README.md) | [Next: Complete questions and answers](99-complete-q-and-a.md) |
| --- | --- | --- |

# 98. All code samples

This chapter gathers every fenced code sample from the core AWS notes. Each section links back to the chapter that explains the example.

## 1. Cloud computing and AWS basics

[Open chapter](01-cloud-computing-and-aws-basics.md)

### Sample 1

```bash
aws sts get-caller-identity
aws configure list
aws ec2 describe-regions --query 'Regions[].RegionName' --output table
```

## 2. Global infrastructure and core services

[Open chapter](02-global-infrastructure-and-core-services.md)

### Sample 1

```bash
aws ec2 describe-regions --all-regions --query 'Regions[].{Name:RegionName,Status:OptInStatus}' --output table
aws ec2 describe-availability-zones --region ap-south-1 --query 'AvailabilityZones[].{Name:ZoneName,Id:ZoneId,State:State}' --output table
```

## 3. Accounts, IAM, and governance

[Open chapter](03-accounts-iam-and-governance.md)

### Sample 1

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

### Sample 2

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

### Sample 3

```bash
aws sts get-caller-identity
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam::123456789012:role/ExampleReadRole --action-names s3:GetObject --resource-arns arn:aws:s3:::example-study-bucket/example.txt
```

## 4. AWS CLI, SDKs, and CloudShell

[Open chapter](04-aws-cli-sdk-and-cloudshell.md)

### Sample 1

```bash
aws configure sso
aws sso login --profile study-dev
aws sts get-caller-identity --profile study-dev
```

### Sample 2

```bash
aws configure list-profiles
aws configure list --profile study-dev
aws ec2 describe-instances --profile study-dev --region ap-south-1 --no-cli-pager
```

### Sample 3

```bash
AWS_PROFILE=study-dev AWS_REGION=ap-south-1 python inspect_account.py
```

### Sample 4

```python
import boto3

session = boto3.Session(profile_name="study-dev", region_name="ap-south-1")
sts = session.client("sts")
identity = sts.get_caller_identity()

print(identity["Arn"])
```

### Sample 5

```bash
aws s3api list-buckets --query 'Buckets[].Name' --output table
aws ec2 describe-instances --query 'Reservations[].Instances[].{Id:InstanceId,State:State.Name}' --output table
aws help
aws s3api help
```

## 5. VPC networking

[Open chapter](05-vpc-networking.md)

### Sample 1

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Minimal VPC and public subnet for a network study example
Parameters:
  VpcCidr:
    Type: String
    Default: 10.20.0.0/16
  PublicSubnetCidr:
    Type: String
    Default: 10.20.1.0/24
Resources:
  StudyVpc:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidr
      EnableDnsSupport: true
      EnableDnsHostnames: true
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref StudyVpc
      CidrBlock: !Ref PublicSubnetCidr
      MapPublicIpOnLaunch: true
  InternetGateway:
    Type: AWS::EC2::InternetGateway
  GatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref StudyVpc
      InternetGatewayId: !Ref InternetGateway
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref StudyVpc
  DefaultInternetRoute:
    Type: AWS::EC2::Route
    DependsOn: GatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  PublicSubnetAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable
Outputs:
  VpcId:
    Value: !Ref StudyVpc
```

### Sample 2

```bash
aws ec2 describe-vpcs --query 'Vpcs[].{Vpc:VpcId,Cidr:CidrBlock,State:State}' --output table
aws ec2 describe-subnets --query 'Subnets[].{Subnet:SubnetId,Vpc:VpcId,Zone:AvailabilityZone,Cidr:CidrBlock}' --output table
aws ec2 describe-route-tables --query 'RouteTables[].{Table:RouteTableId,Routes:Routes}' --output json
```

## 6. EC2 and Auto Scaling

[Open chapter](06-ec2-and-auto-scaling.md)

### Sample 1

```yaml
Parameters:
  PrivateSubnetIds:
    Type: List<AWS::EC2::Subnet::Id>
  WebLaunchTemplateId:
    Type: String
Resources:
  WebFleet:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: '2'
      MaxSize: '4'
      DesiredCapacity: '2'
      VPCZoneIdentifier: !Ref PrivateSubnetIds
      LaunchTemplate:
        LaunchTemplateId: !Ref WebLaunchTemplateId
        Version: '$Latest'
      HealthCheckType: ELB
      HealthCheckGracePeriod: 120
```

### Sample 2

```bash
aws ec2 describe-instance-types --instance-types t3.small --query 'InstanceTypes[].{Type:InstanceType,VCPU:VCpuInfo.DefaultVCpus,Memory:MemoryInfo.SizeInMiB}' --output table
aws ec2 describe-instances --query 'Reservations[].Instances[].{Id:InstanceId,Type:InstanceType,State:State.Name,Zone:Placement.AvailabilityZone}' --output table
aws autoscaling describe-auto-scaling-groups --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Desired:DesiredCapacity,Min:MinSize,Max:MaxSize}' --output table
```

## 7. S3 storage

[Open chapter](07-s3-storage.md)

### Sample 1

```python
import boto3

s3 = boto3.client("s3", region_name="ap-south-1")
s3.upload_file(
    Filename="report.csv",
    Bucket="example-study-bucket",
    Key="exports/report.csv",
)
```

### Sample 2

```python
url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": "example-study-bucket", "Key": "exports/report.csv"},
    ExpiresIn=300,
)
print(url)
```

### Sample 3

```json
{
  "Rules": [
    {
      "ID": "ExpireTemporaryExports",
      "Status": "Enabled",
      "Filter": { "Prefix": "temporary/" },
      "Expiration": { "Days": 30 }
    }
  ]
}
```

### Sample 4

```bash
aws s3api get-public-access-block --bucket example-study-bucket
aws s3api get-bucket-versioning --bucket example-study-bucket
aws s3api get-bucket-encryption --bucket example-study-bucket
aws s3api head-object --bucket example-study-bucket --key exports/report.csv
```

## 8. Databases with RDS and DynamoDB

[Open chapter](08-databases-rds-and-dynamodb.md)

### Sample 1

```sql
CREATE TABLE notes (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    owner_id BIGINT NOT NULL,
    title VARCHAR(160) NOT NULL,
    body TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX notes_owner_created_idx
    ON notes (owner_id, created_at DESC);
```

### Sample 2

```python
cursor.execute(
    "SELECT id, title FROM notes WHERE owner_id = %s ORDER BY created_at DESC",
    (owner_id,),
)
```

### Sample 3

```python
from boto3.dynamodb.conditions import Key

response = table.query(
    KeyConditionExpression=Key("PK").eq("USER#42")
    & Key("SK").begins_with("NOTE#")
)
notes = response["Items"]
```

### Sample 4

```bash
aws rds describe-db-instances --query 'DBInstances[].{Id:DBInstanceIdentifier,Engine:Engine,Status:DBInstanceStatus,Public:PubliclyAccessible}' --output table
aws dynamodb describe-table --table-name StudyNotes --query 'Table.{Name:TableName,Status:TableStatus,Keys:KeySchema}' --output json
```

## 9. Serverless with Lambda and API Gateway

[Open chapter](09-serverless-lambda-and-api-gateway.md)

### Sample 1

```python
import json


def handler(event, context):
    raw_body = event.get("body") or "{}"

    try:
        body = json.loads(raw_body)
    except json.JSONDecodeError:
        return {
            "statusCode": 400,
            "headers": {"content-type": "application/json"},
            "body": json.dumps({"error": "Request body must be valid JSON"}),
        }

    title = body.get("title")
    if not isinstance(title, str) or not title.strip():
        return {
            "statusCode": 400,
            "headers": {"content-type": "application/json"},
            "body": json.dumps({"error": "A title is required"}),
        }

    return {
        "statusCode": 201,
        "headers": {"content-type": "application/json"},
        "body": json.dumps({"title": title.strip()}),
    }
```

## 10. Containers with ECR, ECS, and Fargate

[Open chapter](10-containers-ecr-ecs-and-fargate.md)

### Sample 1

```bash
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com
docker build -t reading-api:1.0.0 .
docker tag reading-api:1.0.0 123456789012.dkr.ecr.ap-south-1.amazonaws.com/reading-api:1.0.0
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/reading-api:1.0.0
```

### Sample 2

```json
{
  "family": "reading-api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ReadingApiExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ReadingApiTaskRole",
  "containerDefinitions": [
    {
      "name": "api",
      "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/reading-api:1.0.0",
      "essential": true,
      "portMappings": [
        { "containerPort": 8080, "protocol": "tcp" }
      ],
      "environment": [
        { "name": "PORT", "value": "8080" }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/reading-api",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "container"
        }
      }
    }
  ]
}
```

## 11. Messaging with SQS, SNS, and EventBridge

[Open chapter](11-messaging-sqs-sns-and-eventbridge.md)

### Sample 1

```python
def handler(event, context):
    failures = []

    for record in event["Records"]:
        try:
            process_message(record["body"])
        except ExpectedTemporaryError:
            failures.append({"itemIdentifier": record["messageId"]})

    return {"batchItemFailures": failures}
```

## 12. Observability with CloudWatch, CloudTrail, and Config

[Open chapter](12-observability-cloudwatch-cloudtrail-and-config.md)

### Sample 1

```text
fields @timestamp, @message
| filter level = "ERROR"
| sort @timestamp desc
| limit 20
```

### Sample 2

```yaml
Resources:
  InstanceCpuHigh:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: Average CPU is above the study threshold
      Namespace: AWS/EC2
      MetricName: CPUUtilization
      Dimensions:
        - Name: InstanceId
          Value: i-0123456789abcdef0
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 80
      ComparisonOperator: GreaterThanThreshold
      TreatMissingData: notBreaching
```

### Sample 3

```bash
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=RunInstances --max-results 10
aws configservice describe-compliance-by-config-rule --output table
aws cloudwatch describe-alarms --state-value ALARM --output table
```

## 13. Infrastructure as code with CloudFormation and CDK

[Open chapter](13-infrastructure-as-code-cloudformation-and-cdk.md)

### Sample 1

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

### Sample 2

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

### Sample 3

```bash
aws cloudformation validate-template --template-body file://storage.yaml
aws cloudformation describe-stacks --stack-name StudyStorage --region ap-south-1
cdk synth
cdk diff
```

## 14. Security, encryption, and secrets

[Open chapter](14-security-encryption-and-secrets.md)

### Sample 1

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

### Sample 2

```python
import json
import boto3

secrets = boto3.client("secretsmanager", region_name="ap-south-1")
response = secrets.get_secret_value(SecretId="reading-api/db")
database_settings = json.loads(response["SecretString"])
```

## 15. Reliability, backups, and disaster recovery

[Open chapter](15-reliability-backups-and-disaster-recovery.md)

### Sample 1

```python
import boto3
from botocore.config import Config

retry_config = Config(
    retries={
        "mode": "standard",
        "max_attempts": 5,
    }
)

s3 = boto3.client("s3", config=retry_config)
```

## 16. Cost management and architecture review

[Open chapter](16-cost-management-and-architecture-review.md)

### Sample 1

```bash
aws ce get-cost-and-usage --time-period Start=2026-09-01,End=2026-10-01 --granularity MONTHLY --metrics UnblendedCost --group-by Type=DIMENSION,Key=SERVICE --output json
```
