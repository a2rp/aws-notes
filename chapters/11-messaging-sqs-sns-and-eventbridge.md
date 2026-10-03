[Back to notes index](../README.md)

| [Previous: Containers with ECR, ECS, and Fargate](10-containers-ecr-ecs-and-fargate.md) | [Notes index](../README.md) | [Next: Observability with CloudWatch, CloudTrail, and Config](12-observability-cloudwatch-cloudtrail-and-config.md) |
| --- | --- | --- |
# 11. Messaging with SQS, SNS, and EventBridge

Queues and event routing reduce direct dependencies between services. A producer can publish work without knowing which worker will process it. This improves isolation, but introduces delivery delay, retries, duplicate messages, and eventual processing that the system must handle.

## Amazon SQS queues

Amazon Simple Queue Service (SQS) stores messages until a consumer receives and deletes them. A standard queue provides high throughput with at-least-once delivery. A message can be delivered more than once, so consumers should be idempotent.

A visibility timeout hides a received message while a consumer processes it. If the consumer does not delete the message before the timeout, it can become visible again. Set the timeout above the expected processing time and extend it when a long task is still active. Long polling reduces empty receives and unnecessary requests.

A dead-letter queue (DLQ) receives messages that exceed the configured receive count. A DLQ preserves failed work for investigation, but it needs alarms, retention, and a redrive procedure. Do not leave messages there without an owner.

FIFO queues preserve order within a message group and support deduplication features. They can be useful when order matters for one entity, but they do not make the entire application side effect exactly once. The consumer still needs safe retry behavior.

## Amazon SNS topics

Amazon Simple Notification Service (SNS) publishes a message to subscribed endpoints. One topic can fan out to SQS queues, Lambda functions, or other supported protocols. A topic is useful when several consumers need to react independently to the same event.

Subscriptions can use filter policies to receive only matching messages. Keep the event contract stable and include identifiers that let consumers find the underlying data. A delivery to one subscriber can fail while others succeed, so monitor delivery and give critical subscribers a recovery path.

## Amazon EventBridge

Amazon EventBridge routes events from AWS services, custom applications, and supported partners to targets. An event bus receives events, and rules match selected fields and send matching events to targets. This is useful for event routing where producers should not know each consumer.

EventBridge patterns match JSON structure. Keep rules focused and test patterns against representative events. Configure retry and a dead-letter queue where delivery failure must be retained. Archives and replay can help recover or reprocess events when configured for the bus and workload.

## Choose the pattern

| Need | Suitable starting point |
| --- | --- |
| One worker should process each unit of queued work | SQS queue |
| Several subscribers should receive a notification | SNS topic with subscriptions |
| Match events against structured rules and route to targets | EventBridge event bus |
| Preserve order for a sequence within one entity | FIFO queue with a stable message group |

These are starting points, not rules. A common design uses EventBridge to route an event to SNS, which fans it out to separate SQS queues for independent consumers.

## Process SQS batches safely

An SQS event source mapping polls a queue and invokes Lambda with a batch. If a batch contains several messages and one fails, configure partial batch responses when the consumer should retry only failed messages. The event source mapping must enable the ReportBatchItemFailures response type.

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

The code assumes that process_message and ExpectedTemporaryError are defined by the application. Do not catch and ignore unexpected failures. Log enough context to investigate without printing secrets or full sensitive message bodies.

## Message design and operations

Messages should contain a schema version, event identifier, event type, creation time, and the fields consumers need to process or look up the event. Avoid copying large data sets into a message. Store the data in a suitable service and include a reference when that is safer.

For each queue or rule, monitor visible messages, age of oldest message, failed invocations, DLQ depth, and consumer capacity. A growing oldest-message age can indicate that producers are faster than consumers or a downstream dependency is failing.

## Key points

- SQS buffers work for consumers. SNS fans out notifications. EventBridge routes structured events.
- Standard queue delivery can include duplicates. Design side effects to be idempotent.
- Visibility timeout, receive count, and DLQ policy work together.
- FIFO ordering is scoped to message groups and does not remove the need for safe consumers.
- Monitor message age and DLQ depth, and define who will redrive failed work.

## Practice

1. Choose a messaging pattern for sending an order to one worker and for notifying three independent systems.
2. Explain what happens when a consumer receives a message but crashes before deleting it.
3. Design an idempotency key for processing a payment status event.
4. Draw a queue, consumer, retry path, DLQ, alarm, and operator redrive path.

## References

- [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [SQS visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
- [SQS dead-letter queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [Amazon SNS Developer Guide](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [Amazon EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [Lambda partial batch responses](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-errorhandling.html)
