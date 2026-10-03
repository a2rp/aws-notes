[Back to notes index](../README.md)

| [Previous: Databases with RDS and DynamoDB](08-databases-rds-and-dynamodb.md) | [Notes index](../README.md) | [Next: Containers with ECR, ECS, and Fargate](10-containers-ecr-ecs-and-fargate.md) |
| --- | --- | --- |
# 9. Serverless with Lambda and API Gateway

AWS Lambda runs code in response to events without requiring the customer to manage the underlying server fleet. Amazon API Gateway can provide HTTP endpoints that invoke a function. This can reduce infrastructure work, but the application still needs careful permissions, retries, limits, observability, and cost controls.

## Lambda execution model

A Lambda function has code, a runtime, memory and timeout settings, an execution role, and an event source. AWS creates an execution environment to run an invocation. The environment may be reused, but the function must not depend on reuse for correctness.

Keep handlers stateless. Store durable state in a database or object store. Initialize SDK clients outside the handler when they can be safely reused. Do not keep one user's sensitive data in global variables because an execution environment may process later invocations.

The execution role grants the function permission to call AWS services. Give each function the smallest permission set it needs. A role used by a function that reads one S3 prefix should not have broad access to every bucket.

Memory affects the CPU available to a function as well as its memory limit. Timeout controls the maximum duration of one invocation. Set both from measurement and expected work. Increasing timeout can hide a slow dependency while increasing the time a request remains active.

## Events, retries, and idempotency

Lambda can be invoked synchronously by an HTTP integration or asynchronously by an event source. Failure and retry behavior depends on the invocation model. An event source mapping for SQS reads batches from a queue, while other integrations follow their own delivery behavior.

Assume an event can be delivered more than once. Make state-changing handlers idempotent by storing a stable event or request identifier and checking whether it was already processed. Use a dead-letter queue or failure destination where supported, and monitor it for work that needs investigation.

Avoid recursive invocation chains. A function that writes to the same event source that triggers it can create repeated work and unexpected charges. Configure concurrency limits and alarms for important functions.

## API Gateway and HTTP behavior

API Gateway defines routes, methods, integrations, authorization, throttling, and response behavior. Choose an API type based on the needed feature set and client behavior. Keep request and response contracts explicit, and validate inputs before expensive processing.

CORS is a browser cross-origin policy, not an authorization system. Configure allowed origins, methods, and headers narrowly. Use a real identity and authorization mechanism for protected operations. Authentication identifies a caller; authorization decides what that caller can do.

API Gateway needs permission to invoke the Lambda function. This is commonly expressed as a resource-based permission on the function. The function execution role has separate permissions for the AWS services used by its code.

## A small Python handler

This handler expects a JSON object with a title field. It returns a status code and JSON body in the API response format. Production handlers should validate the full contract, handle expected errors, and avoid returning internal exception details.

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

The event shape depends on the integration and payload format. Test with representative events from the configured API Gateway version. The handler above validates input but does not persist data.

## Local configuration and secrets

Use environment variables for non-secret operational settings such as a table name or log level. Store credentials and other secrets in a managed secret service and grant the function role access only to the required secret. Do not put secrets in function source code, deployment artifacts, or public logs.

Lambda environments have limits on execution duration, payload size, concurrency, and temporary storage. Check current quotas for the Region and integration before designing around a limit. For long-running or high-volume work, compare Lambda with containers or managed services.

## Observability and cost

Write structured logs to CloudWatch Logs with a request or correlation identifier. Keep logs useful and avoid passwords, tokens, full payment details, or unnecessary personal data. Track invocation errors, duration, throttles, concurrency, and downstream failures. Trace a request across API Gateway, Lambda, and data services when the issue spans components.

Cost depends on requests, execution duration, allocated memory, data transfer, logging, and surrounding services. A free or low request count does not make a design free if it also creates a NAT Gateway, database, or large log volume.

## Key points

- Lambda removes server fleet management, not application and access-control responsibilities.
- Handlers should be stateless and safe to run more than once.
- Retry behavior varies by invocation source. Design failure handling for the selected integration.
- The function execution role and API Gateway invocation permission are separate controls.
- Measure memory, duration, concurrency, logs, and downstream costs.

## Practice

1. Write two test events for the handler: one valid request and one invalid body.
2. Explain why a queue-triggered handler should be safe when the same message arrives again.
3. Draw the separate permission paths from a caller to API Gateway, from API Gateway to Lambda, and from Lambda to a database.
4. Choose a timeout and memory starting point for a small request handler and list the metrics you would use to adjust them.

## References

- [AWS Lambda best practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Lambda execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
- [Lambda invocation](https://docs.aws.amazon.com/lambda/latest/dg/lambda-invocation.html)
- [API Gateway documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
