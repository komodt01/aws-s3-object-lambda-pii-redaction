# AWS S3 Object Lambda Redaction Project

## Overview
This project uses AWS S3 Object Lambda to redact sensitive information from S3 objects at retrieval time, without modifying the underlying object. It uses an AWS Lambda function triggered via an Object Lambda Access Point to inspect and redact data before returning it to the requester.

## Business Use Case
This setup is useful for organizations needing to dynamically filter or redact sensitive data (e.g., PII) from files stored in S3 when accessed, complying with data protection policies without duplicating or altering source files.

## Architecture Flow
1. User requests a file through an S3 Object Lambda Access Point.
2. AWS triggers the Lambda function attached to the access point.
3. The Lambda reads the S3 object, processes/redacts the content, and returns the redacted version to the user.

See the included PNG diagram for visual reference.

## Technologies Used
- AWS Lambda
- AWS S3 Object Lambda
- IAM (Inline policies for Lambda permissions)
- AWS CLI
