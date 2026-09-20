# Technologies – AWS S3 Object Lambda PII Redaction Architecture

## Purpose

This document describes the AWS services and technologies used in the proposed PII redaction architecture and the role each component plays in protecting sensitive data.

The repository is architecture-focused and does not represent a complete production deployment.

---

## 1. Amazon S3

### What It Is

Amazon S3 provides scalable object storage for structured and unstructured data.

### Role in the Architecture

S3 represents the authoritative storage location for source objects that may contain sensitive information.

The architecture preserves the original object while allowing a transformed representation to be returned to consumers that do not require access to all sensitive fields.

### Security Considerations

A production implementation should consider:

- Encryption at rest
- Bucket policies
- IAM authorization
- Public-access blocking
- Logging and monitoring
- Object lifecycle requirements
- Protection against unauthorized direct access

---

## 2. S3 Access Points

### What They Are

S3 Access Points provide dedicated access endpoints that can have policies tailored to specific applications or access patterns.

### Role in the Architecture

An S3 Access Point can provide the supporting access path used by S3 Object Lambda to retrieve the underlying object.

Access-point policies can also help separate different data-access patterns.

For example, an organization might distinguish between:

- Privileged access to original data
- Consumer access to transformed data

The exact access policy would depend on the identities, applications, and data-classification requirements involved.

---

## 3. S3 Object Lambda

### What It Is

S3 Object Lambda allows Lambda-based transformation logic to be incorporated into supported S3 object-retrieval requests.

### Role in the Architecture

Object Lambda represents the transformation layer between stored data and the consumer.

Conceptually:

**Consumer → Object Lambda → Transformation → Source Object → Transformed Response**

The architecture uses this pattern to demonstrate how sensitive fields can be removed from a consumer-facing representation without modifying the original stored object.

### Security Value

This pattern can support:

- Data minimization
- Controlled disclosure
- Request-time transformation
- Preservation of the authoritative source
- Reduced need for duplicate sanitized datasets

Object Lambda does not independently prevent access to the underlying source data. IAM and S3 authorization controls must govern alternate access paths.

---

## 4. AWS Lambda

### What It Is

AWS Lambda provides serverless execution of application logic in response to supported events and service integrations.

### Role in the Architecture

Lambda performs the data transformation.

The project's primary example removes the `ssn` field from structured JSON before the resulting representation is returned to the consumer.

Conceptually, the transformation logic checks whether the `ssn` field exists and removes it from the consumer-facing representation.

The source object remains unchanged.

### Production Considerations

Production transformation logic should include:

- Input validation
- Schema validation
- Error handling
- Secure failure behavior
- Appropriate logging
- Monitoring
- Performance testing
- Protection against logging PII

The simplified logic documented in this repository demonstrates the security pattern rather than a complete production Lambda implementation.

---

## 5. AWS Identity and Access Management (IAM)

### What It Is

AWS IAM controls authentication and authorization to AWS resources and services.

### Role in the Architecture

IAM is essential because redaction is effective only when consumers cannot bypass the intended access path and retrieve sensitive source data through unauthorized means.

A production implementation would need to define permissions for:

- Lambda execution
- Source-object retrieval
- S3 Access Point usage
- S3 Object Lambda usage
- Administrative access
- Privileged raw-data access

### Security Principle

Permissions should follow least privilege.

The transformation function should receive only the permissions required to perform its approved role.

Consumers receiving sanitized data should not automatically receive permission to retrieve the original sensitive object.

---

## 6. JSON

### What It Is

JSON is a structured data format based on key/value relationships.

### Role in the Architecture

The project uses JSON to demonstrate field-level data transformation.

A source object could contain fields such as:

- `customer_id`
- `name`
- `email`
- `ssn`
- `account_balance`

The transformation logic can identify `ssn` as a sensitive field and remove it before returning the consumer-facing representation.

Structured data allows transformation logic to identify specific fields rather than attempting to sanitize an entire object as unstructured text.

---

## 7. Amazon CloudWatch

### What It Is

Amazon CloudWatch provides monitoring, metrics, and logging capabilities for AWS workloads.

### Role in the Architecture

A production Lambda implementation would use logging and monitoring to provide visibility into transformation activity and failures.

Useful operational events could include:

- Transformation invocation
- Successful transformation
- Transformation failure
- Invalid input
- Processing latency

Sensitive values should not be written to logs.

For example, recording that a sensitive field was removed is preferable to recording the sensitive value itself.

---

## 8. Encryption

Sensitive source data should be protected both at rest and in transit.

Amazon S3 supports server-side encryption options including:

- SSE-S3
- SSE-KMS

If AWS KMS is used, IAM and key policies must allow only authorized principals to perform the required cryptographic operations.

Encryption protects stored and transmitted information, while Object Lambda transformation addresses a different security question:

**What portion of the information should a particular consumer receive?**

Both controls can therefore participate in a defense-in-depth architecture.

---

## 9. AWS KMS

### What It Is

AWS Key Management Service provides centralized management and control of cryptographic keys.

### Potential Role

KMS can be used when an organization requires customer-managed encryption keys, additional key-policy control, or specific encryption governance.

If source objects use SSE-KMS, the identities and services retrieving those objects must have the required KMS permissions.

KMS is an architectural option for this design rather than a demonstrated implementation in the current repository.

---

## 10. Terraform

### What It Is

Terraform is an Infrastructure-as-Code platform used to declaratively provision infrastructure.

### Potential Role

Terraform could be used to provision and manage:

- S3 resources
- S3 Access Points
- S3 Object Lambda configuration
- Lambda
- IAM roles and policies
- Logging and monitoring controls

Some project documentation originally described Terraform as the deployment mechanism.

However, Terraform configuration is not present in the current repository.

Terraform should therefore be considered a potential implementation approach rather than a completed project artifact.

---

## Technology Interaction

The proposed architecture can be summarized as:

**Amazon S3 → S3 Access Point → S3 Object Lambda → AWS Lambda Transformation → Consumer**

Supporting security controls include:

- **IAM** – Authorization and least privilege
- **CloudWatch** – Monitoring and operational visibility
- **Encryption/KMS** – Protection of stored information

---

## Architecture Principle

No individual AWS service provides the complete data-protection solution.

Effective protection depends on combining:

**Data classification + IAM authorization + storage security + transformation logic + monitoring + governance**

S3 Object Lambda provides the transformation capability, while the surrounding security architecture determines who may access the data, what information they may receive, and how those decisions are enforced.
