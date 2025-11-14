Cost Considerations – S3 Object Lambda PII Redaction

Serverless-first architecture means there are no running servers and no hourly compute charges. Costs occur only when objects are retrieved.

S3 Object Lambda pricing is based on transformed data volume and request count. This keeps costs proportional to the number of data consumers rather than the dataset size.

Lambda is billed per millisecond of invocation, making the transformation cost-efficient for lightweight JSON redaction.

S3 storage cost is unaffected, because only one version of each object is stored (raw data). Redacted versions are generated dynamically to avoid duplicating objects.

CloudWatch log storage may grow depending on access frequency; enabling log retention limits long-term costs.

Terraform prevents resource sprawl, ensuring unused or test resources are destroyed cleanly to avoid unnecessary charges.
