# Cost Considerations – AWS S3 Object Lambda PII Redaction Architecture

## Purpose

This document identifies the primary cost considerations associated with implementing the proposed request-time PII redaction architecture.

The architecture uses managed and serverless AWS services conceptually, which can reduce the need for continuously running compute infrastructure.

Actual cost depends on workload volume, object size, transformation complexity, logging, monitoring, and the final implementation.

---

## 1. Request-Time Processing

Request-time transformation introduces processing each time applicable data is retrieved.

Potential cost drivers include:

- Number of object requests
- Amount of data processed
- Lambda invocation volume
- Lambda execution duration
- Object size
- Transformation complexity

For lightweight structured-data transformations, processing requirements may be relatively small.

However, cost should be evaluated using representative production workloads rather than assuming request-time transformation is always the least expensive approach.

---

## 2. Amazon S3

Amazon S3 costs can include:

- Data storage
- API requests
- Data transfer where applicable
- Access-point usage
- Other enabled S3 capabilities

One potential advantage of the architecture is reducing the need to maintain a separate sanitized copy of every source object.

Instead of maintaining:

**Original dataset + Sanitized dataset**

the architecture can potentially use:

**Original dataset + Request-time transformed representation**

This may reduce duplicate storage and synchronization requirements.

Whether this produces meaningful cost savings depends on dataset size, request frequency, and the alternative architecture being considered.

---

## 3. AWS Lambda

Lambda introduces execution costs for transformation requests.

Primary cost factors include:

- Number of invocations
- Execution duration
- Memory allocation
- Processing complexity

Simple field removal from relatively small JSON objects may require limited processing.

More complex transformations or larger objects can increase execution time and cost.

---

## 4. Logging and Monitoring

Production implementations should include appropriate logging and monitoring.

Potential cost drivers include:

- Log ingestion
- Log storage
- Metrics
- Alarms
- Retention periods
- Security monitoring

Logging should provide sufficient operational and security visibility without unnecessarily recording large payloads.

Sensitive values should not be written to logs.

Appropriate retention policies can help balance audit requirements with storage cost.

---

## 5. Encryption and AWS KMS

If SSE-KMS is selected for source-data encryption, AWS KMS usage may introduce additional costs associated with cryptographic operations and key management.

The decision between SSE-S3 and SSE-KMS should therefore consider both:

- Security and governance requirements
- Operational cost

Cost alone should not determine the encryption mechanism for sensitive information.

---

## 6. Data Transfer

Depending on the consumers and architecture, data-transfer charges may also need to be considered.

Relevant factors can include:

- AWS Region
- Consumer location
- Cross-Region access
- Internet data transfer
- Other connected AWS services

A production cost model should include the actual expected data flow rather than only Lambda and S3 processing costs.

---

## 7. Request-Time Transformation vs. Sanitized Copies

One of the important architecture tradeoffs is the cost relationship between dynamic transformation and maintaining sanitized datasets.

### Request-Time Transformation

Potential advantages:

- Reduced duplicate storage
- No batch sanitization process
- Centralized transformation logic
- Transformation occurs only when data is requested

Potential cost factors:

- Processing on each request
- Additional request-path services
- Monitoring
- Potentially higher latency

### Pre-Generated Sanitized Dataset

Potential advantages:

- Transformation occurs before consumption
- Consumer retrieval may be simpler
- Useful for frequently accessed static datasets

Potential cost factors:

- Additional storage
- ETL or transformation processing
- Synchronization
- Data lifecycle management
- Additional security controls

Neither approach is universally less expensive.

The appropriate architecture depends on workload characteristics.

---

## 8. Infrastructure Automation

Infrastructure as Code can help organizations consistently create and remove development and test resources.

Terraform is one possible implementation approach.

However, Terraform configuration is not present in the current repository and should not be considered an implemented cost-control mechanism for this project.

If Infrastructure as Code is added later, automated teardown and lifecycle management could help reduce unnecessary resource consumption.

---

## 9. Cost Monitoring

A production implementation should consider:

- AWS cost monitoring
- Budget thresholds
- Usage alerts
- Request-volume trends
- Lambda execution trends
- Logging growth
- Data-transfer patterns

Unexpected increases in cost can also provide operational signals that usage patterns have changed.

---

## Architecture Cost Principle

The economic value of request-time transformation should not be evaluated solely by the cost of a Lambda invocation.

The complete comparison should consider:

**Storage + Requests + Transformation + Logging + Monitoring + Encryption + Data Transfer + Operational Complexity**

Request-time redaction may reduce the cost and complexity of maintaining duplicate sanitized datasets, but that benefit must be evaluated against the additional processing introduced into every applicable retrieval request.
