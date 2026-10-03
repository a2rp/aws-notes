[Back to notes index](../README.md)

| [Previous: VPC networking](05-vpc-networking.md) | [Notes index](../README.md) | [Next: S3 storage](07-s3-storage.md) |
| --- | --- | --- |
# 6. EC2 and Auto Scaling

Amazon Elastic Compute Cloud (EC2) provides virtual machine instances. You choose an image, instance type, network placement, storage, and access controls. AWS operates the underlying hardware, while the customer manages the operating system and application on the instance.

## What defines an instance

An instance launch combines several decisions:

- An Amazon Machine Image (AMI) provides the operating system and initial software.
- An instance type selects CPU, memory, network, and accelerator capacity.
- A subnet and security group define network placement and allowed connections.
- An IAM instance profile gives the workload temporary permissions through a role.
- EBS volumes provide persistent block storage, while instance store disks are temporary host storage.
- A user data script can run setup commands during first boot.

Use an AMI from a trusted publisher and keep operating system packages patched. Prefer Systems Manager Session Manager for managed administrative access when the prerequisites are met. This can avoid opening inbound SSH to the internet. If SSH is needed, restrict its source and use controlled key handling.

## Instance and storage choices

Select an instance family based on the workload. General purpose instances balance CPU and memory. Compute optimized instances suit CPU-bound work. Memory optimized instances suit workloads that hold large data sets in memory. Benchmark the real application before selecting a final size.

EBS volumes persist independently of an instance lifecycle when configured to do so. They can be backed up with snapshots, which are stored incrementally. Instance store is tied to the host and should be treated as temporary. The application must tolerate loss of instance store data.

An instance can have a public address, but that does not mean it should. Prefer private application instances behind a load balancer. Keep databases and management interfaces off public paths unless an explicit requirement and controls justify exposure.

## Launch templates and Auto Scaling

A launch template records instance launch settings for repeatable deployments. An Auto Scaling group maintains a desired number of instances across selected subnets. It can replace unhealthy instances and adjust capacity according to a scaling policy.

Auto Scaling helps manage compute capacity, but it does not make an application stateless. Store durable user data in a managed database or object store. Put shared web assets in a suitable shared service. Use a load balancer health check that reflects whether a target can serve traffic.

A target tracking policy adjusts capacity to keep a metric near a target, such as average CPU use or request count per target. Set minimum, maximum, and desired capacity deliberately. A maximum protects cost but can also limit the workload during a traffic spike.

## Example launch configuration

This CloudFormation fragment describes an Auto Scaling group using a supplied launch template and private subnets. It can create billable EC2 instances if deployed. The security group, launch template, subnets, and permissions are expected to be provided by the surrounding stack.

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
      HealthCheckType: EC2
      HealthCheckGracePeriod: 120
```

This fragment uses EC2 health checks because it does not attach a load balancer. In a complete service, attach a target group and use load balancer health checks when application readiness should control replacement. Review the AMI, instance profile, network rules, and maximum capacity before deployment.

## Inspect capacity without launching it

The following commands inspect existing instance types and instances. They do not launch new capacity.

```bash
aws ec2 describe-instance-types --instance-types t3.small --query 'InstanceTypes[].{Type:InstanceType,VCPU:VCpuInfo.DefaultVCpus,Memory:MemoryInfo.SizeInMiB}' --output table
aws ec2 describe-instances --query 'Reservations[].Instances[].{Id:InstanceId,Type:InstanceType,State:State.Name,Zone:Placement.AvailabilityZone}' --output table
aws autoscaling describe-auto-scaling-groups --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Desired:DesiredCapacity,Min:MinSize,Max:MaxSize}' --output table
```

## Safe operations

EC2 resources can continue to incur charges while running, and attached storage or snapshots can continue to incur charges after an instance stops. Review the billing model for the selected Region and purchase option. Set a budget alert before a long experiment.

Before a change, verify the account, Region, instance ID, and expected effect. A stop preserves an EBS-backed instance for later use, while termination removes the instance and may delete volumes marked for deletion on termination. Verify attached volumes and backup requirements first.

## Key points

- AMI, instance type, network, role, and storage together define the launch.
- Use an instance role for AWS API access instead of access keys on disk.
- Keep management access private where possible and patch the operating system.
- Auto Scaling replaces or adjusts compute capacity; it does not store application state for you.
- Stopped instances and snapshots may still cost money. Check the billing behavior of every resource.

## Practice

1. Choose an instance family for a CPU-bound job and explain which metrics you would benchmark.
2. Draw a private Auto Scaling group behind a public load balancer across two Availability Zones.
3. Explain how an instance role differs from an SSH key pair.
4. Before launching a test instance, list the Region, size, disk, duration, budget, and cleanup checks you would make.

## References

- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [EC2 instance types](https://docs.aws.amazon.com/ec2/latest/instancetypes/instance-types.html)
- [Auto Scaling groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)
- [Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
