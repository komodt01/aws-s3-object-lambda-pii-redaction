# Compliance Mapping – AWS S3 Object Lambda Redaction Project

This document maps key security and privacy controls from major frameworks to the functionality implemented in this project.

## 🛡️ AWS S3 Object Lambda + IAM + Lambda Compliance Mapping

| Framework      | Control ID      | Control Description                                              | Implementation Details                                           |
|----------------|------------------|------------------------------------------------------------------|------------------------------------------------------------------|
| **NIST 800-53**| AC-6             | Least Privilege                                                  | IAM policy grants only `s3:GetObject*` actions required          |
|                | SC-12            | Cryptographic Key Establishment                                  | S3 encryption in transit and at rest assumed                    |
|                | AU-2             | Audit Events                                                     | CloudTrail can be used to log Lambda and S3 Access Point usage  |
|                | SC-28            | Protection of Information at Rest                                | S3 bucket encryption enabled                                    |
|                | SC-18            | Mobile Code                                                       | Lambda function sandboxed in AWS-managed runtime                |
|                | SI-10            | Information Input Validation                                     | Lambda redacts known sensitive fields from object content       |
|                | AC-4             | Information Flow Enforcement                                     | Redaction is enforced before object reaches requester           |

| **ISO 27001**  | A.8.2.3          | Handling of assets – Prevent unauthorized access to PII          | Redaction before access via Object Lambda ensures data privacy  |
|                | A.9.1.2          | Access Control – User Access Management                          | IAM policies restrict who can invoke Lambda or access S3        |
|                | A.10.1.1         | Cryptographic Controls                                            | S3 default encryption supports secure storage                   |
|                | A.12.4.1         | Event Logging                                                     | CloudTrail tracks API usage and Lambda access                   |

| **PCI DSS**    | Req 3.3          | Mask PAN when displayed                                          | Object Lambda can redact cardholder data dynamically            |
|                | Req 7.1          | Limit access to cardholder data by business need to know         | IAM and Lambda code enforce strict access conditions            |
|                | Req 10.2         | Audit Trails for Access                                          | CloudTrail + S3 Access Logs fulfill this control                |

| **GDPR**       | Art. 5(1)(c)     | Data Minimization                                                 | Only necessary fields are returned to users via redaction       |
|                | Art. 25          | Data Protection by Design and by Default                         | Redaction logic built directly into access flow                 |
|                | Art. 32          | Security of Processing                                            | Uses AWS managed encryption and scoped IAM policies             |

| **HIPAA**      | §164.312(a)(1)   | Access Control                                                    | Object Lambda and IAM policies restrict PHI access              |
|                | §164.312(b)      | Audit Controls                                                    | All accesses can be logged and reviewed                         |
|                | §164.312(c)(1)   | Integrity Controls                                                | Lambda ensures content is sanitized before delivery             |

