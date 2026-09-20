# Technical Case Study – AWS S3 Object Lambda PII Redaction Architecture

## Technical Objective

The objective of this project was to evaluate an AWS architecture for dynamically removing sensitive fields from structured data before that data is returned to a consumer.

The design uses Amazon S3 as the authoritative data store and S3 Object Lambda with AWS Lambda as a request-time transformation layer.

The initial transformation example removes an `ssn` field from JSON while preserving the original S3 object.

---

## Architecture Pattern

The conceptual request path is:

**Consumer → S3 Object Lambda Access Point → Lambda Transformation → S3 Source Object → Sanitized Response**

The architecture separates three concerns:

1. Storage of the authoritative object
2. Authorization to access the data
3. Transformation of the representation returned to the consumer

This separation is important because transformation alone does not prevent access to the original data.

---

## Source Data

Amazon S3 represents the authoritative storage layer.

A source JSON object may contain both ordinary business information and sensitive fields.

Conceptually, a record might contain:

`name`, `email`, `account information`, and `ssn`

The source object remains unchanged during request-time transformation.

This allows the organization to preserve the authoritative record while providing a reduced representation to consumers with different information requirements.

---

## S3 Object Lambda

S3 Object Lambda provides the architectural interception point for transforming an object as part of the retrieval workflow.

Instead of requiring every downstream consumer to implement its own redaction logic, transformation can occur within a controlled data-access path.

The intended sequence is:

1. A consumer requests an object through the approved Object Lambda access path.
2. The transformation workflow obtains the source information.
3. Lambda processes the structured data.
4. Defined sensitive fields are removed.
5. The transformed representation is returned to the consumer.
6. The original S3 object remains unchanged.

---

## Lambda Transformation

AWS Lambda provides the transformation logic.

The project's initial transformation rule is intentionally simple:

**If the JSON object contains an `ssn` field, remove that field from the consumer-facing representation.**

This demonstrates the architectural pattern without treating the example logic as a complete enterprise data-classification engine.

A production implementation would need to account for:

- Nested fields
- Arrays
- Schema changes
- Multiple sensitive-data classifications
- Malformed input
- Unsupported content
- Large objects
- Transformation failures
- Versioned transformation policies

---

## Identity and Authorization

IAM is a critical component of the architecture.

Two distinct authorization questions must be answered:

**Can the consumer access the transformed representation?**

and

**Can the consumer access the original source object?**

A consumer authorized only for sanitized information should not also receive permissions that allow retrieval of the sensitive source object through another path.

Production access controls may involve:

- IAM policies
- S3 bucket policies
- S3 Access Point policies
- Object Lambda permissions
- Lambda execution-role permissions
- KMS permissions when applicable

---

## Hands-On IAM Troubleshooting

Practical work on the project exposed a permissions issue during object retrieval.

The initial assumption was that the Lambda execution role already had sufficient permissions.

Testing showed that additional `s3:GetObject` permission was required for the intended workflow.

Troubleshooting included reviewing AWS CLI responses associated with operations such as:

- `aws lambda update-function-code`
- `aws s3api get-object`

The permissions issue was resolved by manually creating and attaching the required IAM policy.

This reinforced the need to trace authorization through the complete request path rather than assuming that the existence of an execution role means access is correctly configured.

---

## Least-Privilege Design

The Lambda execution role should receive only permissions required by the transformation workflow.

Depending on the final implementation, these may include permissions for:

- Required S3 objects
- Required access paths
- CloudWatch logging
- KMS operations when applicable

Permissions should be scoped to specific resources wherever practical.

Least privilege must also be validated through testing.

The architecture should verify both:

**Required operation → Allowed**

and

**Unauthorized operation → Denied**

---

## Transformation-Path Bypass

One of the most important technical risks is bypass of the transformation layer.

If a consumer can retrieve the original S3 object directly, the redaction control can be bypassed regardless of how well the Lambda function operates.

A production implementation must therefore enforce separation between:

**Privileged Raw-Data Path**

and

**Transformed Consumer Path**

IAM and resource policies must be designed and tested together to enforce that separation.

---

## Secure Failure Behavior

Transformation failure represents a confidentiality concern as well as an availability concern.

Examples include:

- Lambda exceptions
- Malformed JSON
- Unexpected schemas
- Timeouts
- Permission failures
- Unsupported content

