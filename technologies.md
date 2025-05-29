# Technologies Used

## AWS Lambda
Used to run a Python function to inspect and redact S3 content dynamically.

## S3 Object Lambda
Provides the interface for intercepting GetObject requests and triggering a Lambda function to transform the data.

## IAM
An inline policy was added to the Lambda execution role to allow access to required S3 resources.

## AWS CLI
Used to deploy and test the project through terminal commands.
