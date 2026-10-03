[Back to notes index](../README.md)

| [Previous: Reliability, backups, and disaster recovery](15-reliability-backups-and-disaster-recovery.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
| --- | --- | --- |
# 16. Cost management and architecture review

AWS cost is part of system design. The bill depends on the services used, configuration, Region, request volume, storage, data transfer, support, and commitments. Estimate cost before deployment, track actual usage, and review changes as the workload evolves.

## Understand cost dimensions

Compute can be billed by time, request, or allocated capacity. Storage can be billed by size, operations, retrieval, and retention. Networking can add charges for data transfer, NAT processing, load balancing, and endpoints. Logging and backups can grow over time even when application traffic remains steady.

Create cost allocation tags for application, owner, environment, and cost center. Activate the tags for billing reports. Untagged shared resources still need an owner and allocation method. Use separate accounts when a stronger billing and permission boundary is useful.

## Budgets and cost visibility

AWS Budgets can compare actual or forecasted costs with a threshold and notify an owner. A budget alert does not automatically stop spending. Cost Explorer helps analyze past usage by service, account, Region, or tag. Cost Anomaly Detection can identify unusual spend patterns, but an alert still needs an investigation path.

Review cost on a regular schedule and after architecture changes. Estimate the workload using the AWS Pricing Calculator, then compare the estimate with actual billing data after deployment. Check that the estimate includes network transfer, logging, backups, and idle capacity, not only the primary compute service.

A read-only Cost Explorer query can group a previous month's usage by service. The caller needs billing permissions, and Cost Explorer data may not be available immediately for a new account.

```bash
aws ce get-cost-and-usage --time-period Start=2026-09-01,End=2026-10-01 --granularity MONTHLY --metrics UnblendedCost --group-by Type=DIMENSION,Key=SERVICE --output json
```

Treat the returned report as financial data. Do not publish account-level bills or business-sensitive cost details without approval.

## Practical optimization

- Right-size compute from observed CPU, memory, network, and application latency.
- Scale capacity with demand and remove unused instances, disks, addresses, load balancers, and snapshots.
- Use storage lifecycle rules based on retention and access needs.
- Set log retention and avoid high-volume debug logs in steady state.
- Compare NAT Gateway use with private service endpoints for supported traffic.
- Review database capacity, backup retention, replicas, and idle development environments.
- Consider Savings Plans or Reserved Instances only after demand is understood and commitments match the workload.
- Use Spot capacity for interruptible work that can checkpoint or retry safely.

Optimization should preserve reliability and security requirements. A low monthly estimate is not a success if the service cannot recover or protect user data.

## Architecture review

The AWS Well-Architected Framework organizes review around operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability. Use the questions to find risks and trade-offs rather than treating the review as a scorecard.

For each workload, record:

1. User need, expected load, and success measures.
2. Data classification, access paths, and retention requirement.
3. Components, dependencies, Region, and failure domains.
4. Identity boundaries, network paths, encryption, and audit signals.
5. Scaling behavior, quotas, timeouts, retries, and overload behavior.
6. Backup, restore, RPO, RTO, and a tested incident process.
7. Expected and actual monthly cost, owners, and cleanup responsibilities.

A short decision record should explain why a service was selected, what alternative was considered, and which condition would cause the decision to be revisited.

## A small review record

| Decision | Current choice | Reason | Revisit when |
| --- | --- | --- | --- |
| Region | One approved Region near primary users | Meets latency and data-location needs | User base or regulatory needs change |
| Compute | Managed service sized from measurements | Reduces host administration | Runtime or scale requirements change |
| Data recovery | Daily protected backup with restore test | Meets the agreed recovery objective | RPO or RTO changes |
| Cost owner | Application team with monthly review | Makes spend visible to the service owner | Ownership model changes |

Replace this example with decisions for the actual workload. Do not use a template as evidence that a system meets its requirements.

## Key points

- Model storage, network, logs, backups, and idle resources in addition to compute.
- Budgets notify owners; they are not automatic hard spending limits.
- Tags make cost reports useful only after they are activated and consistently applied.
- Optimize from measured demand while retaining security and recovery controls.
- Revisit architecture decisions when traffic, data, users, or business objectives change.

## Practice

1. Estimate a small web workload and list at least five cost categories beyond instance hours.
2. Create a tagging plan for owner, application, environment, and cost center.
3. Review a hypothetical monthly bill and identify which service, Region, or request pattern needs investigation.
4. Write a decision record for choosing a single Region and define a condition that would justify a second Region.

## References

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS Pricing Calculator](https://calculator.aws/)
- [AWS Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [Cost allocation tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)
- [AWS cost optimization guidance](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
