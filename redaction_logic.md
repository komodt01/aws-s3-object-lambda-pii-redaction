# Redaction Logic – AWS S3 Object Lambda PII Redaction

## Purpose

This document describes the transformation logic used to demonstrate how sensitive fields can be removed from structured data before that data is returned to a consumer.

The example focuses on **field removal for data minimization** rather than maintaining a second sanitized copy of the source object.

The original S3 object remains unchanged.

---

## Example Source Object

```json
{
  "customer_id": "123456",
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "ssn": "123-45-6789",
  "account_balance": 10000
}
