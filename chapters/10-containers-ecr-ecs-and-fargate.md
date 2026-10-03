[Back to notes index](../README.md)

| [Previous: Serverless with Lambda and API Gateway](09-serverless-lambda-and-api-gateway.md) | [Notes index](../README.md) | [Next: Messaging with SQS, SNS, and EventBridge](11-messaging-sqs-sns-and-eventbridge.md) |
| --- | --- | --- |
# 10. Containers with ECR, ECS, and Fargate

Containers package an application with its runtime dependencies. Amazon Elastic Container Registry (ECR) stores container images. Amazon Elastic Container Service (ECS) schedules containers as tasks and services. AWS Fargate runs ECS tasks without the customer managing EC2 worker hosts.

## Build and store an image

A container image is an immutable set of layers identified by a digest. A tag such as 1.4.2 is a readable reference, but tags can be overwritten unless the repository prevents it. For production deployments, record the image digest and use a controlled release process.

Keep credentials and environment-specific secrets out of the image. Use a small trusted base image, scan dependencies, rebuild when base images receive security updates, and avoid including local build files. ECR repositories should have lifecycle rules that remove old unneeded images without deleting versions required for rollback.

A normal ECR workflow creates a repository, authenticates the Docker client with short-lived credentials, builds an image, tags it with the registry path, and pushes it. Review the registry account and Region before logging in.

```bash
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com
docker build -t reading-api:1.0.0 .
docker tag reading-api:1.0.0 123456789012.dkr.ecr.ap-south-1.amazonaws.com/reading-api:1.0.0
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/reading-api:1.0.0
```

The account ID, repository, image tag, and Region above are fictional. Build and push only to a registry you own or are authorized to use.

## ECS concepts

An ECS cluster is a logical grouping for services and tasks. A task definition describes the containers, image, ports, CPU, memory, environment, logging, and roles. A task is one running copy of a task definition. An ECS service maintains the requested number of tasks and can replace unhealthy tasks.

The task execution role lets the ECS agent perform startup work such as pulling a private ECR image and sending container logs. The task role is made available to application code inside the container when it calls AWS APIs. These roles have different purposes and should be scoped separately.

Fargate tasks use the awsvpc network mode and receive network interfaces in selected subnets. Security groups control traffic to and from the task. Place application tasks in private subnets when a public load balancer can receive internet traffic. The service should use health checks and a deployment configuration that keeps enough healthy capacity during updates.

## Fargate, ECS on EC2, and EKS

Fargate removes the need to maintain the container host fleet. It is useful when a team wants to focus on tasks and services. ECS on EC2 provides control over the worker instances and can suit special host, capacity, or cost needs. Amazon EKS runs Kubernetes and is appropriate when Kubernetes APIs or ecosystem compatibility are requirements.

The orchestrator choice affects operations. Kubernetes introduces its own control plane concepts and add-ons. Do not choose it only because containers are used. Compare operational skills, workload constraints, networking, deployment, and total cost.

## Task definition example

This task definition describes one Fargate-compatible container. Replace the fictional image and role ARNs, create the log group, and select valid CPU and memory values before registration. This definition alone does not create a service or start a task.

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

The container process must listen on the configured port and bind to an address reachable through its task network interface. Production configuration should obtain secrets from a managed secret store rather than embedding them in the task definition.

## Deploy and observe

A deployment updates the task definition revision and asks the ECS service to use it. Use a load balancer health check that reflects whether the application is ready. Watch deployment events, task stopped reasons, CPU, memory, and application logs. Roll back when the new task set cannot become healthy.

Do not assume a task's local filesystem is durable. Persist important data in an appropriate database or object store. If a workload writes temporary files, understand the task's storage and lifecycle behavior.

## Key points

- ECR stores images, ECS defines and runs tasks, and Fargate manages the task host capacity.
- Pin and track image versions. A mutable tag alone may not identify the exact image deployed.
- The execution role supports ECS startup. The task role is for application calls.
- Network mode, subnet, security group, and load balancer settings determine task reachability.
- Containers are replaceable. Persist important application state outside the container filesystem.

## Practice

1. Draw the path from a source commit to a container image in ECR and then to an ECS service.
2. Explain why the task role should not contain permissions needed only by the ECS agent.
3. Review the task definition and list which values must match the actual image, port, roles, and log group.
4. Compare Fargate and ECS on EC2 for a workload with steady, predictable capacity.

## References

- [Amazon ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [Amazon ECR User Guide](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [Task execution IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html)
- [Task IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)
