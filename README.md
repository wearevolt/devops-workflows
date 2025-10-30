# DevOps Reusable Workflows

This repository contains reusable GitHub Actions workflows for building and deploying applications to AWS EKS.

## 📦 Available Workflows

### 1. Build Workflow
Builds Docker images and pushes them to Amazon ECR.

**Path**: `.github/workflows/build.yml`

### 2. Deployment Workflow
Deploys applications to EKS using Helmfile.

**Path**: `.github/workflows/deployment.yml`

## 🚀 Quick Start

### Option 1: Using with deploy.json config file

This approach uses a configuration file to map application parameters.

#### Step 1: Create deploy.json in your repository

Create `.github/configs/deploy.json`:

```json
{
  "your-app__your-namespace.prod": {
    "ECR_REPOSITORY": "your-ecr-repo",
    "DOCKER_FILE": "Dockerfile",
    "HELM_FILE_PATH": "../envs/prod",
    "WORKING_DIRECTORY": "helm/your-app/code",
    "ENVIRONMENT": "prod",
    "NAMESPACE": "your-namespace",
    "APPLICATION": "your-app",
    "AWS_ACCOUNT_ID": "123456789012",
    "EKS_CLUSTER_NAME": "prod-cluster"
  }
}
```

#### Step 2: Create workflow in your repository

Create `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  workflow_dispatch:
    inputs:
      namespace:
        type: string
        description: Namespace
        required: true
      environment:
        type: string
        description: Environment
        required: true
      application:
        type: string
        description: Application
        required: true

jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      config: ${{ steps.load-config.outputs.config }}
    steps:
      - uses: actions/checkout@v4
      
      - id: load-config
        run: |
          CONFIG=$(cat .github/configs/deploy.json | jq -c)
          echo "config=$CONFIG" >> $GITHUB_OUTPUT
      
      - uses: kanga333/variable-mapper@master
        id: map
        with:
          key: "${{ inputs.application }}__${{ inputs.namespace }}.${{ inputs.environment }}"
          map: ${{ steps.load-config.outputs.config }}

  build:
    needs: setup
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@main
    with:
      namespace: ${{ inputs.namespace }}
      environment: ${{ inputs.environment }}
      application: ${{ inputs.application }}
      working_directory: ${{ needs.setup.outputs.WORKING_DIRECTORY }}
      docker_file: ${{ needs.setup.outputs.DOCKER_FILE }}
      ecr_repository: ${{ needs.setup.outputs.ECR_REPOSITORY }}

  deploy:
    needs: [setup, build]
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@main
    with:
      namespace: ${{ inputs.namespace }}
      environment: ${{ inputs.environment }}
      application: ${{ inputs.application }}
      aws_account_id: ${{ needs.setup.outputs.AWS_ACCOUNT_ID }}
      eks_cluster_name: ${{ needs.setup.outputs.EKS_CLUSTER_NAME }}
      working_directory: ${{ needs.setup.outputs.WORKING_DIRECTORY }}
      helm_file_path: ${{ needs.setup.outputs.HELM_FILE_PATH }}
      ecr_repository: ${{ needs.setup.outputs.ECR_REPOSITORY }}
      image_tag: ${{ needs.build.outputs.image_tag }}
```

### Option 2: Direct parameters (Recommended for simplicity)

Create `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  workflow_dispatch:

jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: my-namespace
      environment: prod
      application: my-app
      working_directory: helm/my-app/code
      docker_file: Dockerfile
      ecr_repository: my-ecr-repo
      aws_account_id: "123456789012"

  deploy:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: my-namespace
      environment: prod
      application: my-app
      aws_account_id: "123456789012"
      eks_cluster_name: prod-cluster
      working_directory: helm/my-app/code
      helm_file_path: ../envs/prod
      ecr_repository: my-ecr-repo
      image_tag: ${{ needs.build.outputs.image_tag }}
```

## 📝 Workflow Inputs

