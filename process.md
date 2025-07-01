# Documentation on the workflow followed to create the stack

## First Steps

### Setup
- Installed AWS CLI.
- Procured IAM credintials for a user with AdministratorAcess Policy.
- Configured the AWS CLI with the user's access key ID and secret access key.

For testing, I used 
``` aws sts get-caller-identity ```

### Creating the Template 

## Requirements 
- Cognito User Pool with email-based authentication
- S3 bucket with proper CORS configuration
- AppSync GraphQL API with Cognito authentication
- IAM roles and policies for service integration
- GraphQL schema with the required mutation
- Resolver for generating presigned URLs