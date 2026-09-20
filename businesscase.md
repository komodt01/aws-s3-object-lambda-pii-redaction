# Business Case – AWS S3 Object Lambda PII Redaction

## Business Problem

Organizations frequently store datasets containing Personally Identifiable Information (PII) in Amazon S3.

The same dataset may need to support multiple consumers with different access requirements. An internal privileged user may require access to the complete record, while an analytics team, application, contractor, or third party may only require a subset of the information.

Providing every consumer with access to the complete source object increases the risk of unnecessary sensitive-data exposure.

Creating and maintaining separate sanitized copies introduces additional challenges:

- Duplicate data
- Additional storage
- Synchronization requirements
- Data lifecycle complexity
- Potential inconsistencies between source and sanitized datasets
- Additional processes for maintaining redaction logic

The business requirement is therefore to provide appropriate access to useful data while minimizing unnecessary disclosure of sensitive information.

---

## Proposed Architecture

The architecture uses Amazon S3 Object Lambda and AWS Lambda as a request-time transformation layer.

Conceptually:

**Consumer → S3 Object Lambda → Lambda Transformation → S3 Source Object → Sanitized Response**

The original object remains the authoritative data source.

When data is requested through the controlled transformation path, sensitive fields can be removed before the resulting representation is returned to the consumer.

This allows the organization to preserve the source dataset while presenting a reduced-data view for consumers that do not require access to all fields.

---

## Business Value

### Reduced Sensitive-Data Exposure

Consumers can receive only the information required for their approved business purpose.

This supports the principle of data minimization and reduces unnecessary exposure of sensitive information.

### Reduced Data Duplication

Request-time transformation can reduce the need to maintain separate sanitized copies of the same source dataset.

### Preservation of the Authoritative Dataset

The original object remains unchanged.

Transformation occurs when the data is requested rather than permanently modifying the source record.

### Centralized Transformation Logic

Redaction logic can be placed within a defined transformation layer rather than requiring every downstream consumer to implement its own sanitization process.

### Flexible Data-Sharing Patterns

The architecture can support scenarios where different consumers require different representations of the same underlying information.

---

## Example Use Cases

Potential business scenarios include:

- Financial-services teams sharing records without unnecessary customer identifiers
- Healthcare analytics using reduced datasets that exclude sensitive identifiers
- Development and testing teams receiving sanitized production-derived data
- Third parties receiving only fields required for an approved business process
- Internal analytics teams accessing business information without unnecessary PII
- Public-sector datasets excluding citizen identifiers before disclosure

These are representative use cases for the architecture pattern rather than implemented workloads in this repository.

---

## Security Value

The architecture supports several security principles.

### Data Minimization

Only information required for the approved purpose should be disclosed.

### Least Privilege

Authorization should restrict access to both the transformation path and the underlying source data according to business need.

### Controlled Disclosure

Sensitive fields can be removed before information reaches consumers that do not require them.

### Defense in Depth

Request-time transformation complements other controls such as:

- IAM authorization
- S3 policies
- Encryption
- Logging
- Monitoring
- Data classification
- Security governance

Redaction should not be treated as a replacement for these controls.

---

## Compliance Relevance

The architecture can support organizational controls associated with:

- Data minimization
- Access enforcement
- Least privilege
- Controlled disclosure
- Information protection
- Auditability
- Information sanitization

These concepts are relevant to frameworks and regulatory requirements such as GDPR, HIPAA, PCI DSS, ISO 27001, and NIST SP 800-53.

The architecture should not be interpreted as establishing compliance with any framework by itself.

Compliance depends on the complete implementation, organizational processes, data involved, control effectiveness, and applicable regulatory requirements.

---

## Business Tradeoffs

### Request-Time Transformation vs. Sanitized Copies

Request-time transformation can reduce data duplication and synchronization requirements.

However, transformation becomes part of the retrieval path and introduces additional processing, availability, permissions, monitoring, and operational considerations.

### Centralized Transformation vs. Consumer Flexibility

Centralizing transformation can improve consistency, but different consumers may require different data views.

The organization therefore needs governance around which fields are disclosed to which consumers.

### Automation vs. Policy Governance

Lambda can automate transformation, but the code should not independently determine organizational data policy.

Data owners and security governance processes should determine:

- Which data is sensitive
- Which consumers require access
- Which fields may be disclosed
- Which transformation method is appropriate
- How exceptions are approved

The technology should enforce those decisions.

---

## Risk Considerations

The architecture must account for several business and security risks:

- Consumers bypassing the transformation path
- Incorrect classification of sensitive fields
- Transformation failures exposing original information
- Excessive IAM permissions
- Sensitive information appearing in logs
- Changes to source-data schemas
- Incorrect transformation rules
- Insufficient monitoring
- Performance or availability issues in the transformation path

These risks require technical controls as well as governance and operational procedures.

---

## Success Criteria

A production implementation should be evaluated on whether it can:

- Prevent unauthorized access to sensitive source data
- Provide consumers with only the information required for their business purpose
- Preserve the authoritative source object
- Apply transformation consistently
- Fail securely when transformation cannot be completed
- Provide sufficient logging and monitoring without exposing PII
- Support manageable access and transformation policies
- Meet required performance and availability expectations

---

## Strategic Value

The primary value of this architecture is not simply hiding an `ssn` field.

It demonstrates a broader data-security pattern:

**Separate the data an organization stores from the data a particular consumer is authorized to receive.**

That distinction becomes increasingly important as organizations reuse centralized datasets across applications, analytics platforms, partners, development environments, and other consumers.

Request-time transformation provides one architectural option for enforcing that separation while preserving the authoritative source dataset.
