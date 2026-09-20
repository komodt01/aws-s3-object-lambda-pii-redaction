# Security Requirements – AWS S3 Object Lambda PII Redaction Architecture

## Purpose

This document defines the security requirements for a production implementation of the proposed AWS S3 Object Lambda PII redaction architecture.

The requirements describe the intended security posture of the architecture.

They should not be interpreted as evidence that every control has been implemented in the current repository.

---

## 1. Security Objectives

### SO1 – Prevent Unauthorized Disclosure

Sensitive information must not be disclosed to consumers that are not authorized to receive it.

### SO2 – Enforce Data Minimization

Consumers should receive only the information required for their approved business purpose.

### SO3 – Preserve Source Data Integrity

Request-time transformation must not modify the authoritative source object solely to produce a sanitized consumer representation.

### SO4 – Enforce Authorized Access Paths

Consumers intended to receive transformed information must not be able to bypass the approved transformation path and retrieve sensitive source data through an unauthorized alternate path.

### SO5 – Provide Auditability

Security-relevant access, transformation activity, failures, and administrative changes should provide sufficient visibility for monitoring, investigation, and audit requirements.

### SO6 – Fail Securely

Transformation failures must not result in unauthorized disclosure of the original sensitive content.

---

## 2. Identity and Access Management

### IAM-1 – Least-Privilege Lambda Role

The Lambda execution role must receive only the permissions required to perform the approved transformation workflow.

Permissions should be scoped to the required:

- AWS services
- S3 resources
- Object prefixes where appropriate
- Logging resources
- Cryptographic keys when required

Broad administrative permissions should not be used.

### IAM-2 – Consumer Access

Consumers authorized only for sanitized data must not also receive permissions that allow unauthorized retrieval of the underlying sensitive source object.

### IAM-3 – Access-Point Policies

Access-point policies should restrict usage to approved principals and approved data resources.

Policies should reflect the intended separation between:

- Privileged raw-data access
- Transformed consumer access

### IAM-4 – No Embedded Credentials

Transformation code must not contain:

- AWS access keys
- Passwords
- API keys
- Long-lived credentials
- Other embedded secrets

AWS service authentication should use appropriately scoped IAM roles.

### IAM-5 – Privileged Access

Any identity permitted to retrieve the original sensitive object should be treated as privileged access and governed accordingly.

---

## 3. Data Protection

### DP-1 – Sensitive Field Removal

The transformation logic must remove fields classified as inappropriate for the requesting consumer.

The project's initial example uses the `ssn` field.

### DP-2 – Source Object Preservation

Transformation must operate on the consumer-facing representation without modifying the original source object.

### DP-3 – Encryption at Rest

Sensitive source objects must use an appropriate S3 server-side encryption mechanism.

Potential options include:

- SSE-S3
- SSE-KMS

The selected mechanism should reflect organizational data-classification and key-management requirements.

### DP-4 – Encryption in Transit

Sensitive information must be transmitted using encrypted transport.

AWS service endpoints and client access should use HTTPS/TLS.

### DP-5 – KMS Least Privilege

If SSE-KMS is used, key policies and IAM permissions must restrict cryptographic operations to authorized identities and services.

### DP-6 – Temporary Data

Transformation logic should minimize unnecessary persistence of sensitive information.

If temporary processing is required, the design must consider:

- Data sensitivity
- Storage location
- Encryption
- Retention
- Cleanup
- Access control

### DP-7 – Data Classification

Sensitive fields must be identified through an approved data-classification process rather than solely through application-code assumptions.

---

## 4. Transformation Security

### TS-1 – Input Validation

Transformation logic must validate incoming content before processing it.

Validation should consider:

- Expected data format
- Expected schema
- Required fields
- Unsupported structures
- Malformed input
- Size constraints

### TS-2 – Secure Error Handling

Errors must not expose:

- Sensitive values
- Credentials
- Internal implementation details
- Unnecessary stack traces

### TS-3 – Fail-Closed Behavior

When transformation cannot be completed safely, the architecture should deny or fail the request rather than return unmodified sensitive information to an unauthorized consumer.

### TS-4 – Controlled Transformation Rules

Redaction rules should be derived from approved data policies.

Changes to those rules should follow appropriate review and change-control procedures.

### TS-5 – Minimal Dependencies

Transformation code should minimize unnecessary third-party dependencies and external service calls.

Dependencies that are required should be appropriately maintained and reviewed for security risk.

