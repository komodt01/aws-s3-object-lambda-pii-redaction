# Technologies – AWS S3 Object Lambda PII Redaction

This document explains the AWS services and core technologies used in the S3 Object Lambda PII Redaction project.  
Each section includes:

1. **What the technology is**  
2. **Why it is used**  
3. **How it works in this specific project**

---

# 1. Amazon S3

## **What It Is**
Amazon Simple Storage Service (S3) is AWS’s object storage platform designed for high durability, availability, and scalability. S3 stores data as objects inside buckets and supports fine-grained access controls.

## **Why We Use It**
S3 serves as the authoritative data store for raw JSON objects that may contain sensitive information such as SSNs. It provides:

- Durable storage of original (unredacted) data  
- Server-side encryption  
- Native integration with Access Points and Object Lambda  
- Low operational overhead  

## **How It Works Here**
- The project stores raw JSON objects containing PII directly in an S3 bucket.
- Users are **not allowed** to access the bucket directly.
- All sanitized data flows through **Object Lambda**, not through the bucket itself.
- Original objects remain unchanged, preserving data integrity for internal users.

---

# 2. S3 Access Points

## **What They Are**
S3 Access Points provide unique hostnames and IAM-controlled access policies for specific datasets within an S3 bucket. They simplify permissions for multi-tenant systems or complex data access patterns.

## **Why We Use Them**
Access Points allow us to:

- Restrict who can access data  
- Gate access through identity-based policies  
- Separate raw-data access from sanitized-data access  

## **How They Work in This Project**
We define **two** distinct access paths:

### **1. Standard Access Point**
- Used internally (optional)
- Provides direct, unmodified access to raw data
- Locked down by IAM to only privileged roles

### **2. Object Lambda Access Point**
- Intercepts every read request
- Routes the request to Lambda for transformation
- Returns sanitized JSON with PII fields removed

This dual-path access model supports strong separation between **internal privileged access** and **external sanitized access**.

---

# 3. S3 Object Lambda

## **What It Is**
S3 Object Lambda allows you to intercept `GetObject` requests and apply custom transformations using AWS Lambda before returning the object to the client.

## **Why We Use It**
Object Lambda lets us:

- Remove or transform sensitive fields *on demand*  
- Avoid maintaining separate PII and non-PII datasets  
- Introduce a compliance layer without redesigning storage  
- Apply redaction logic transparently to downstream systems  

## **How It Works in This Project**
1. A client requests an object using the **Object Lambda Access Point**.  
2. S3 retrieves the original object.  
3. S3 passes the object to a **Lambda function** for transformation.  
4. Lambda loads the JSON and removes the `"ssn"` field.  
5. S3 returns the **sanitized object** to the requester.

This creates a **just-in-time redaction pipeline**, aligned with GDPR, HIPAA, PCI, and NIST.

---

# 4. AWS Lambda

## **What It Is**
AWS Lambda is a serverless compute service that runs stateless code in response to events. Lambda automatically scales based on demand.

## **Why We Use It**
Lambda provides:

- A fully serverless transformation engine  
- Zero infrastructure to manage  
- Millisecond-level execution  
- Built-in CloudWatch logging  
- Native integration with S3 Object Lambda  

## **How It Works Here**
- The Lambda function loads the raw JSON from S3.
- Checks for a `"ssn"` field.
- Logs the redaction event (without logging the PII value).
- Removes the sensitive field from the object.
- Returns the sanitized JSON back to Object Lambda.

The Lambda execution role is restricted to:

- `s3:GetObject` on the specific bucket  
- CloudWatch logging permissions  

No broader S3 or AWS permissions are granted.

---

# 5. Terraform

## **What It Is**
Terraform is an Infrastructure-as-Code (IaC) tool that provisions cloud resources declaratively.

## **Why We Use It**
Terraform ensures:

- **Repeatability** (same configuration every time)  
- **Consistency** across dev/test/prod patterns  
- **Auditability** (changes tracked in version control)  
- **Rapid provisioning** of S3, Access Points, Lambda, IAM  

## **How It Works Here**
Terraform defines:

- S3 bucket and permissions  
- Standard Access Point  
- Object Lambda Access Point  
- IAM role and execution policy for Lambda  
- Lambda function configuration  
- IAM trust policy  
- Output variables for simplified testing  

This allows the entire redaction pipeline to be deployed and destroyed safely.

---

# 6. JSON Transformation Logic

## **What It Is**
JSON is a structured text format used for representing key/value data. Lambda loads JSON into Python dictionaries for manipulation.

## **Why We Use It**
Most PII-containing event data in enterprise environments is JSON-based. It’s easy for Lambda to:

- load  
- inspect  
- modify  
- return  

…which makes JSON ideal for transformation workflows.

## **How It Works Here**
The Lambda code performs:

```python
original_json = json.loads(original_data)

if "ssn" in original_json:
    del original_json["ssn"]

7. CloudWatch Logs
What It Is
CloudWatch Logs captures operational logs from AWS services.

Why We Use It
Auditability and traceability are compliance requirements. Logs allow:
Visibility into Lambda redaction actions
Monitoring error conditions
Debugging malformed input

How It Works Here
Lambda logs:
“Invoking transformation function”
“Removing SSN field”

Exceptions
Transformation failures
No raw PII values are logged, maintaining compliance.

8. IAM (Identity and Access Management)
What It Is
AWS IAM manages identities and permissions for AWS services.

Why We Use It
PII redaction requires strict access control:
Only the Lambda role should read raw data
Users should only retrieve sanitized objects
Access policies must enforce least privilege

How It Works Here
IAM enforces:
Lambda execution roles
Bucket policies denying direct access to raw data
Access Point policies limiting who may use the endpoint

Access through Object Lambda → YES
Direct access to bucket → NO
This aligns with NIST AC-3 (“Access Enforcement”).

9. Optional: KMS (If Enabled)
What It Is
AWS Key Management Service manages encryption keys.

Why We Use It
When storing sensitive data, per-object encryption or bucket-level encryption may be required.

How It Works Here
If SSE-KMS is enabled:
Lambda must be granted kms:Decrypt
Roles must be scoped to only the relevant CMK