### Build Workflow Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `namespace` | ✅ | - | Kubernetes namespace |
| `environment` | ✅ | - | Environment (prod, staging, dev) |
| `application` | ✅ | - | Application name |
| `working_directory` | ✅ | - | Build context directory |
| `ecr_repository` | ✅ | - | ECR repository name |
| `docker_file` | ❌ | `Dockerfile` | Dockerfile path relative to working_directory |
| `ecr_registry` | ❌ | `345594573454.dkr.ecr.us-east-1.amazonaws.com` | ECR registry URL |
| `aws_region` | ❌ | `us-east-1` | AWS region |
| `aws_account_id` | ❌ | `345594573454` | AWS account ID for build OIDC |
| `github_oidc_role` | ❌ | `github-oidc-provider-aws` | GitHub OIDC role name |

### Build Workflow Outputs

| Output | Description |
|--------|-------------|
| `image_tag` | Full Docker image tag (format: `app.namespace.env.sha`) |

### Deployment Workflow Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `namespace` | ✅ | - | Kubernetes namespace |
| `environment` | ✅ | - | Environment name |
| `application` | ✅ | - | Application name |
| `aws_account_id` | ✅ | - | AWS account ID |
| `eks_cluster_name` | ✅ | - | EKS cluster name |
| `working_directory` | ✅ | - | Working directory with helmfile |
| `helm_file_path` | ✅ | - | Path to helmfile relative to working_directory |
| `ecr_repository` | ✅ | - | ECR repository name |
| `image_tag` | ✅ | - | Docker image tag to deploy |
| `ecr_registry` | ❌ | `345594573454.dkr.ecr.us-east-1.amazonaws.com` | ECR registry URL |
| `aws_region` | ❌ | `us-east-1` | AWS region |
| `github_oidc_role` | ❌ | `iam-github-oidc-role` | GitHub OIDC role name |
| `kubectl_version` | ❌ | `1.34.1` | Kubectl version |
| `kubectl_release_date` | ❌ | `2025-09-19` | Kubectl release date |
| `deployment_timeout` | ❌ | `300s` | Deployment timeout |

## 🔐 Required Secrets

Both workflows use GitHub OIDC for AWS authentication. Ensure you have:

1. **AWS IAM OIDC Provider** configured for GitHub Actions
2. **IAM Roles** with appropriate permissions:
   - Build role (default: `github-oidc-provider-aws`) needs ECR push permissions
   - Deploy role (default: `iam-github-oidc-role`) needs EKS and ECR read permissions

No secrets need to be configured in GitHub - authentication uses OIDC!

## 🏷️ Versioning

### Using Specific Versions

Always use versioned releases in production:

```yaml
uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1.0.0
```

### Version Tags

- `@main` - Latest development version (not recommended for production)
- `@v1` - Latest v1.x.x release
- `@v1.0.0` - Specific version

## 📖 Examples

### Example 1: Simple Single-Environment Deployment

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: production
      environment: prod
      application: my-api
      working_directory: .
      ecr_repository: my-api

  deploy:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: production
      environment: prod
      application: my-api
      aws_account_id: "123456789012"
      eks_cluster_name: prod-cluster
      working_directory: helm
      helm_file_path: ./production
      ecr_repository: my-api
      image_tag: ${{ needs.build.outputs.image_tag }}
```

### Example 2: Multi-Environment with Manual Approval

```yaml
name: Multi-Environment Deployment

on:
  push:
    branches: [main]

jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: staging
      environment: staging
      application: my-app
      working_directory: .
      ecr_repository: my-app

  deploy-staging:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: staging
      environment: staging
      application: my-app
      aws_account_id: "123456789012"
      eks_cluster_name: staging-cluster
      working_directory: helm
      helm_file_path: ./staging
      ecr_repository: my-app
      image_tag: ${{ needs.build.outputs.image_tag }}

  deploy-production:
    needs: [build, deploy-staging]
    environment: production  # GitHub environment with approvals
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: production
      environment: prod
      application: my-app
      aws_account_id: "123456789012"
      eks_cluster_name: prod-cluster
      working_directory: helm
      helm_file_path: ./production
      ecr_repository: my-app
      image_tag: ${{ needs.build.outputs.image_tag }}
