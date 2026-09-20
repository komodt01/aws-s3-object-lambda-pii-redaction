# Executive Case Study – Request-Time PII Redaction Architecture

## Executive Summary

Organizations frequently need to share information from centralized datasets with applications, analytics teams, business units, and third parties that do not require access to every field in the underlying record.

Providing complete records increases sensitive-data exposure.

Maintaining separate sanitized copies can introduce additional storage, synchronization, lifecycle, and governance complexity.

This project evaluates an AWS architecture using S3 Object Lambda and AWS Lambda to create a controlled transformation layer between sensitive source data and consumers.

The architectural objective is straightforward:

**Allow consumers to receive the information they need without automatically exposing all information stored in the authoritative source.**

---

## Business Challenge

A single S3 dataset may support consumers with significantly different information requirements.

For example, a source record could contain:

- Customer identifiers
- Contact information
- Transaction information
- Internal metadata
- Social Security numbers
- Other sensitive attributes

An analytics process may require transaction information while having no legitimate need for an SSN.

Traditional approaches may provide access to the entire record or require creation of a separate sanitized dataset.

Both approaches introduce risk or operational complexity.

---

## Architecture Decision

The proposed architecture introduces request-time transformation between the consumer and the authoritative S3 object.

Conceptually:

**Consumer → S3 Object Lambda Access Point → Lambda Transformation → S3 Source Object → Sanitized Response**

The source object remains unchanged.

The transformation layer removes fields that the consumer is not authorized or required to receive before returning the representation.

The project's initial transformation example removes the `ssn` field from structured JSON data.

---

## Security Strategy

The architecture applies several complementary security principles.

### Data Minimization

Consumers should receive only information required for their approved business purpose.

### Least Privilege

IAM permissions should limit consumers and services to required resources and operations.

### Controlled Disclosure

Sensitive fields can be removed before information is returned to consumers that do not require them.

### Source Preservation

The authoritative source record remains unchanged by the transformation process.

### Controlled Access Paths

Consumers authorized only for sanitized data must not be able to bypass the transformation path and directly retrieve sensitive source objects.

### Secure Failure

Transformation failures should not result in unauthorized disclosure of the original information.

---

## Key Architecture Insight

The most important security decision is not the Lambda redaction function itself.

A perfectly functioning redaction function provides little protection if the same consumer can retrieve the original S3 object through another access path.

The effective security boundary therefore includes:

**Identity + Authorization + Access Path + Transformation + Monitoring + Governance**

This changes the architecture discussion from:

**“Can Lambda remove an SSN?”**

to:

**“How do we ensure that this consumer can receive only the information they are authorized to receive?”**

---

## Risk Considerations

The architecture evaluates risks including:

- Transformation-path bypass
- Excessive IAM permissions
- Incomplete redaction
- Schema changes
- Incorrect data classification
- Sensitive information appearing in logs
- Transformation failures
- Availability dependencies
- Performance impact
- Request-driven cost growth
- Insufficient monitoring

These risks demonstrate why data protection cannot depend on transformation logic alone.

---

## Governance Model

The architecture separates policy decisions from technical enforcement.

Data owners and governance functions should determine:

- Which information is sensitive
- Which consumers require specific fields
- Which fields must be removed
- Which transformation method is appropriate
- Who may access the original information
- How exceptions are approved

AWS services then provide mechanisms for enforcing those decisions.

This prevents application code from becoming the organization's unofficial data-classification policy.

---

## Compliance Relevance

The architecture can support security and privacy objectives associated with frameworks such as:

- NIST SP 800-53
- ISO/IEC 27001
- GDPR
- HIPAA
- PCI DSS

Relevant control themes include:

- Data minimization
- Least privilege
- Access enforcement
- Information protection
- Controlled disclosure
- Logging
- Monitoring
- Security by design

The architecture does not establish regulatory compliance by itself.

Compliance depends on the complete implementation, applicable regulatory scope, organizational processes, testing, and demonstrated control effectiveness.

---

## Cost and Operational Tradeoff

Request-time transformation can reduce the need to maintain separate sanitized datasets.

Potential benefits include:

- Reduced duplicate storage
- Reduced synchronization requirements
- Centralized transformation logic
- Transformation only when information is requested

The architecture also introduces:

- Additional request-path processing
- Transformation latency
- Additional monitoring requirements
- Lambda and service usage costs
- Availability dependencies

The appropriate architecture therefore depends on workload characteristics rather than assuming request-time transformation is always preferable.

---

## Hands-On Learning

Practical work with the architecture exposed an IAM permissions issue during the S3 Object Lambda workflow.

Troubleshooting required examining AWS CLI responses and adjusting permissions required for object retrieval.

This reinforced an important architecture principle:

**Security controls must be evaluated across the complete request and authorization path rather than as isolated components.**

The repository distinguishes this hands-on troubleshooting from broader target-state architecture requirements that have not been fully implemented.

---

## Business Outcome

The architecture demonstrates a reusable data-protection pattern for organizations that need to provide different consumers with different representations of centralized sensitive information.

Rather than treating every authorized user as authorized for every field, the design introduces a policy-enforcement point between stored information and its consumer.

The resulting architectural principle is:

**Store the authoritative information securely, control who can reach it, and disclose only what each consumer is authorized and required to receive.**
