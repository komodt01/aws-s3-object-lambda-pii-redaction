# Lessons Learned – AWS S3 Object Lambda PII Redaction

## Purpose

This document captures practical lessons from working through the AWS S3 Object Lambda PII redaction scenario.

The most significant challenge was not the redaction logic itself. It was understanding how identity, permissions, and the S3 Object Lambda request path interact.

---

## 1. IAM Permissions Were the Primary Troubleshooting Challenge

An initial assumption was that the Lambda execution role already had sufficient permissions to retrieve the required S3 object.

Testing showed that the permissions were not sufficient for the intended request path.

The issue was resolved by manually creating and attaching the required IAM policy granting the necessary `s3:GetObject` access.

### Lesson

A Lambda execution role existing does not mean the function has all permissions required by the workload.

Permissions must be evaluated against:

- The specific AWS API action
- The target resource
- The identity performing the action
- Resource policies involved in the request
- The complete S3 Object Lambda access path

This reinforced the importance of tracing authorization through the complete architecture rather than looking at an IAM role in isolation.

---

## 2. Access Paths Matter as Much as Transformation Logic

The initial focus of the project was PII transformation.

Working through the scenario demonstrated that the larger security problem includes how the transformation function reaches the source object and how consumers reach the transformed representation.

### Lesson

A redaction function is only one part of the security architecture.

The complete request path must be considered:

**Consumer → S3 Object Lambda Access Point → Lambda Transformation → S3 Source Object → Transformed Response**

Each interaction introduces authorization, configuration, and failure considerations.

---

## 3. AWS CLI Responses Were Important Troubleshooting Evidence

Troubleshooting included examining responses from AWS CLI operations such as:

- `aws lambda update-function-code`
- `aws s3api get-object`

These responses helped identify where the workflow was failing rather than assuming the problem was within the redaction logic.

### Lesson

Cloud troubleshooting should begin with evidence from the failing operation.

Failures may originate from:

- IAM permissions
- Resource policies
- Service configuration
- Request parameters
- Application logic

Separating these failure domains makes troubleshooting more systematic.

---

## 4. Least Privilege Requires Validation

The permissions issue reinforced an important distinction:

**Least privilege does not simply mean granting fewer permissions.**

It means granting the minimum permissions necessary for the intended workflow while ensuring that unauthorized operations remain unavailable.

### Lesson

IAM design should be validated from both directions:

**Required operation → Allowed**

**Unauthorized operation → Denied**

A policy should not be considered correct simply because it appears restrictive.

---

## 5. Redaction and Access Control Solve Different Problems

Transformation controls which fields are included in the representation returned to a consumer.

IAM and S3 policies control which resources and access paths the consumer can use.

These controls address different risks.

### Lesson

Redaction cannot compensate for an authorization model that allows the same consumer to retrieve the original sensitive object directly.

A production architecture therefore requires both:

**Controlled Access + Controlled Transformation**

The consumer should receive the transformed representation while access controls prevent unauthorized bypass of that transformation.

---

## 6. Failure Behavior Is a Security Decision

Working through the request path highlighted an important architectural question:

**What happens when transformation fails?**

Returning the original object might preserve availability, but it could also expose information that the consumer was never authorized to receive.

### Lesson

Failure handling must be explicitly designed as part of the security architecture.

For consumers authorized only for sanitized information:

**Transformation Failure → Controlled Failure**

should be preferred over:

**Transformation Failure → Return Original Object**

This is an example of applying fail-closed behavior to sensitive-data processing.

---

## 7. Data Classification Must Drive Transformation

Removing an `ssn` field demonstrates the transformation mechanism, but production redaction requirements cannot be defined solely within application code.

The organization must determine which information is sensitive and which consumers require access to specific fields.

### Lesson

There should be a clear separation between:

**Policy – What information is this consumer allowed to receive?**

and

**Enforcement – How does the architecture ensure that only that information is returned?**

Data classification and governance define the policy.

The transformation layer enforces it.

---

## 8. Logging Can Become Another Data-Exposure Path

Transformation processing creates opportunities to log request information, processing details, and errors.

If complete objects or sensitive values are written to logs, the architecture may protect the consumer-facing response while creating another repository containing PII.

### Lesson

Operational visibility must be designed without unnecessarily duplicating sensitive information.

Logs should record useful metadata such as:

**Sensitive field removed**

rather than the sensitive value itself.

---

## 9. Infrastructure Documentation Must Match Implementation

The project documentation originally described Terraform-based deployment.

The current repository does not contain Terraform configuration.

### Lesson

Architecture documentation must distinguish among:

- What was tested manually
- What was actually implemented
- What represents target-state architecture
- What is recommended for production
- What remains future work

Design intent should not be presented as implementation evidence.

Maintaining this distinction makes architecture documentation more credible and easier to evaluate.

---

## 10. The Security Boundary Is Larger Than Lambda

The project began with a relatively narrow technical objective: transform an S3 object and remove sensitive information before returning it.

The work demonstrated that the real security boundary extends well beyond the transformation code.

It includes:

- Identity
- IAM authorization
- S3 access paths
- Object Lambda
- Lambda transformation
- Source-data protection
- Failure handling
- Logging
- Monitoring
- Data classification
- Governance

### Lesson

Security architecture requires evaluating how controls interact across the complete request path.

A component can operate correctly while the overall architecture remains insecure because another path bypasses the control.

---

## Key Takeaway

The most important lesson from this project was that protecting sensitive data is not simply a matter of writing redaction logic.

The effective architecture depends on:

**Classification + Identity + Authorization + Access Path + Transformation + Secure Failure + Monitoring + Governance**

The hands-on IAM troubleshooting reinforced the broader architecture principle:

**A security control must be evaluated within the complete request and authorization path, not in isolation.**
