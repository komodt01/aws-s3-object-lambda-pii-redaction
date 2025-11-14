# Requirements – AWS S3 Object Lambda PII Redaction

This document defines the **business**, **functional**, **security**, **compliance**, and **non-functional** requirements for the AWS S3 Object Lambda PII Redaction solution.  
The project implements on-demand PII removal for structured JSON data stored in Amazon S3, enforced through a serverless transformation pipeline.

---

## 1. Business Requirements

### **BR1 – Controlled PII Disclosure**
The system must prevent exposure of sensitive data fields (e.g., SSN) to users who do not require full visibility of the original data object.

### **BR2 – No Data Duplication**
The organization must avoid duplicating S3 objects to maintain sanitized and unsanitized versions. All redaction must occur dynamically.

### **BR3 – Low Operational Overhead**
The solution must operate without dedicated servers, databases, scheduled jobs, ETL pipelines, or additional manual intervention.

### **BR4 – Auditable and Explainable**
Redaction decisions must be fully observable through logs to support compliance audits and investigations.

### **BR5 – On-Demand Access**
Clients must retrieve sanitized data via a standard S3 `GetObject` workflow with minimal change to their existing integration.

---

## 2. Functional Requirements

### **FR1 – PII Field Removal**
The Lambda function must identify PII fields and *remove* them before returning the response through S3 Object Lambda. Initial scope includes:

- `ssn` (Social Security Number)

### **FR2 – JSON-based Transformation**
The transformation function must support structured objects in JSON format.

### **FR3 – No Modification of Original Data**
Original data stored in the S3 bucket must remain unchanged.

### **FR4 – S3 Object Lambda Invocation**
All transformations must be triggered transparently when a user retrieves an object from the Object Lambda Access Point.

### **FR5 – Error Handling**
If the Lambda function fails, S3 must return client-visible error messages with CloudWatch logs for troubleshooting.

### **FR6 – IAM-Based Access Control**
Access must be restricted through IAM policies and S3 Access Point policies.

### **FR7 – Terraform Deployment**
Infrastructure (S3, Access Points, Lambda, IAM) must be deployable using Terraform for repeatability.

---

## 3. Security Requirements

### **SR1 – PII Sanitization**
PII fields must be permanently removed from the response payload before returning to the user.

### **SR2 – IAM Least Privilege**
IAM roles for Lambda and Access Points must grant only the minimum required permissions to perform:

- S3 GetObject  
- PutObject (if logs or artifacts were implemented)  
- CloudWatch logging  
- No administrative or wildcard permissions

### **SR3 – S3 Access Path Enforcement**
Users must not have direct access to the underlying S3 bucket; all GetObject requests must flow through the Object Lambda Access Point.

### **SR4 – Encrypted Data Paths**
Traffic between S3, Object Lambda, and Lambda must be encrypted (TLS in transit, SSE-S3 or SSE-KMS at rest).

### **SR5 – Lambda Hardening**
The Lambda execution environment must follow AWS security best practices:

- No inline secrets  
- Logging enabled  
- Limited network exposure  
- Managed runtime  

### **SR6 – Logging/Auditability**
Lambda must log:

- Invocation events  
- PII removal actions  
- Errors and exceptions  

Logs must be written to CloudWatch.

---

## 4. Compliance Requirements

### **GDPR**
- **Art. 5(1)(c)** – Data Minimization  
- **Art. 5(1)(b)** – Purpose Limitation  
- **Art. 15** – Subject Access Requests with redaction  
- **Art. 25** – Data Protection by Design  

### **PCI DSS 3.2.1**
- **Req. 3.4** – Render sensitive data unreadable  
- **Req. 7** – Restrict access by business need-to-know  
- **Req. 10** – Track access and audit logs  

### **HIPAA Security Rule**
- **164.312(a)** – Access Controls  
- **164.312(c)** – Integrity Controls  
- **164.306** – General Security Requirements  

### **NIST SP 800-53 Rev 5**
- **AC-3** – Access Enforcement  
- **SC-28** – Protect Information at Rest  
- **SI-12** – Information Sanitization  
- **AU-2** – Audit Events  

---

## 5. Non-Functional Requirements

### **NFR1 – Scalability**
The system must scale automatically with the volume of requests, leveraging the serverless nature of S3 and Lambda.

### **NFR2 – Performance**
Transformations should complete within the default Lambda timeout and maintain low latency (single-digit milliseconds to low hundreds of milliseconds).

### **NFR3 – High Availability**
AWS-managed components (S3, Lambda) must provide high resiliency with no single points of failure.

### **NFR4 – Cost Efficiency**
The architecture must incur charges only for:

- Lambda invocations  
- S3 requests  
- CloudWatch logs  

No long-running compute resources are allowed.

### **NFR5 – Observability**
CloudWatch metrics and logs must be used to monitor:

- Redaction events  
- Invocation errors  
- Latency  
- Request counts  

---

## 6. Success Criteria

- All `ssn` fields are removed from responses.  
- Users cannot retrieve raw, unredacted S3 objects directly.  
- Infrastructure deploys cleanly using Terraform.  
- Lambda logs show clear redaction actions.  
- Access paths satisfy least privilege.  
- The design meets GDPR, PCI, HIPAA, and NIST minimization guidelines.

