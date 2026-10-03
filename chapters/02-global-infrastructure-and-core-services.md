[Back to notes index](../README.md)

| [Previous: Cloud computing and AWS basics](01-cloud-computing-and-aws-basics.md) | [Notes index](../README.md) | [Next: Accounts, IAM, and governance](03-accounts-iam-and-governance.md) |
| --- | --- | --- |
# 2. Global infrastructure and core services

AWS runs services in a global infrastructure. Choosing where a workload runs affects latency, availability, data location, service availability, and cost. A Region is therefore an architecture decision, not just a dropdown in the console.

## Regions

A Region is a separate geographic area with multiple Availability Zones. Resources are usually regional. An EC2 instance in one Region is separate from an instance with the same name in another Region. S3 bucket names are globally unique within an AWS partition, but the bucket itself is created in a selected Region.

Choose a Region by checking:

- Where users are located and the network latency they need.
- Data residency and regulatory requirements.
- Whether each required AWS service and feature is available there.
- The price of the service and expected data transfer.
- Whether the organization has approved that Region.

A workload can use multiple Regions for disaster recovery or global access, but doing so adds replication, identity, monitoring, and operational complexity. Multi-Region design should solve a stated requirement.

## Availability Zones

An Availability Zone is an isolated location within a Region. It contains one or more data centers with independent power, networking, and connectivity. AWS designs Availability Zones to reduce correlated failure, while keeping them close enough for low-latency connections within a Region.

A subnet belongs to one Availability Zone. A VPC can span multiple Availability Zones. A common pattern places application capacity in at least two zones and keeps a database subnet group across multiple zones. The service configuration must actually use that layout. Creating a subnet in a second zone alone does not make an application resilient.

Availability Zone names such as us-east-1a can map to different physical zones for different AWS accounts. AZ IDs such as use1-az1 identify the same physical zone across accounts. Use IDs when comparing cross-account placement.

## Edge locations and other placement options

CloudFront uses edge locations to cache and deliver content closer to viewers. Route 53 provides DNS services through a global network. These are different from placing an application server in every Region.

Local Zones extend selected AWS services closer to a large population center. Wavelength Zones place AWS infrastructure in carrier networks for applications with strict mobile latency requirements. These options are available only in selected locations and add design choices that should be justified by latency or locality requirements.

## Global, regional, and zonal resources

Service scope is not the same for every AWS resource. IAM is configured at the account level, while an EC2 instance is regional and a subnet is tied to one Availability Zone. Some services expose global endpoints while storing or processing data regionally.

Before relying on a resource in a second Region, ask whether the service replicates it automatically, requires a separate copy, or has no cross-Region behavior. Do not assume that a backup, key, image, policy, or log exists in a second Region because the original exists in one.

## Inspect regional availability

The AWS CLI can list Regions and Availability Zones visible to the caller. The selected profile needs permission to make these describe requests.

```bash
aws ec2 describe-regions --all-regions --query 'Regions[].{Name:RegionName,Status:OptInStatus}' --output table
aws ec2 describe-availability-zones --region ap-south-1 --query 'AvailabilityZones[].{Name:ZoneName,Id:ZoneId,State:State}' --output table
```

Use the service documentation to verify that the specific feature is available in the chosen Region. The Region list alone does not show every service limitation.

## Placement example

For a small application used in Bengaluru, ap-south-1 may be a reasonable Region to evaluate because it is in India. The decision still needs a latency check from actual users, a data-location review, a service availability check, and a cost comparison. A second Region should be chosen only if recovery or global access requirements justify the extra operations.

For a resilient regional design, place stateless application capacity in more than one Availability Zone. Keep the database private, enable the appropriate managed backup or replication feature, and test the failure path. A diagram should show which components are regional and which are shared or global.

## Key points

- Regions are independent geographic areas. Most resources do not appear in every Region.
- Availability Zones are isolated locations inside a Region. A subnet belongs to one zone.
- A multi-zone design needs services configured across zones, not just extra subnets.
- Availability Zone names may map differently across accounts. AZ IDs identify physical zones consistently.
- Edge delivery, multi-Region recovery, and specialized zones solve different requirements.

## Practice

1. Choose a candidate Region for an application and list the data, latency, service, approval, and price checks you would perform.
2. Draw two application subnets in separate Availability Zones and show the route to a database.
3. Use the CLI commands above to inspect Regions and AZ IDs. Do not create resources.
4. Explain why putting a second EC2 instance in the same Availability Zone may not protect against a zone failure.

## References

- [AWS global infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)
- [Regions and Availability Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
- [AWS service endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/aws-service-information.html)
- [AWS CloudFront locations](https://aws.amazon.com/cloudfront/features/)
