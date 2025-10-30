# Usage Guide

## Getting Started

### 1. Choose Your Integration Method

#### Method A: Direct Parameters (Recommended)
Simple and explicit - pass all parameters directly in your workflow.

**Pros:**
- Clear and explicit
- Easy to understand
- No config file needed
- Best for single or few applications

**Cons:**
- More verbose
- Harder to manage many apps

#### Method B: Config File Mapping
Use a JSON config file to map application parameters.

**Pros:**
- DRY for multiple applications
- Centralized configuration
- Backward compatible with old workflows

**Cons:**
- Extra setup step
- Requires variable-mapper action

### 2. Set Up AWS OIDC Authentication

Both workflows require GitHub OIDC authentication to AWS.

#### Build Workflow IAM Role
**Default Role Name**: `github-oidc-provider-aws`

**Required Permissions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Deployment Workflow IAM Role
**Default Role Name**: `iam-github-oidc-role`

**Required Permissions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:DescribeCluster",
        "eks:ListClusters"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": "*"
    }
  ]
}
```

**Trust Policy** (both roles):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:wearevolt/*:*"
        }
      }
    }
  ]
}
```

### 3. Create Your Workflow File

Create `.github/workflows/ci-cd.yml` in your repository:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  workflow_dispatch:
    inputs:
      force_deploy:
        description: 'Force deployment'
        type: boolean
        default: false

jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: my-namespace
      environment: prod
      application: my-app
      working_directory: .
      ecr_repository: my-app

  deploy:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: my-namespace
      environment: prod
      application: my-app
      aws_account_id: "123456789012"
      eks_cluster_name: prod-cluster
      working_directory: helm
      helm_file_path: ./prod
      ecr_repository: my-app
      image_tag: ${{ needs.build.outputs.image_tag }}
```

### 4. Test Your Workflow

1. Commit and push the workflow file
2. Go to Actions tab in GitHub
3. Select "CI/CD Pipeline"
4. Click "Run workflow"
5. Monitor the execution

## Common Patterns

### Pattern 1: Multi-Environment with Branch-Based Deployment

```yaml
name: Multi-Environment CI/CD

on:
  push:
    branches: [develop, staging, main]

jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: ${{ github.ref == 'refs/heads/main' && 'production' || github.ref_name }}
      environment: ${{ github.ref == 'refs/heads/main' && 'prod' || github.ref_name }}
      application: my-app
      working_directory: .
      ecr_repository: my-app

  deploy:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: ${{ github.ref == 'refs/heads/main' && 'production' || github.ref_name }}
      environment: ${{ github.ref == 'refs/heads/main' && 'prod' || github.ref_name }}
      application: my-app
      aws_account_id: "123456789012"
      eks_cluster_name: ${{ github.ref == 'refs/heads/main' && 'prod-cluster' || 'dev-cluster' }}
      working_directory: helm
      helm_file_path: ./envs/${{ github.ref_name }}
      ecr_repository: my-app
      image_tag: ${{ needs.build.outputs.image_tag }}
```

### Pattern 2: Build Once, Deploy Many

```yaml
name: Build Once Deploy Many

on:
  push:
    branches: [main]

jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: shared
      environment: prod
      application: my-app
      working_directory: .
      ecr_repository: my-app

  deploy-us-east-1:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: production
      environment: prod
      application: my-app
      aws_account_id: "123456789012"
      aws_region: us-east-1
      eks_cluster_name: prod-us-east-1
      working_directory: helm
      helm_file_path: ./prod
      ecr_repository: my-app
      image_tag: ${{ needs.build.outputs.image_tag }}

  deploy-eu-west-1:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: production
      environment: prod
      application: my-app
      aws_account_id: "123456789012"
      aws_region: eu-west-1
      eks_cluster_name: prod-eu-west-1
      working_directory: helm
      helm_file_path: ./prod
      ecr_repository: my-app
      image_tag: ${{ needs.build.outputs.image_tag }}
```

### Pattern 3: PR Preview Environments

```yaml
name: PR Preview

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: pr-${{ github.event.pull_request.number }}
      environment: preview
      application: my-app
      working_directory: .
      ecr_repository: my-app

  deploy:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: pr-${{ github.event.pull_request.number }}
      environment: preview
      application: my-app
      aws_account_id: "123456789012"
      eks_cluster_name: dev-cluster
      working_directory: helm
      helm_file_path: ./preview
      ecr_repository: my-app
      image_tag: ${{ needs.build.outputs.image_tag }}
```

## Advanced Configuration

### Custom Docker Build Args

The build workflow doesn't currently support build args, but you can fork and modify it.

### Custom Helm Values

Pass custom values through helmfile by modifying your helmfile.yaml:

```yaml
releases:
  - name: {{ .Values.NAMESPACE }}
    namespace: {{ .Values.NAMESPACE }}
    chart: ./chart
    values:
      - image:
          repository: {{ .Values.DOCKER_IMAGE | splitList ":" | first }}
          tag: {{ .Values.DOCKER_IMAGE | splitList ":" | last }}
      - custom.yaml
```

### Multiple Applications in One Repo

```yaml
name: Multi-App CI/CD

on:
  push:
    branches: [main]

jobs:
  build-api:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: production
      environment: prod
      application: api
      working_directory: ./api
      ecr_repository: my-api

  build-worker:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: production
      environment: prod
      application: worker
      working_directory: ./worker
      ecr_repository: my-worker

  deploy-api:
    needs: build-api
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: production
      environment: prod
      application: api
      aws_account_id: "123456789012"
      eks_cluster_name: prod-cluster
      working_directory: helm/api
      helm_file_path: ./prod
      ecr_repository: my-api
      image_tag: ${{ needs.build-api.outputs.image_tag }}

  deploy-worker:
    needs: build-worker
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: production
      environment: prod
      application: worker
      aws_account_id: "123456789012"
      eks_cluster_name: prod-cluster
      working_directory: helm/worker
      helm_file_path: ./prod
      ecr_repository: my-worker
      image_tag: ${{ needs.build-worker.outputs.image_tag }}
```

## Tips & Best Practices

### 1. Use Version Tags
Always use version tags in production:
```yaml
uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1.0.0
```

### 2. Enable GitHub Environments
Use GitHub environments for deployment protection rules:
```yaml
deploy:
  environment: production  # Requires approval
  needs: build
  uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
```

### 3. Monitor Workflow Runs
Set up notifications for failed workflows in Slack/Teams.

### 4. Cache Docker Layers
The build workflow uses GitHub Actions cache for Docker layers automatically.

### 5. Timeout Configuration
Adjust deployment timeout for slow-starting applications:
```yaml
with:
  deployment_timeout: 600s  # 10 minutes
```

## Troubleshooting

### Issue: "Resource not accessible by integration"
**Solution**: Ensure the repository has workflows enabled and OIDC is configured.

### Issue: "Error: role is not authorized to perform sts:AssumeRoleWithWebIdentity"
**Solution**: Check IAM role trust policy includes your repository path.

### Issue: "Error: context deadline exceeded"
**Solution**: Increase `deployment_timeout` value.

### Issue: "Error: failed to solve with frontend dockerfile.v0"
**Solution**: Check Dockerfile path and working_directory are correct.

## Getting Help

For issues or questions:
1. Check the [main README](../README.md)
2. Review [example workflows](../README.md#examples)
3. Contact DevOps team

