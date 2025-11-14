# AWS S3 Object Lambda – PII Redaction (Compliance-Focused Architecture)

This project implements a **compliance-driven PII redaction pipeline** using **Amazon S3 Object Lambda**, **AWS Lambda**, and **Terraform**. The system intercepts `GetObject` requests at an S3 Object Lambda Access Point and **removes sensitive data (PII)**—such as Social Security Numbers—before returning the object to the requester.

The design aligns with **GDPR**, **PCI DSS**, **NIST 800-53**, and **HIPAA Security Rule** requirements for data minimization, de-identification, and controlled disclosure.

---

## 🔍 Executive Summary

Organizations often store structured or semi-structured data in S3 that includes Personally Identifiable Information (PII). Granting direct access to these objects increases compliance risk, especially when users only need **partial, sanitized, or de-identified versions**.

This project demonstrates a **policy-enforced, serverless redaction pattern**:

- No duplicate data storage  
- No need for batch ETL pipelines  
- Redaction happens **on demand**, per request  
- PII is **removed** before delivery  
- Least-privilege IAM is enforced  
- Fully deployable via **Terraform**  

---

## 🧩 Architecture Overview

### **Flow Summary**
1. A client issues a `GetObject` request through an **S3 Object Lambda Access Point**.  
2. S3 invokes a **Lambda function** with the object payload.  
3. Lambda loads the original JSON, identifies PII fields, and **removes the `ssn` field**.  
4. The sanitized JSON is returned to S3 Object Lambda.  
5. S3 returns the redacted object to the client.

### **Key Components**
- **S3 Bucket** – Stores original data (with PII).  
- **S3 Standard Access Point** – Controls which clients may access the data.  
- **S3 Object Lambda Access Point** – Intercepts requests for transformation.  
- **Lambda Redaction Function** – Removes PII fields (`ssn`).  
- **IAM Roles & Policies** – Enforce least-privilege access.  
- **Terraform** – Deploys infrastructure consistently and repeatably.  

---

## 🛡 Compliance Motivation

This project demonstrates alignment with:

### **GDPR**
- Data Minimization (Art. 5(1)(c))  
- Purpose Limitation (Art. 5(1)(b))  
- Right to Access (Art. 15) with redaction  
- Pseudonymization & De-identification techniques  

### **PCI DSS 3.2.1**
- Requirement 3.4 – Render sensitive data unreadable  
- Requirement 7 – Restrict access to cardholder data by business need  
- Requirement 10 – Logging and audit trails  

### **HIPAA Security Rule**
- §164.312(a) – Access Control  
- §164.312(c) – Integrity Controls  
- §164.306 – General Data Protection Standards  

### **NIST 800-53 Rev 5**
- AC-3 – Access Enforcement  
- SC-28 – Protection of Information at Rest  
- AU-2 – Audit Events  
- SI-12 – Information Sanitization  

---

## 🧪 Lambda Redaction Logic

This project uses **PII removal**, not masking.

Example input (from `test-event.json`):
```json
{
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "ssn": "123-45-6789"
}