---

## 5. Logging and Monitoring

### LM-1 – Transformation Logging

Production logging should provide visibility into events such as:

- Transformation invocation
- Successful transformation
- Transformation failure
- Invalid input
- Processing errors

### LM-2 – No PII in Logs

Raw sensitive values must not be written to operational logs.

For example, the system may record that an `ssn` field was removed without recording the SSN itself.

### LM-3 – Data Access Visibility

The production architecture should provide sufficient visibility into access to sensitive source data.

Depending on the implementation, appropriate AWS logging capabilities may include CloudTrail and other supported S3 logging mechanisms.

### LM-4 – Monitoring

Operational monitoring should consider:

- Transformation failure rates
- Processing latency
- Request volume
- Authorization failures
- Unexpected access patterns
- Service errors

### LM-5 – Alerting

Security or operational alerts should be considered for conditions such as:

- Repeated transformation failures
- Significant increases in errors
- Unexpected raw-data access
- Unauthorized access attempts
- Significant changes in request patterns

---

## 6. Access-Path Security

### AP-1 – Prevent Unauthorized Bypass

The security architecture must ensure that consumers intended to receive transformed information cannot bypass the transformation mechanism through another authorized S3 path.

### AP-2 – Administrative Access

Administrative and break-glass access to source data should be explicitly governed and monitored.

### AP-3 – Policy Testing

IAM, bucket, and access-point policies should be tested to verify both:

- Intended access is permitted.
- Unintended access is denied.

Access control should not be considered effective solely because a policy exists.

---

## 7. Infrastructure and Deployment Security

### INF-1 – Repeatable Deployment

A production implementation should use a controlled and repeatable infrastructure-deployment process.

Infrastructure as Code is preferred where appropriate.

Terraform is one potential implementation approach but is not currently included in this repository.

### INF-2 – Version Control

Infrastructure definitions and transformation source code should be maintained in version control.

### INF-3 – Change Control

Changes affecting:

- IAM
- Data-access paths
- Transformation rules
- Encryption
- Logging
- Source-data access

should follow appropriate review and approval processes.

### INF-4 – Rollback

Deployment processes should provide a controlled method for restoring a known-good version when a change causes security or operational problems.

Rollback procedures should be designed carefully so they do not reintroduce an insecure configuration.

---

## 8. Testing Requirements

### TEST-1 – Redaction Testing

Testing must verify that defined sensitive fields are excluded from unauthorized consumer responses.

### TEST-2 – Negative Testing

Testing should verify behavior when:

- Sensitive fields are missing
- Unexpected fields are present
- JSON is malformed
- Input schemas change
- Transformation fails

### TEST-3 – Authorization Testing

Testing must verify that consumers cannot retrieve sensitive source data through unauthorized access paths.

### TEST-4 – Logging Testing

Testing should confirm that operational logs provide useful information without recording sensitive values.

### TEST-5 – Failure Testing

Failure scenarios should confirm that transformation errors do not result in disclosure of the original sensitive content.

---

## 9. Governance Requirements

### GOV-1 – Data Ownership

Data owners should define which information is sensitive and which consumers are authorized to receive it.

### GOV-2 – Redaction Policy Ownership

Ownership must be established for approving and maintaining transformation policies.

### GOV-3 – Exceptions

Exceptions allowing access to unredacted information should require explicit authorization and appropriate documentation.

### GOV-4 – Periodic Review

Access permissions and transformation policies should be reviewed periodically and when business requirements change.

---

## 10. Compliance Alignment

The architecture can support controls associated with:

- Data minimization
- Access enforcement
- Least privilege
- Information protection
- Auditability
- Information sanitization
- Security by design

These concepts may be relevant to frameworks and regulatory requirements including:

- GDPR
- PCI DSS
- HIPAA Security Rule
- NIST SP 800-53
- ISO/IEC 27001

Use of this architecture does not establish compliance by itself.

Compliance depends on regulatory scope, data classification, complete implementation, organizational processes, testing, and demonstrated control effectiveness.

---

## Security Architecture Principle

The primary security control is not the Lambda function by itself.

The protection model depends on the combination of:

**Data classification + authorization + controlled access paths + transformation + encryption + monitoring + governance**

The transformation function enforces one part of the policy: removing information that the consumer is not authorized or required to receive.

The surrounding architecture must ensure that the consumer cannot simply bypass that control and retrieve the sensitive source data another way.
