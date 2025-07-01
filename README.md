# Image Upload Service

This project provisions an AppSync GraphQL endpoint that generates presigned URLs for direct image uploads to S3.

## 🚀 Deployment

1. Install AWS CLI (if not installed)
2. Clone the repo
3. Deploy the stack:

```
aws cloudformation deploy --template-file template.yaml --stack-name image-upload-stack --capabilities CAPABILITY_IAM --region REGION_HERE
```

4. Get outputs after deployment:

```
aws cloudformation describe-stacks --stack-name image-upload-stack --query "Stacks[0].Outputs"
```

Save these values: _GraphQLApiEndpoint, UserPoolId, UserPoolClientId, S3BucketName._

## 🔐 User Setup

1. Create a user:

```
aws cognito-idp sign-up --client-id YOUR_USERPOOL_CLIENT_ID --username user@example.com --password "SecurePass123!" --region YOUR_REGION
```

2. Confirm the user:

```
aws cognito-idp admin-confirm-sign-up --user-pool-id YOUR_USER_POOL_ID --username user@example.com --region YOUR_REGION
```

3. Get an authentication token:

```
aws cognito-idp initiate-auth
  --client-id YOUR_USERPOOL_CLIENT_ID
  --auth-flow USER_PASSWORD_AUTH
  --auth-parameters USERNAME=user@example.com,PASSWORD="SecurePass123!"
  --region YOUR_REGION
  --query "AuthenticationResult.IdToken"
```

4. Copy the returned ID Token for API requests.

📮 Using the GraphQL API

1. Set headers:

```
http
Authorization: <ID_TOKEN>
Content-Type: application/json
```

2. Get an upload URL:

```
mutation GetUploadUrl {
  getImageUploadUrl(
    fileName: "sunset.jpg",
    fileType: "image/jpeg"
  ) {
    uploadUrl
    key
  }
}
```

3. Response example:

```
{
  "data": {
    "getImageUploadUrl": {
      "uploadUrl": "https://...",
      "key": "images/.../sunset.jpg"
    }
  }
}
```

✅ Testing Workflow

1. Unauthenticated access:
   Call the API without token → Should return 401 Unauthorized.
2. Upload an image:

```
curl -X PUT -H "Content-Type: image/jpeg" --data-binary "@local-image.jpg"         "<UPLOAD_URL_FROM_RESPONSE>"
```

### Verify in S3:

Check the bucket path:
s3://YOUR_BUCKET_NAME/images/<user-id>/sunset.jpg
