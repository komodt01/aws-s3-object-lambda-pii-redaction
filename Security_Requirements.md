# Security Requirements – AWS S3 Object Lambda PII Redaction

This document defines the security requirements for the AWS S3 Object Lambda PII Redaction solution. The goal is to ensure that sensitive data is protected, access is controlled, and the architecture aligns with security best practices and relevant compliance expectations.

---

## 1. Security Objectives

- **SO1 – Prevent Unauthorized Exposure of PII**  
  Ensure that sensitive data (e.g., SSN) is not exposed to users who do not require full access.

- **SO2 – Enforce Data Minimization**  
  Only provide the minimum necessary data to the requester, removing PII fields from responses.

- **SO3 – Maintain Integrity of Original Data**  
  Ensure that original S3 objects remain unchanged, with redaction applied only to responses.

- **SO4 – Enable Audit and Traceability**  
  Provide sufficient logging to trace redaction operations and access patterns.

---

## 2. Identity & Access Management (IAM)

### **IAM-1 – Least Privilege for Lambda Role**
The Lambda execution role must grant only the minimal permissions required:

- `s3:GetObject` on the source bucket (scoped to relevant prefixes/objects where possible)
- `logs:CreateLogGroup`, `logs:CreateLogStream`, and `logs:PutLogEvents` for CloudWatch logging

No wildcard (`"*"` on `"*"`) administrative permissions are allowed.

### **IAM-2 – Restricted S3 Access for Callers**
Callers must **not** have direct access to the underlying S3 bucket. They should only access data through:

- S3 Access Point and  
- S3 Object Lambda Access Point

Bucket policies and IAM policies must enforce this restriction.

### **IAM-3 – Scoped Access Point Policies**
Both the standard Access Point and Object Lambda Access Point must:

- Allow access only from approved IAM principals or roles
- Restrict access to intended prefixes/objects
- Deny direct bucket-level access where appropriate

### **IAM-4 – No Inline Secrets**
The Lambda function code must not embed credentials, secrets, or API keys. All authentication must rely on the Lambda execution role.

---

## 3. Data Protection

### **DP-1 – PII Removal in Responses**
The redaction function must remove PII fields (e.g., `ssn`) from the response payload before returning it to the client. Original objects in S3 remain unchanged.

### **DP-2 – Encryption at Rest**
The S3 bucket must use server-side encryption:

- SSE-S3 or SSE-KMS

If KMS is used, the IAM role must be granted only necessary KMS permissions.

### **DP-3 – Encryption in Transit**
All communication:

- Between clients and S3  
- Between S3, Object Lambda, and Lambda  

must use TLS (HTTPS). Access over HTTP must be blocked or redirected to HTTPS.

### **DP-4 – No Local Persistence**
The Lambda function must not persist PII to temporary external storage (e.g., RDS, DynamoDB, SQS, external services) as part of the transformation.

---

## 4. Logging & Monitoring

### **LM-1 – CloudWatch Logging**
Lambda must log:

- Invocation events and request context (non-sensitive metadata)
- Redaction actions (e.g., detection and removal of `ssn` field)
- Errors and exceptions

Logs must not contain raw PII values, only metadata indicating that redaction occurred.

### **LM-2 – Access Logging**
Where possible, S3 access logging (or CloudTrail data events) should be enabled to record who accessed which objects via the Object Lambda Access Point.

### **LM-3 – Alerting (Optional Enhancement)**
Operational teams should be able to configure alerts for:

- High failure rates of the Lambda function
- Unusual spikes in access volume
- Repeated errors when processing specific objects

---

## 5. Application Security

### **AS-1 – Input Validation**
The Lambda function must:

- Safely parse JSON input
- Handle malformed or unexpected payloads without crashing
- Fail securely when the structure does not match expected JSON

### **AS-2 – Defensive Coding**
The function must:

- Catch and log exceptions
- Avoid exposing internal stack traces to clients
- Return clear but non-sensitive error messages

### **AS-3 – Minimal Attack Surface**
The Lambda function should:

- Use a minimal set of dependencies
- Avoid unnecessary network access
- Avoid calling external, non-AWS third-party endpoints as part of the transformation

---

## 6. Infrastructure & Deployment

### **INF-1 – Immutable Infrastructure via Terraform**
Infrastructure changes must be applied via Terraform to reduce configuration drift and manual misconfigurations.

### **INF-2 – Version Control**
Terraform configuration and Lambda source code must be stored in version control (e.g., GitHub) with proper commit history.

### **INF-3 – Safe Rollback**
Deployment procedures should support:

- Rolling back to a previous Lambda version
- Reverting Terraform changes when necessary (e.g., via `terraform destroy` / `terraform apply` with previous plan)

---

## 7. Compliance Alignment (High-Level)

These security requirements support alignment with:

- **GDPR** – Data minimization, protection by design, and controlled disclosure of personal data.
- **PCI DSS** – Rendering sensitive data unreadable and restricting access by business need-to-know.
- **HIPAA** – Technical safeguards for access control, integrity, and audit logging.
- **NIST 800-53** – Controls for access enforcement, information sanitization, and auditing.

This solution is not a complete compliance program but demonstrates a compliant *pattern* for on-demand redaction of sensitive data in S3.

