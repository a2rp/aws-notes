[Back to notes index](../README.md)

| [Previous: Security, encryption, and secrets](14-security-encryption-and-secrets.md) | [Notes index](../README.md) | [Next: Cost management and architecture review](16-cost-management-and-architecture-review.md) |
| --- | --- | --- |
# 15. Reliability, backups, and disaster recovery

Reliability means a workload performs its intended function correctly and consistently. A reliable design anticipates failures, limits their impact, monitors service health, and practices recovery. AWS provides building blocks, but the application and its operators determine whether the recovery path works.

## Define recovery objectives

Recovery Point Objective (RPO) describes how much recent data loss a business can tolerate. Recovery Time Objective (RTO) describes how long the service can remain unavailable. These objectives should come from business impact, not from a default setting in a console.

A five-minute RPO may require frequent replication or durable event capture. A four-hour RTO may allow restore from a backup. Lower objectives usually increase cost and design complexity. Confirm whether the stated objective applies to one component or the whole user-facing service.

## Design for failure domains

A single Availability Zone failure should not remove every component needed by the service if the availability requirement calls for multi-zone operation. Place capacity across zones, use health checks that detect an inability to serve requests, and ensure the load balancer can route to healthy targets.

A multi-zone topology does not protect against every application bug, compromised credential, accidental deletion, or regional outage. Replication can copy bad data or destructive changes. Backups and access controls address different failure modes from high availability.

Use retries carefully. Retry transient failures with bounded attempts, backoff, and jitter. Make operations idempotent when possible. Avoid retrying permanent validation failures or multiplying a failing dependency's load. Apply timeouts and define what the application returns when a downstream service is unavailable.

## Backups and restore tests

A backup is useful only if it can be restored within the required time and yields usable data. Identify which resources need backup, how often, how long copies remain, which account or vault protects them, and who can delete them.

AWS Backup centralizes backup plans for supported services. Separate backup administration from production administration where possible. A protected backup vault, vault lock, or cross-account copy can reduce the impact of an attacker or mistaken deletion, subject to the resource and service requirements.

Restore a sample into an isolated environment. Verify data integrity, access controls, application compatibility, and restore time. Record the procedure and repeat the exercise after material architecture changes.

## Disaster recovery patterns

| Pattern | Recovery behavior | Typical trade-off |
| --- | --- | --- |
| Backup and restore | Rebuild after a failure from protected backups and infrastructure definitions | Lowest standing cost, longer recovery |
| Pilot light | Keep core data or minimal services ready, then scale the rest during recovery | Faster than a full rebuild, still needs deployment and scaling steps |
| Warm standby | Run a reduced-capacity copy that can scale up | Shorter recovery, ongoing operating cost |
| Multi-site active | Serve traffic from multiple locations continuously | Fast recovery, highest operational and data consistency complexity |

Choose the simplest pattern that meets the agreed RPO and RTO. A second Region is not automatically a disaster recovery plan. It needs tested replication, identity, DNS, quotas, observability, backups, and a controlled failover process.

## SDK retry configuration

AWS SDKs implement retry behavior. This Python example asks Botocore to use its standard retry mode with a bounded number of attempts. Check the SDK version and operation semantics. A retry does not make a non-idempotent write safe to repeat.

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

## Recovery exercise

For a reading-list service, write down how each component recovers:

- The API can be redeployed from versioned infrastructure and application artifacts.
- The database can be restored from a protected backup or promoted according to its configured recovery mode.
- Static files can be recovered from object versions or a protected copy.
- Identity and encryption permissions are available in the recovery account and Region.
- DNS and load balancing send traffic only after the application is healthy.

Then run the exercise in a non-production environment. Measure actual restore and failover time, and compare it with the objectives.

## Key points

- RPO measures tolerable data loss. RTO measures tolerable service recovery time.
- High availability, replication, and backup solve different failure problems.
- Retries need bounds, timeouts, jitter, and idempotent operation design.
- Restore tests reveal whether backups meet real recovery requirements.
- A multi-Region copy needs a tested operating procedure and compatible identity and data paths.

## Practice

1. Choose an RPO and RTO for a personal notes service and explain the business reason.
2. Compare a backup restore with a warm standby for cost and recovery steps.
3. Draw how a database failover affects application connections and request retries.
4. Write a recovery checklist for a Region outage, including identity, data, DNS, quotas, and validation.

## References

- [AWS Well-Architected Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)
- [AWS Backup Developer Guide](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [AWS Backup vault lock](https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html)
- [Disaster recovery options in the cloud](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)
- [AWS SDK retry behavior](https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html)
