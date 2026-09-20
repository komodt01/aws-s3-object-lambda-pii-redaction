# AWS S3 Object Lambda – PII Redaction Architecture

## Project Overview

This project explores a data-security architecture for protecting sensitive information stored in Amazon S3 by transforming object content before it is returned to a requester.

The design uses **Amazon S3 Object Lambda** and **AWS Lambda** to introduce a transformation layer between stored data and the consumer. Sensitive fields can be removed or redacted at request time while the original object remains unchanged.

The repository focuses on the **security architecture, requirements, risks, compliance considerations, and redaction logic** associated with this pattern.

It does not represent a complete production deployment.

---

## Business Problem

Organizations frequently store structured data containing Personally Identifiable Information (PII) in Amazon S3.

Different consumers may require access to the same dataset without requiring access to every sensitive field.

Maintaining separate sanitized copies can introduce:

- Duplicate data
- Additional storage
- Synchronization problems
- Data lifecycle complexity
- Increased risk of inconsistent protection

The architectural question is:

**How can an organization provide a sanitized view of an S3 object without modifying or duplicating the authoritative source object?**

---

## Architecture Approach

The proposed architecture introduces a transformation layer using S3 Object Lambda.

Conceptually:

**Requester → S3 Object Lambda Access Point → Lambda Transformation → S3 Object → Sanitized Response**

The original object remains the authoritative data source.

When a consumer requests data through the controlled access path, Lambda transforms the content before it is returned.

This separates:

**Data at rest**

from

**Data presented to the consumer**

and allows disclosure controls to be applied during retrieval.

---

## Conceptual Request Flow

1. Sensitive structured data is stored in Amazon S3.
2. A consumer requests an object through an S3 Object Lambda access path.
3. S3 Object Lambda invokes a Lambda transformation function.
4. The transformation logic evaluates the structured data.
5. Defined sensitive fields are removed or redacted.
6. The transformed content is returned to the requester.
7. The original S3 object remains unchanged.

This is the intended architecture pattern and should not be interpreted as evidence that every component has been deployed as a production system.

---

## Redaction Strategy

The architecture demonstrates field-level protection of structured JSON data.

Example source object:

```json
{
  "customer_id": "123456",
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "ssn": "123-45-6789",
  "account_balance": 10000
}
