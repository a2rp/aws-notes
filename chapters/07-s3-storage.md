[Back to notes index](../README.md)

| [Previous: EC2 and Auto Scaling](06-ec2-and-auto-scaling.md) | [Notes index](../README.md) | [Next: Databases with RDS and DynamoDB](08-databases-rds-and-dynamodb.md) |
| --- | --- | --- |
# 7. S3 storage

Amazon Simple Storage Service (S3) stores objects in buckets. It is useful for files, exports, backups, static assets, and data exchange. S3 is object storage, not a mounted disk or a traditional folder hierarchy.

## Buckets, objects, and keys

A bucket is created in a Region. Bucket names are unique within an AWS partition, so choose a name that does not contain private information and can remain stable. An object has a key, data, metadata, and optional tags. The slash characters in a key can make a console display look like folders, but the object namespace is flat.

An object key such as reports/2026/october.csv is one key. Prefixes such as reports/ help organize and query objects. Choose prefixes that support access patterns, lifecycle rules, and inventory.

S3 is designed for durable object storage, but durability does not replace deletion protection, versioning, backup strategy, or access control. Decide how objects will be retained and recovered before relying on a bucket for important data.

## Access control

New buckets are private by default. Keep all S3 Block Public Access controls enabled unless public access is a clear requirement with an approved architecture. For public static content, a private S3 origin behind CloudFront with Origin Access Control is generally easier to govern than a public bucket.

Use IAM identity policies for principals and bucket policies for resource-level rules. Account and organization controls can impose additional restrictions. S3 Object Ownership with bucket-owner-enforced settings disables ACLs and makes policy-based access easier to reason about for most new designs.

A presigned URL grants temporary access to a specific operation on an object. Anyone who has the URL can use it until it expires or the underlying access changes, so treat it like a temporary secret. Keep its lifetime short and do not log it where others can read it.

## Encryption and transport

S3 encrypts new objects at rest by default with S3-managed encryption. Server-side encryption with AWS KMS keys can provide key policy control and audit visibility, with additional configuration and possible request costs. Use TLS for requests in transit. Do not store credentials or sensitive values in object names or metadata.

A bucket policy can deny requests that do not use TLS. Review any policy change in a test bucket first, since a broad deny can also block legitimate access paths.

## Versioning and lifecycle

Versioning keeps prior object versions when an object is overwritten or deleted. It helps recover from some accidental changes, but old versions consume storage and can have their own retention requirements. A normal delete in a versioned bucket often creates a delete marker rather than removing every stored version.

Lifecycle rules can transition or expire objects based on age and prefix. Test rules against sample objects and understand noncurrent version behavior before applying them broadly. Replication requires versioning on the relevant buckets and needs a role that can read and write the replicated objects.

S3 Object Lock can enforce retention for protected records. Retention modes and legal holds need careful planning because they can prevent deletion even by administrators.

## Upload and inspect an object

Boto3 uses the configured credential provider chain. This example uploads a file under a prefix. The caller needs permission to write the target object. The bucket should have the encryption, access, lifecycle, and logging settings required by the workload.

```python
import boto3

s3 = boto3.client("s3", region_name="ap-south-1")
s3.upload_file(
    Filename="report.csv",
    Bucket="example-study-bucket",
    Key="exports/report.csv",
)
```

Generate a short-lived link only when a recipient needs temporary read access:

```python
url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": "example-study-bucket", "Key": "exports/report.csv"},
    ExpiresIn=300,
)
print(url)
```

The returned URL is a bearer credential for the requested operation. Do not publish it or save it in application logs.

## Lifecycle example

This rule expires objects under a temporary prefix after 30 days. Apply it only when the retention requirement permits deletion.

```json
{
  "Rules": [
    {
      "ID": "ExpireTemporaryExports",
      "Status": "Enabled",
      "Filter": { "Prefix": "temporary/" },
      "Expiration": { "Days": 30 }
    }
  ]
}
```

## Inspect bucket settings

These commands read bucket state. Replace the example name with a bucket you are authorized to inspect.

```bash
aws s3api get-public-access-block --bucket example-study-bucket
aws s3api get-bucket-versioning --bucket example-study-bucket
aws s3api get-bucket-encryption --bucket example-study-bucket
aws s3api head-object --bucket example-study-bucket --key exports/report.csv
```

## Key points

- S3 stores objects addressed by bucket and key. Prefixes are a naming convention, not real directories.
- Keep buckets private and use policy-based access. Do not enable public access as a quick fix for permission errors.
- Versioning helps recover previous versions but can increase storage use.
- Lifecycle, replication, and Object Lock need retention and deletion behavior designed deliberately.
- A presigned URL is a temporary credential. Keep its scope and lifetime narrow.

## Practice

1. Design key prefixes for user uploads, reports, and temporary exports.
2. Explain how versioning changes the result of overwriting and deleting an object.
3. Review a bucket's public access, encryption, and versioning settings with the CLI.
4. List the permissions and expiry controls needed before generating a temporary download link.

## References

- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Blocking public access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [S3 security best practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)
- [S3 versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- [S3 lifecycle management](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
- [Boto3 S3 presigned URLs](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/s3-presigned-urls.html)
