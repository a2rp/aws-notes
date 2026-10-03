[Back to notes index](../README.md)

| [Previous: AWS CLI, SDKs, and CloudShell](04-aws-cli-sdk-and-cloudshell.md) | [Notes index](../README.md) | [Next: EC2 and Auto Scaling](06-ec2-and-auto-scaling.md) |
| --- | --- | --- |
# 5. VPC networking

A Virtual Private Cloud (VPC) is a logically isolated network in an AWS Region. A VPC defines address space and routing boundaries for resources such as EC2 instances, load balancers, and databases. A VPC is regional. Each subnet belongs to one Availability Zone.

## Address ranges and subnets

A VPC has one or more IPv4 CIDR blocks. Choose a range that does not overlap with connected networks such as a company network, another VPC, or a VPN. Overlapping ranges make routing and future connectivity difficult. A subnet receives a smaller range from the VPC and cannot span Availability Zones.

Plan subnet ranges before deployment. Leave room for additional application tiers and zones. AWS reserves some addresses in every subnet, so the usable count is smaller than the total addresses in the CIDR block.

A public subnet and a private subnet are defined by their routes, not by their names. A subnet is public when its route table sends internet-bound traffic to an Internet Gateway. A resource also needs a public address and a security group rule that allows the traffic. A private subnet has no direct route from the internet to its resources.

## Route tables and gateways

A route table contains destination ranges and a target. Every subnet is associated with one route table, either explicitly or through the VPC main route table. More-specific routes take precedence over less-specific routes.

An Internet Gateway is attached to a VPC and provides an internet route for resources with public addressing. It does not automatically make every resource public. A NAT Gateway lets private subnet resources initiate outbound connections without accepting unsolicited inbound internet connections. NAT Gateways have hourly and data processing charges, so use VPC endpoints for supported AWS services when they fit the design.

A gateway endpoint provides private access to supported services such as S3 and DynamoDB. An interface endpoint uses private network interfaces for supported services. Endpoint policies and service policies still control what requests are allowed.

## Security groups and network ACLs

Security groups are stateful virtual firewalls attached to network interfaces. They contain allow rules. When a connection is allowed in one direction, response traffic is tracked automatically.

Network ACLs are stateless filters associated with subnets. Their inbound and outbound rules are evaluated independently and can allow or deny traffic. Because they are stateless, return traffic may need an explicit rule. Security groups are usually the first control to review for an application connection.

Prefer rules that name the source security group for service-to-service traffic. For example, a database security group can allow the database port from the application security group, rather than from every IP address.

## A small public subnet template

This CloudFormation example creates a VPC, a subnet, an Internet Gateway, and a default route. It does not create an instance or an inbound security group rule. A route alone does not expose an application. Review CIDR ranges and Region placement before deploying a template.

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

The template has no AvailabilityZone property, so AWS chooses one in the deployment Region. Add a second subnet in another zone for a multi-zone design. A production template should also define resource tags, network controls, logging, and deployment parameters appropriate to the workload.

## Trace a request path

For a public web request, trace each hop: client, DNS name, load balancer, application target, and response. Check the route table, public addressing, and security group at each boundary. For an application-to-database request, check the application egress rule, database ingress source, database port, subnet route, and database endpoint.

Use read-only commands to inspect existing networking:

```bash
aws ec2 describe-vpcs --query 'Vpcs[].{Vpc:VpcId,Cidr:CidrBlock,State:State}' --output table
aws ec2 describe-subnets --query 'Subnets[].{Subnet:SubnetId,Vpc:VpcId,Zone:AvailabilityZone,Cidr:CidrBlock}' --output table
aws ec2 describe-route-tables --query 'RouteTables[].{Table:RouteTableId,Routes:Routes}' --output json
```

## Key points

- A VPC is regional. A subnet belongs to one Availability Zone.
- Public and private describe routing and reachability, not the subnet name.
- A public route needs a public address and an allowed security group path before a resource is reachable.
- Security groups are stateful and allow-only. Network ACLs are stateless and can allow or deny.
- NAT Gateways and interface endpoints can add cost. Compare them with supported gateway endpoints.

## Practice

1. Draw a VPC with two public application subnets and two private database subnets across separate Availability Zones.
2. Trace an inbound web request and an application-to-database connection through routes and security groups.
3. Explain why an EC2 instance with no public IP is not reachable from the internet through an Internet Gateway route alone.
4. Compare a NAT Gateway with an S3 gateway endpoint for private subnet access to object storage.

## References

- [VPC user guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Route tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [VPC endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)
