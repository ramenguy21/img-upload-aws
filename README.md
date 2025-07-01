# Image Upload with AppSync & S3

This project implements a full-stack image upload solution using AWS AppSync, Cognito, and S3, all provisioned via CloudFormation.

## Architecture Overview

- **Cognito User Pool**: Handles user authentication
- **AppSync GraphQL API**: Provides the GraphQL endpoint with authentication
- **S3 Bucket**: Stores uploaded images with proper folder structure
- **IAM Roles**: Manages permissions between services

## Deployment

### Prerequisites

- AWS CLI configured with appropriate permissions
- Valid AWS account with permissions to create CloudFormation stacks

### Deploy the Stack

```bash
aws cloudformation deploy
  --template-file template.yaml
  --stack-name image-upload-stack
  --parameter-overrides StackName=image-upload-stack
  --capabilities CAPABILITY_NAMED_IAM
  --region me-south-1
```

### Get Stack Outputs

After deployment, retrieve the important values:

```bash
aws cloudformation describe-stacks \
  --stack-name image-upload-stack \
  --query 'Stacks[0].Outputs'
```

## Authentication Setup

### 1. Sign Up a New User

```bash
aws cognito-idp sign-up \
  --client-id YOUR_CLIENT_ID \
  --username user@example.com \
  --password TempPassword123! \
  --user-attributes Name=email,Value=user@example.com
```

### 2. Confirm User (if auto-verification is disabled)

```bash
aws cognito-idp admin-confirm-sign-up \
  --user-pool-id YOUR_USER_POOL_ID \
  --username user@example.com
```

### 3. Get Authentication Token

```bash
aws cognito-idp initiate-auth \
  --client-id YOUR_CLIENT_ID \
  --auth-flow USER_PASSWORD_AUTH \
  --auth-parameters USERNAME=user@example.com,PASSWORD=TempPassword123!
```

Save the `AccessToken` from the response for GraphQL requests.

## GraphQL Usage

### Example Mutation

```graphql
mutation GetImageUploadUrl($fileName: String!, $fileType: String!) {
  getImageUploadUrl(fileName: $fileName, fileType: $fileType) {
    uploadUrl
    key
  }
}
```

### Variables

```json
{
  "fileName": "my-image.jpg",
  "fileType": "image/jpeg"
}
```

### Headers

```json
{
  "Authorization": "Bearer YOUR_ACCESS_TOKEN"
}
```

## Testing Checklist

### ✅ Authentication Tests

- [ ] **Unauthorized Request**: Call the mutation without an Authorization header
  - Expected: GraphQL error indicating authentication failure
- [ ] **Invalid Token**: Call with an expired or malformed token

  - Expected: GraphQL error indicating invalid authentication

- [ ] **Valid Token**: Call with a valid JWT token from Cognito
  - Expected: Successful response with uploadUrl and key

### ✅ Functionality Tests

- [ ] **Presigned URL Generation**: Verify the mutation returns a valid presigned URL
  - Expected: URL should be accessible and properly formatted
- [ ] **Correct S3 Key Format**: Verify the key follows `images/{userId}/{fileName}`
  - Expected: Key should contain the authenticated user's ID
- [ ] **URL Expiration**: Test that the URL expires after ~5 minutes
  - Expected: URL should become invalid after expiration time

### ✅ File Upload Tests

- [ ] **Successful Upload**: Use the presigned URL to upload an image
  ```bash
  curl -X PUT \
    -H "Content-Type: image/jpeg" \
    --data-binary @test-image.jpg \
    "PRESIGNED_URL_FROM_MUTATION"
  ```
  - Expected: HTTP 200 response
- [ ] **Verify S3 Location**: Check that the file appears in the correct S3 path
  ```bash
  aws s3 ls s3://YOUR_BUCKET_NAME/images/USER_ID/
  ```
  - Expected: File should be visible in the user's folder

### ✅ Security Tests

- [ ] **Cross-User Access**: Verify users cannot access other users' upload paths
  - Expected: Each user should only be able to upload to their own folder
- [ ] **File Type Validation**: Test with different file types
  - Expected: Content-Type should be properly set based on fileType parameter

## Project Structure

```
.
├── template.yaml          # CloudFormation template
├── README.md             # This documentation
└── test-image.jpg        # Sample image for testing (add your own)
```

## Troubleshooting

### Common Issues

1. **Stack Creation Fails**: Check CloudFormation events in AWS Console
2. **Authentication Errors**: Verify Cognito configuration and token validity
3. **S3 Upload Fails**: Check CORS configuration and presigned URL format
4. **GraphQL Errors**: Verify schema and resolver mapping templates

### Useful Commands

```bash
# Check stack status
aws cloudformation describe-stacks --stack-name image-upload-stack

# View stack events
aws cloudformation describe-stack-events --stack-name image-upload-stack

# Delete stack (cleanup)
aws cloudformation delete-stack --stack-name image-upload-stack
```

## Notes

- The presigned URL expires in 5 minutes (300 seconds)
- Images are stored in the format: `images/{userId}/{fileName}`
- Only authenticated users can generate upload URLs
- CORS is configured on the S3 bucket for web uploads# img-upload-aws
