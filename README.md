# deploy-aws-lambda

Deploy AWS Lambda container functions from GitHub Actions with AWS OIDC, ECR push, Lambda update, optional API Gateway trigger creation, and Lambda Function URL provisioning.


## Features

- Builds Docker image from your function directory
- Pushes both `latest` and commit SHA tags to ECR
- Creates ECR repository if it does not exist
- Creates Lambda function on first deploy, updates image on subsequent deploys
- Ensures API Gateway HTTP API trigger exists (idempotent)
- Ensures Lambda Function URL exists
- Exposes reusable outputs (`image-uri-latest`, `image-uri-sha`, `image-tag-sha`, `lambda-url`, `api-gateway-url`)

## Quick Start

### 1) Caller workflow example

Create `.github/workflows/deploy.yml` in your application repository:

```yaml
name: Deploy Lambda

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # Required: this action no longer performs checkout internally.
      - name: Checkout
        uses: actions/checkout@v6

      - name: Deploy AWS Lambda
        id: deploy
        uses: ruandada/deploy-aws-lambda@v1
        with:
          build-function: aws-lambda-typescript-docker-starter

      - name: Print outputs
        run: |
          echo "latest=${{ steps.deploy.outputs.image-uri-latest }}"
          echo "sha=${{ steps.deploy.outputs.image-uri-sha }}"
          echo "tag=${{ steps.deploy.outputs.image-tag-sha }}"
          echo "url=${{ steps.deploy.outputs.lambda-url }}"
          echo "apigw=${{ steps.deploy.outputs.api-gateway-url }}"
```



## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `build-function` | Yes | - | Function directory path that contains source and deployment config |
| `config-file` | No | `aws-lambda.yaml` | Deployment config filename under `build-function` |

## Outputs

| Name | Description |
| --- | --- |
| `image-uri-latest` | ECR image URI with `latest` tag |
| `image-uri-sha` | ECR image URI with immutable SHA tag |
| `image-tag-sha` | Generated tag value (for example `sha-abc123...`) |
| `lambda-url` | Lambda Function URL |
| `api-gateway-url` | API Gateway HTTP API endpoint |

## Required AWS Setup

Your AWS account should have:

- An IAM role for GitHub OIDC deployment (`DeploymentRoleName`)
- An IAM role for Lambda runtime (`ExecutionRoleName`)
- OIDC trust policy that allows your GitHub repository to assume the deployment role
- Deployment role permissions to manage ECR, Lambda, and API Gateway resources used by this action

## Required GitHub Workflow Permissions

In the caller workflow/job:

```yaml
permissions:
  id-token: write
  contents: read
```

## Behavior Notes

- The action is designed to be idempotent for common deploy paths
- If Lambda function does not exist, it will be created
- If API Gateway with the same `ApiGatewayName` already exists, creation is skipped
- Function URL is reused if already present

## Versioning

Recommended pinning strategy:

- Stable major: `@v1`
- Exact release: `@v1.0.0`
- Immutable commit SHA for maximum supply-chain control

Example release commands:

```bash
git tag v1.0.0
git push origin v1.0.0
git tag -f v1
git push -f origin v1
```

## License

This code is made available under the MIT license.