```

### Example 3: With Custom AWS Account per Environment

```yaml
name: Multi-Account Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [staging, production]

jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: my-namespace
      environment: ${{ inputs.environment }}
      application: my-app
      working_directory: .
      ecr_repository: my-app
      # Use staging account for builds
      aws_account_id: "111111111111"

  deploy:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: my-namespace
      environment: ${{ inputs.environment }}
      application: my-app
      # Dynamic account based on environment
      aws_account_id: ${{ inputs.environment == 'production' && '222222222222' || '111111111111' }}
      eks_cluster_name: ${{ inputs.environment }}-cluster
      working_directory: helm
      helm_file_path: ./envs/${{ inputs.environment }}
      ecr_repository: my-app
      image_tag: ${{ needs.build.outputs.image_tag }}
```

## 🔄 Migration from Old Workflows

If you're migrating from the old workflow pattern with `deploy.json`, follow these steps:

### Step 1: Keep your deploy.json file
Your existing `.github/configs/deploy.json` can still be used.

### Step 2: Create a mapping workflow

Create `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD

on:
  workflow_dispatch:
    inputs:
      namespace:
        type: string
        required: true
      environment:
        type: string
        required: true
      application:
        type: string
        required: true

jobs:
  map-config:
    runs-on: ubuntu-latest
    outputs:
      ecr_repository: ${{ steps.map.outputs.ECR_REPOSITORY }}
      docker_file: ${{ steps.map.outputs.DOCKER_FILE }}
      helm_file_path: ${{ steps.map.outputs.HELM_FILE_PATH }}
      working_directory: ${{ steps.map.outputs.WORKING_DIRECTORY }}
      aws_account_id: ${{ steps.map.outputs.AWS_ACCOUNT_ID }}
      eks_cluster_name: ${{ steps.map.outputs.EKS_CLUSTER_NAME }}
    steps:
      - uses: actions/checkout@v4
      
      - id: deploy-map
        run: |
          echo "map=$(cat .github/configs/deploy.json | jq -c)" >> $GITHUB_OUTPUT
      
      - uses: kanga333/variable-mapper@master
        id: map
        with:
          key: "${{ inputs.application }}__${{ inputs.namespace }}.${{ inputs.environment }}"
          map: ${{ steps.deploy-map.outputs.map }}
          export_to: output

  build:
    needs: map-config
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      namespace: ${{ inputs.namespace }}
      environment: ${{ inputs.environment }}
      application: ${{ inputs.application }}
      working_directory: ${{ needs.map-config.outputs.working_directory }}
      docker_file: ${{ needs.map-config.outputs.docker_file }}
      ecr_repository: ${{ needs.map-config.outputs.ecr_repository }}

  deploy:
    needs: [map-config, build]
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      namespace: ${{ inputs.namespace }}
      environment: ${{ inputs.environment }}
      application: ${{ inputs.application }}
      aws_account_id: ${{ needs.map-config.outputs.aws_account_id }}
      eks_cluster_name: ${{ needs.map-config.outputs.eks_cluster_name }}
      working_directory: ${{ needs.map-config.outputs.working_directory }}
      helm_file_path: ${{ needs.map-config.outputs.helm_file_path }}
      ecr_repository: ${{ needs.map-config.outputs.ecr_repository }}
      image_tag: ${{ needs.build.outputs.image_tag }}
```

## 🛠️ Troubleshooting

### Image not found in ECR
Ensure the `image_tag` output from build workflow is properly passed to deployment workflow.

### OIDC Authentication Failed
Check that:
1. IAM OIDC provider is configured for GitHub
2. IAM role exists and has correct trust policy
3. Role has necessary permissions (ECR, EKS)

### Helmfile Not Found
Ensure `working_directory` and `helm_file_path` are correct relative paths.

## 📄 License

Internal use only - WeAreVolt

## 🤝 Contributing

To contribute:
1. Create a feature branch
2. Make changes
3. Test with a real application
4. Create a PR
5. Tag a new version after merge

## 📞 Support

For issues or questions, contact the DevOps team.

