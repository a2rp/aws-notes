[Back to notes index](../README.md)

| [Previous: Messaging with SQS, SNS, and EventBridge](11-messaging-sqs-sns-and-eventbridge.md) | [Notes index](../README.md) | [Next: Infrastructure as code with CloudFormation and CDK](13-infrastructure-as-code-cloudformation-and-cdk.md) |
| --- | --- | --- |
# 12. Observability with CloudWatch, CloudTrail, and Config

Observability helps explain what a workload is doing and why it is failing. AWS provides services for application metrics and logs, account API activity, and resource configuration history. These data sources answer different questions and should be designed together.

## CloudWatch metrics and alarms

CloudWatch metrics are time-series values associated with a namespace and dimensions. AWS services publish many built-in metrics. Applications can publish custom metrics for business and service behavior.

A metric needs a clear name, unit, dimensions, and aggregation period. Avoid high-cardinality dimensions such as a unique request ID because each unique dimension combination can create additional metric series and cost. Use logs or traces for per-request detail.

An alarm evaluates a metric over a time window and changes state when its condition is met. Choose a threshold that indicates an actionable problem, account for missing data, and route the alarm to a team or response mechanism. An alarm without an owner and a runbook becomes noise.

## CloudWatch Logs

CloudWatch Logs stores log events in log groups and streams. Set an explicit retention period rather than leaving all logs forever by default. Retention should reflect operational needs, privacy requirements, and cost.

Write structured logs so fields can be queried. Include a request ID or correlation ID, service name, severity, and a concise event message. Do not write passwords, access tokens, session cookies, or unnecessary personal data. Sanitize user-controlled values to prevent forged log entries.

CloudWatch Logs Insights can search and aggregate events. A simple query can find recent error-level messages:

```text
fields @timestamp, @message
| filter level = "ERROR"
| sort @timestamp desc
| limit 20
```

The field names depend on the application's log format. Start with a small time range and narrow filters to control query cost and make results easier to read.

## CloudTrail audit events

CloudTrail records account activity made through the console, SDKs, CLI, and AWS services. Management events describe control-plane operations such as creating or changing a resource. Data events can record activity on supported resources, such as object-level access, but often need explicit selection and can produce additional event volume and cost.

Use a trail to deliver audit records to a protected destination, and consider organization-wide and multi-Region collection for centralized review. Restrict who can change trail settings or delete its logs. CloudTrail helps answer who called an API and when. It is not a replacement for application logs or a complete record of every application request.

## AWS Config resource history

AWS Config records resource configuration changes and can evaluate resources against rules. It helps answer what changed in a resource configuration and whether it complied with a rule at a point in time. Config is not the same as CloudTrail: one focuses on resource configuration state, while the other records API activity.

Review the recording scope, aggregation, rules, and retention for the accounts and Regions that matter. Config rules can report noncompliance, but a finding still needs an owner and a remediation process.

## Alarm example

This CloudFormation resource creates a CPU alarm for one EC2 instance. Replace the example identifier and connect the alarm to an approved notification target. A single CPU threshold is only an example; select signals that correspond to user impact and workload behavior.

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

TreatMissingData should match the signal. For an absent heartbeat, missing data may itself be a failure. For a short-lived resource that has been removed, missing data may be expected.

## Investigate a change

Use CloudTrail event history or a configured trail to find a relevant API event. Compare the event time with CloudWatch metrics and application logs. Use AWS Config history to inspect how the resource properties changed. Correlate times carefully because different sources may have different delivery delays and timestamps.

Read-only command examples:

```bash
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=RunInstances --max-results 10
aws configservice describe-compliance-by-config-rule --output table
aws cloudwatch describe-alarms --state-value ALARM --output table
```

## Key points

- CloudWatch covers metrics, alarms, logs, and dashboards for services and applications.
- CloudTrail records AWS API activity. Data events may require explicit configuration.
- AWS Config tracks resource configuration and rule compliance.
- Set log retention and avoid recording secrets or unnecessary personal data.
- Every important alarm needs an owner, a threshold tied to impact, and a response guide.

## Practice

1. Pick three signals for an API: one user-impact metric, one dependency metric, and one capacity metric.
2. Write an alarm condition and a short response step for rising queue age.
3. Explain which service you would use to determine who changed a security group and which service to inspect its configuration history.
4. Design a structured application log entry that includes a request ID without exposing a token.

## References

- [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [CloudWatch Logs Insights query syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [CloudTrail management and data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)
- [AWS Config Developer Guide](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)
