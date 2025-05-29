# Teardown Instructions

## Manual Steps
1. Delete the S3 Object Lambda Access Point:
```bash
aws s3control delete-access-point-for-object-lambda --account-id <your-account-id> --name secure-object-lambda-ap
```

2. Delete the Lambda function:
```bash
aws lambda delete-function --function-name s3-object-lambda-redactor
```

3. Delete any related inline IAM policies.