The architecture should not respond to transformation failure by returning the original sensitive object to a consumer that is not authorized to receive it.

The preferred security behavior is:

**Transformation failure → Controlled failure response**

rather than:

**Transformation failure → Return original object**

This is a fail-closed design principle.

---

## Logging and Observability

A production implementation should provide visibility into:

- Transformation invocations
- Successful transformations
- Transformation failures
- Processing latency
- Authorization failures
- Unexpected source-data access

Logging must avoid creating another sensitive-data repository.

For example, a log may record:

`Sensitive field removed`

It should not record the actual sensitive value.

Monitoring should focus on both operational health and potential control failures.

---

## Encryption

Sensitive source objects should use appropriate encryption at rest.

Potential S3 options include:

- SSE-S3
- SSE-KMS

SSE-KMS may provide additional key-management and authorization capabilities where organizational requirements justify them.

Data in transit should use HTTPS/TLS.

Encryption and redaction address different threats.

Encryption protects information while stored or transmitted.

Redaction controls which information is disclosed to the consumer.

Both may be required.

---

## Data Classification

The Lambda function should not independently define organizational data policy.

A production implementation requires an approved classification process identifying:

- Sensitive fields
- Consumer populations
- Permitted disclosures
- Required transformation methods
- Exceptions

The transformation code then enforces those decisions.

This separates:

**Security Policy**

from

**Technical Enforcement**

---

## Testing Strategy

A production implementation should test more than the successful redaction scenario.

### Functional Testing

Verify that defined sensitive fields are removed from expected input.

### Negative Testing

Test:

- Missing sensitive fields
- Additional fields
- Malformed JSON
- Nested data
- Unexpected schemas
- Unsupported content

### Authorization Testing

Verify that:

- Approved transformed access succeeds.
- Unauthorized raw-data access fails.
- Privileged access behaves according to policy.

### Failure Testing

Simulate:

- Lambda errors
- Permission failures
- Invalid input
- Processing failures

Confirm that sensitive source data is not returned when transformation fails.

### Logging Testing

Verify that sufficient operational evidence is generated without recording raw sensitive values.

---

## Infrastructure as Code

Infrastructure as Code would be appropriate for repeatable deployment of components such as:

- S3 resources
- Access points
- Lambda
- IAM policies
- Logging
- Encryption configuration

Terraform is one potential implementation approach.

However, Terraform configuration is not currently present in this repository.

Terraform should therefore be treated as a future implementation option rather than a completed project capability.

---

## Performance Considerations

Request-time transformation adds processing to the object-retrieval path.

Performance should be evaluated based on:

- Object size
- Transformation complexity
- Lambda execution duration
- Request volume
- Concurrency
- Consumer latency requirements

No fixed latency target should be assumed without workload testing.

---

## Architecture Tradeoff

Request-time transformation provides centralized control without requiring a permanently stored sanitized copy of every object.

The tradeoff is that transformation becomes part of the live request path.

This creates dependencies involving:

- Availability
- Latency
- Cost
- IAM
- Monitoring
- Failure handling

For some workloads, pre-generated sanitized datasets or other transformation mechanisms may be more appropriate.

The architecture decision should therefore be based on workload requirements rather than the availability of a particular AWS service.

---

## Implementation Scope

This repository demonstrates:

- The architecture pattern
- PII redaction logic
- Security requirements
- IAM and access-path considerations
- Risk analysis
- Compliance considerations
- Cost tradeoffs
- Hands-on IAM troubleshooting lessons

The repository does not currently demonstrate:

- Complete Terraform deployment
- Production monitoring configuration
- Full enterprise data-classification integration
- Production-scale performance testing
- Complete automated deployment

This distinction separates implemented and explored work from target-state architecture.

---

## Technical Outcome

The project began with a relatively simple technical objective:

**Remove an `ssn` field before returning an S3 object.**

The deeper architectural problem proved to be significantly broader.

Protecting the information requires coordinating:

**S3 + Object Lambda + Lambda + IAM + Encryption + Failure Handling + Monitoring + Governance**

The central technical lesson is:

**Transformation is effective only when the surrounding authorization architecture prevents consumers from bypassing it.**

That makes access-path design and IAM enforcement as important as the redaction logic itself.
