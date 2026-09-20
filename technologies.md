# Technologies – AWS S3 Object Lambda PII Redaction Architecture

## Purpose

This document describes the AWS services and technologies used in the proposed PII redaction architecture and the role each component plays in protecting sensitive data.

The repository is architecture-focused and does not represent a complete production deployment.

---

# 1. Amazon S3

## What It Is

Amazon S3 provides scalable object storage for structured and unstructured data.

## Role in the Architecture

S3 represents the authoritative storage location for source objects that may contain sensitive information.

The architecture preserves the original object while allowing a transformed representation to be returned to consumers that do not require access to all sensitive fields.

## Security Considerations

A production implementation should consider:

- Encryption at rest
- Bucket policies
- IAM authorization
- Public-access blocking
- Logging and monitoring
- Object lifecycle requirements
- Protection against unauthorized direct access

---

# 2. S3 Access Points

## What They Are

S3 Access Points provide dedicated access endpoints that can have policies tailored to specific applications or access patterns.

## Role in the Architecture

An S3 Access Point can provide the supporting access path used by S3 Object Lambda to retrieve the underlying object.

Access-point policies can also help separate different data-access patterns.

For example, an organization might distinguish between:

- Privileged access to original data
- Consumer access to transformed data

The exact access policy would depend on the identities, applications, and data-classification requirements involved.

---

# 3. S3 Object Lambda

## What It Is

S3 Object Lambda allows Lambda-based transformation logic to be incorporated into supported S3 object-retrieval requests.

## Role in the Architecture

Object Lambda represents the transformation layer between stored data and the consumer.

Conceptually:

**Consumer → Object Lambda → Transformation → Source Object → Transformed Response**

The architecture uses this pattern to demonstrate how sensitive fields can be removed from a consumer-facing representation without modifying the original stored object.

## Security Value

This supports:

- Data minimization
- Controlled disclosure
- Request-time transformation
- Preservation of the authoritative source
- Reduced need for duplicate sanitized datasets

Object Lambda does not independently prevent access to the underlying source data. IAM and S3 authorization controls must govern any alternate access paths.

---

# 4. AWS Lambda

## What It Is

AWS Lambda provides serverless execution of application logic in response to supported events and service integrations.

## Role in the Architecture

Lambda performs the data transformation.

The project's primary example removes the `ssn` field from structured JSON before the resulting representation is returned to the consumer.

Conceptually:

```python
if "ssn" in data:
    del data["ssn"]
