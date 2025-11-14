# Risks and Mitigations – AWS S3 Object Lambda PII Redaction

This document outlines the key security, operational, compliance, and architectural risks associated with the AWS S3 Object Lambda PII Redaction solution, along with recommended mitigations.  
The goal is to ensure that the design is safe, auditable, resilient, and compliant.

---

# 1. Security Risks

## **SR-1 – Incomplete Redaction (PII Not Removed)**
If the Lambda logic fails to detect all PII fields or encounters unexpected JSON structure, sensitive data may be returned to the requester.

### Mitigations
- Validate JSON structure before processing.
- Add fallback rules to explicitly remove known PII fields.
- Implement unit tests for redaction logic.
- Log detection/removal events for auditing.
- Include a “future PII patterns” section for easy expansion.

---

## **SR-2 – Direct S3 Bucket Access Bypasses Redaction**
If clients can access the underlying S3 bucket directly, they may retrieve unredacted objects.

### Mitigations
- Deny direct S3 access via bucket policy.
- Allow access **only** through Access Points.
- Explicitly block S3:GetObject for all non-privileged roles.
- Monitor CloudTrail for suspicious direct access attempts.

---

## **SR-3 – Excessive IAM Permissions**
Overly broad IAM permissions could allow Lambda (or users) to read or modify sensitive data outside intended scope.

### Mitigations
- Enforce least privilege IAM policies.
- Use resource-level restrictions (e.g., only certain prefixes).
- Avoid `s3:*` or `iam:*` wildcards.
- Review Terraform plan output during code review.

---

## **SR-4 – Logging Sensitive Data**
If Lambda logs full objects or PII values, logs may leak sensitive information.

### Mitigations
- Never log raw object contents.
- Only log metadata (e.g., “SSN removed”).
- Enforce log retention & protection through CloudWatch.
- Enable CloudTrail and restrict access to log groups.

---

# 2. Compliance Risks

## **CR-1 – Violation of Data Minimization Requirements**
If PII is not reliably removed, the system may violate GDPR, HIPAA, PCI, or NIST standards.

### Mitigations
- Confirm that all required fields are removed before release.
- Maintain version-controlled evidence of redaction logic.
- Document compliance mappings in `requirements.md` and `security_requirements.md`.

---

## **CR-2 – Data Residency or Retention Issues**
If Lambda or S3 store temporary files, sensitive data may persist longer than intended.

### Mitigations
- Avoid writing to `/tmp` in Lambda.
- Remove any unused persistence.
- Use S3 lifecycle policies for internal copies (if created later).

---

## **CR-3 – Misconfigured Access Point Policies**
Incorrect policies may expose sanitized or raw datasets unintentionally.

### Mitigations
- Validate policies via IAM Access Analyzer.
- Keep Access Point policies version-controlled.
- Restrict access to specific IAM roles and principals.

---

# 3. Operational Risks

## **OR-1 – Lambda Transformation Failures**
If Lambda times out, crashes, or encounters malformed JSON, the redaction pipeline may break.

### Mitigations
- Add robust exception handling.
- Configure realistic timeouts.
- Enable CloudWatch alarms for errors.
- Add fallback message (“redaction unavailable”) if transformation fails.

---

## **OR-2 – High Request Volume / Scaling Issues**
Sudden spikes in requests may cause latency or throttle limits.

### Mitigations
- Use AWS Lambda’s automatic scaling.
- Monitor concurrency through CloudWatch metrics.
- Consider reserved concurrency for predictable workloads.

---

## **OR-3 – Cost Overruns (Lambda + S3 Calls)**
High access volume may result in unexpected S3 or Lambda costs.

### Mitigations
- Enable AWS Budgets and alerts.
- Optimize Lambda logic for efficiency.
- Compress objects to reduce payload size.

---

# 4. Architectural Risks

## **AR-1 – Lack of Structural Validation**
If the JSON structure changes, redaction may silently fail.

### Mitigations
- Add JSON schema validation.
- Introduce fail-fast behavior for malformed objects.
- Test against multiple input variations.

---

## **AR-2 – Single Point of Redaction Logic**
If the redaction logic is centralized but not versioned properly, changes may break production.

### Mitigations
- Use versioned Lambda deployments.
- Test redaction logic in isolated environments.
- Document expected input/output formats.

---

## **AR-3 – Object Lambda Limitations**
Object Lambda supports specific file types and payload sizes; large objects may fail or generate latency.

### Mitigations
- Validate object sizes before transformation.
- Document known Object Lambda limitations.
- Use S3 Select or ETL pipelines for large-scale transformations.

---

# 5. Observability & Monitoring Risks

## **OM-1 – Undetected Redaction Failures**
If monitoring is inadequate, failures may go unnoticed.

### Mitigations
- Configure CloudWatch alarms on:
  - Lambda errors  
  - Timeouts  
  - Throttles  
  - Non-200 responses from Object Lambda
- Enable CloudTrail data events for S3 Access Point usage.

---

## **OM-2 – Insufficient Audit Logs**
Without proper retention and access controls, audit trails may be incomplete.

### Mitigations
- Enforce log retention policies (90+ days recommended).
- Log redaction events (“SSN field removed”).
- Protect logs with IAM boundaries.

---

# 6. Summary

This architecture introduces a secure, compliant method for delivering sanitized data from Amazon S3 using Object Lambda.  
By addressing risks across IAM, data protection, compliance, and operations—and by implementing the mitigations described above—the solution can meet enterprise-grade governance and regulatory expectations.

