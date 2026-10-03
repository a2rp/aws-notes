[Back to notes index](../README.md)

| [Previous: S3 storage](07-s3-storage.md) | [Notes index](../README.md) | [Next: Serverless with Lambda and API Gateway](09-serverless-lambda-and-api-gateway.md) |
| --- | --- | --- |
# 8. Databases with RDS and DynamoDB

Amazon Relational Database Service (RDS) and Amazon DynamoDB solve different data problems. Choose a database from the data relationships, consistency needs, and access patterns. Do not choose only because a service is managed or popular.

## Relational data with RDS

RDS operates managed relational database engines. Tables have defined columns and relationships, and SQL can join and filter related records. Relational databases are a good fit when transactions, constraints, joins, and flexible queries are central to the application.

The customer still owns schema design, query behavior, database users, application permissions, and data protection. Place a production database in private subnets. Allow its database port from the application security group, not from the public internet. Store credentials in a protected secret store and rotate them with a planned application change.

A Multi-AZ deployment improves availability through a standby in another Availability Zone and managed failover. A read replica is mainly for read scaling or a separate copy for certain recovery designs. A read replica is not a backup. Automated backups and snapshots serve different recovery needs, and restore procedures should be tested.

Use connection pooling or RDS Proxy when the application creates many short-lived connections. Set connection limits and timeouts. A sudden connection spike can exhaust database capacity even when CPU use is low.

## Access-pattern data with DynamoDB

DynamoDB is a managed key-value and document database. Its primary key determines how items are stored and queried. A table can use a partition key alone or a partition key and sort key. The access patterns should be known before choosing these keys.

A query uses a partition key and can narrow results by sort key. A scan reads across the table and can consume capacity as the table grows, so it is usually unsuitable for a frequent application path. A global secondary index supports an alternate key pattern, but it has its own storage, throughput, and consistency considerations.

On-demand capacity adjusts to request volume and is convenient when traffic is unpredictable. Provisioned capacity can be appropriate when demand is predictable and capacity is managed. Both models need monitoring and a plan for throttling. Partition keys should distribute traffic; a hot key can constrain throughput.

DynamoDB supports strongly consistent reads for supported table and local secondary index operations, while global secondary index reads are eventually consistent. Choose consistency based on the user-visible requirement. Time to Live can remove expired items asynchronously, so it should not be treated as an exact deletion timer.

## Compare the models

| Need | RDS | DynamoDB |
| --- | --- | --- |
| Relationships and joins | Natural fit | Usually modeled into access patterns |
| Flexible SQL queries | Strong fit | Query patterns need planned keys and indexes |
| Fixed high-throughput key lookups | Possible, with capacity planning | Strong fit with suitable partition keys |
| Transactions | Relational transaction model | Supports transactions with different limits and costs |
| Schema changes | Managed through database migrations | Items can have varying attributes, but key design remains critical |
| Operations | Engine settings, connections, backups, and queries | Keys, indexes, capacity, throttling, and item size |

## SQL and key-value examples

A relational schema can express a constraint at the database boundary:

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

Use parameterized statements in application code rather than joining user input into SQL text:

```python
cursor.execute(
    "SELECT id, title FROM notes WHERE owner_id = %s ORDER BY created_at DESC",
    (owner_id,),
)
```

For a DynamoDB table with partition key PK and sort key SK, the following query returns notes for one user. The key schema must match the table definition.

```python
from boto3.dynamodb.conditions import Key

response = table.query(
    KeyConditionExpression=Key("PK").eq("USER#42")
    & Key("SK").begins_with("NOTE#")
)
notes = response["Items"]
```

The table variable should come from a boto3 resource configured with the intended Region and role credentials. Paginate query results when a response includes a LastEvaluatedKey.

## Backups and recovery

Define recovery point and recovery time objectives before choosing a backup pattern. Enable the managed backup features that fit the database and retention requirement. Keep recovery copies protected from the same access path that can delete production data. Restore a copy in a controlled environment and verify that the application can use it.

Encryption at rest and in transit should be part of the design. Key policy access and database access are separate controls. A database snapshot may contain personal or sensitive data, so restrict who can copy, share, restore, or export it.

## Inspect database resources

These commands read service configuration and status. They do not print database passwords.

```bash
aws rds describe-db-instances --query 'DBInstances[].{Id:DBInstanceIdentifier,Engine:Engine,Status:DBInstanceStatus,Public:PubliclyAccessible}' --output table
aws dynamodb describe-table --table-name StudyNotes --query 'Table.{Name:TableName,Status:TableStatus,Keys:KeySchema}' --output json
```

## Key points

- Relational databases suit related data, transactions, and queries that benefit from SQL joins.
- DynamoDB works best when the access patterns and key structure are designed together.
- A read replica and a backup solve different problems.
- Keep databases private and grant network access from the application tier only.
- Test restore procedures and treat backups and snapshots as sensitive data.

## Practice

1. Choose RDS or DynamoDB for an order system and explain which reads and writes determine the choice.
2. Design a DynamoDB partition key and sort key for listing a user's notes by time.
3. Explain how Multi-AZ, read replicas, automated backups, and snapshots differ.
4. Review the sample SQL and identify the role of the index and parameter placeholder.

## References

- [Amazon RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
- [RDS Multi-AZ deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [DynamoDB core components](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
- [DynamoDB read consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
- [DynamoDB best practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
