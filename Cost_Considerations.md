# Lessons Learned – AWS S3 Object Lambda PII Redaction

## Purpose

This document captures practical lessons from working through the S3 Object Lambda PII redaction scenario.

The most significant challenge was not the redaction logic itself. It was understanding how identity, permissions, and the S3 Object Lambda request path interact.

---

## 1. IAM Permissions Were the Primary Troubleshooting Challenge

An initial assumption was that the Lambda execution role already had sufficient permissions to retrieve the required S3 object.

Testing showed that the permissions were not sufficient for the intended request path.

The issue was resolved by manually creating and attaching the required IAM policy granting the necessary `s3:GetObject` access.

### Lesson

A Lambda execution role existing does not mean that the function has all permissions required by the workload.

Permissions must be evaluated against:

- The specific AWS API action
- The target resource
- The role performing the action
- S3 and access-point policies involved in the request path

This reinforced the importance of tracing authorization through the complete architecture rather than looking at IAM roles in isolation.

---

## 2. Access Paths Matter as Much as Transformation Logic

The initial focus of the project was PII transformation.

Troubleshooting demonstrated that the larger security problem includes how the function reaches the source object and how consumers reach the transformed representation.

### Lesson

A redaction function is only one part of the security architecture.

The complete path must be considered:

**Consumer → Object Lambda Access Path → Lambda Transformation → Source Object → Transformed Response**

Each interaction introduces authorization and failure considerations.

---

## 3. AWS CLI Responses Were Important Troubleshooting Evidence

Troubleshooting included reviewing responses from AWS CLI operations such as:

- `aws lambda update-function-code`
- `aws s3api get-object`

These responses helped identify where the workflow was failing.

### Lesson

When troubleshooting cloud services, start with the actual API or CLI response rather than assuming the problem is in application logic.

Errors can originate from:

- IAM
- Resource policies
- Service configuration
- Request parameters
- Application logic

Separating these failure domains makes troubleshooting more systematic.

---

## 4. Least Privilege Requires Testing

The permissions issue reinforced an important distinction:

**Least privilege does not mean simply granting fewer permissions.**

It means granting the minimum permissions required for the intended workflow while verifying that unauthorized operations remain unavailable.

### Lesson

IAM design should be tested from both directions:

- Does the intended operation succeed?
- Are unintended operations denied?

A policy should not be considered correct simply because it appears restrictive.

---

## 5. Redaction and Access Control Solve Different Problems

Transformation removes sensitive information from a representation of the data.

IAM determines who can access the underlying resources.

### Lesson

Redaction cannot compensate for an access-control path that allows the same consumer to retrieve the original sensitive object directly.

A production design therefore needs both:

**Controlled access + Controlled transformation**

---

## 6. Failure Behavior Is a Security Decision

Working through the request path highlighted another architectural concern: what should happen when transformation fails?

Returning the original object would preserve availability but could expose sensitive information.

### Lesson

For sensitive-data transformation, failure handling must be explicitly designed.

Where the consumer is not authorized for the original information, the safer architecture is to fail the request rather than silently return the unredacted source object.

---

## 7. Infrastructure Documentation Must Match Implementation

The project documentation originally described Terraform-based deployment.

The current repository does not contain Terraform configuration.

### Lesson

Architecture documentation should distinguish between:

- What was tested manually
- What is part of the proposed architecture
- What is planned
- What is actually implemented in the repository

This distinction improves technical credibility and prevents design intent from being mistaken for implementation evidence.

---

## Key Takeaway

The most important lesson from this project was that protecting sensitive data is not simply a matter of writing redaction logic.

The architecture depends on the interaction between:

**IAM + S3 access paths + Lambda transformation + failure handling + monitoring + governance**

The hands-on permissions troubleshooting reinforced a broader security architecture principle:

**A security control must be evaluated within the complete request and authorization path, not in isolation.**
