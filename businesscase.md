# Business Case: Redacting PII with AWS S3 Object Lambda

## 🔍 Problem Statement
Organizations storing sensitive data in Amazon S3 often need to provide access to third parties, internal teams, or applications—without exposing personally identifiable information (PII). Direct access to raw S3 objects can lead to inadvertent data exposure, violating privacy policies and data protection regulations.

## 💡 Solution Overview
This project implements AWS S3 Object Lambda in combination with AWS Lambda to dynamically redact PII (e.g., names, SSNs, emails) from S3 objects at the time of access. This enables secure sharing of sanitized data without modifying the source data, preserving original integrity while enforcing contextual privacy.

## 🎯 Business Value
- Zero duplication of redacted vs. non-redacted data
- Real-time redaction, reducing compliance risk
- Scalable and serverless — no manual intervention required
- Auditability via integration with CloudTrail and access logs
- Enables secure cross-team data access for analytics, QA, or third-party reviews

## 🔐 Compliance & Security Relevance

| Regulation/Framework | Relevance                                                                 |
|----------------------|---------------------------------------------------------------------------|
| GDPR                 | Supports "data minimization" and "purpose limitation" principles          |
| HIPAA                | Redaction supports safe data access for non-covered entities              |
| PCI DSS              | Enables secure handling of S3-stored cardholder data via Lambda filtering |
| ISO 27001            | Enforces access control and data masking for information security         |
| NIST 800-53          | Aligns with AC-6 (Least Privilege) and SC-12 (Cryptographic Key Establishment) |

## 🧑‍💻 Use Case Scenarios
- Financial services sharing transaction logs without customer info
- Healthcare applications providing anonymized records for research
- Public sector transparency reports that exclude citizen identifiers

## 🔄 Strategic Fit
This project aligns with Zero Trust principles by:
- Restricting data exposure at the point of access
- Enforcing redaction dynamically based on context
- Eliminating reliance on trust in static S3 ACLs or file copies
