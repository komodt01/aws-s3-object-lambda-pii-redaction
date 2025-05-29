# Redaction Logic – AWS S3 Object Lambda Redaction

This document explains the logic used by the AWS Lambda function to redact sensitive data from S3 objects using Object Lambda Access Points.

## 🎯 Objective
The goal of the Lambda function is to intercept object retrieval requests and redact sensitive fields (e.g., names, emails, SSNs) from the content before delivering the response.

## 🧪 Input Example (Original S3 Object)
```json
{
  "customer_id": "123456",
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "ssn": "123-45-6789",
  "account_balance": 10000
}
```

## 🛠️ Lambda Redaction Logic (Python)
```python
import json

def lambda_handler(event, context):
    # Extract the original object content (decoded)
    payload = event['body']
    data = json.loads(payload)

    # Redact sensitive fields
    redacted_fields = ['name', 'email', 'ssn']
    for field in redacted_fields:
        if field in data:
            data[field] = "[REDACTED]"

    # Return the modified object
    return {
        'statusCode': 200,
        'body': json.dumps(data)
    }
```

## ✅ Output Example (Redacted)
```json
{
  "customer_id": "123456",
  "name": "[REDACTED]",
  "email": "[REDACTED]",
  "ssn": "[REDACTED]",
  "account_balance": 10000
}
```

## 🔐 Security Considerations
- Redaction happens at request-time, ensuring the original object remains untouched.
- Lambda is permissioned only to read from S3 via a scoped IAM role.
- Object Lambda Access Point ensures external users cannot bypass redaction logic.

## 🔄 Benefits
- No need for duplicate sanitized copies of files
- Dynamic enforcement of privacy policies
- Easily auditable and modifiable via Lambda code updates
