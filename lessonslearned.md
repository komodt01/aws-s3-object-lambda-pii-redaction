# Lessons Learned

## Misassumptions
- Initially believed the Lambda had sufficient permissions from the execution role alone.

## Resolution
- Discovered that an inline IAM policy had to be manually created and attached to grant explicit `s3:GetObject` access to the Object Lambda Access Point.
- Debugging required inspecting `aws lambda update-function-code` and `aws s3api get-object` responses carefully.

## Time Consumption
- The longest time was spent troubleshooting permissions, which were ultimately resolved by manually creating and attaching a custom policy.
