# Quick Start Guide

## ✅ Repository Successfully Created!

The `devops-workflows` repository is now ready at:
**https://github.com/wearevolt/devops-workflows**

## 📦 What's Included

```
devops-workflows/
├── .github/
│   ├── workflows/
│   │   ├── build.yml           # Docker build & ECR push workflow
│   │   └── deployment.yml      # EKS deployment workflow
│   └── configs/
│       └── deploy.json.example # Configuration example
├── docs/
│   ├── usage.md                # Detailed usage guide
│   ├── setup-instructions.md   # Setup guide
│   └── example-ci-cd.yml       # Example workflow
├── README.md                   # Main documentation
├── CHANGELOG.md                # Version history
└── LICENSE                     # MIT License
```

## 🚀 Quick Start in 3 Steps

### Step 1: Configure Repository Access

Go to https://github.com/wearevolt/devops-workflows/settings/actions

Under **Access**, select:
- ✅ **Accessible from repositories in the 'wearevolt' organization**

Click **Save**.

### Step 2: Create Workflow in Your Repository

Create `.github/workflows/ci-cd.yml` in your project:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  workflow_dispatch:

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

### Step 3: Test It

1. Commit and push the workflow file
2. Go to **Actions** tab
3. Run the workflow manually
4. Check the logs

## 📚 Documentation

- **Main README**: [README.md](./README.md)
- **Usage Guide**: [docs/usage.md](./docs/usage.md)
- **Setup Instructions**: [docs/setup-instructions.md](./docs/setup-instructions.md)
- **Example Workflow**: [docs/example-ci-cd.yml](./docs/example-ci-cd.yml)

## 🏷️ Versions

Current versions:
- `@v1` - Latest v1.x.x (recommended for production)
- `@v1.0.0` - Specific version
- `@main` - Latest development (not recommended for production)

## 🔐 AWS OIDC Setup

Both workflows use OIDC for AWS authentication (no secrets needed!).

### Build Account Role
- **Role Name**: `github-oidc-provider-aws`
- **Account**: 345594573454 (or your build account)
- **Permissions**: ECR push

### Deploy Account Role
- **Role Name**: `iam-github-oidc-role`
- **Account**: Your target account
- **Permissions**: EKS, ECR read

See [docs/setup-instructions.md](./docs/setup-instructions.md) for detailed IAM configuration.

## 🎯 Example: Aurora AI Agent

An example migration is available in the `devops` repository:
```
devops/.github/workflows/aurora-ai-agent-ci-cd.yml
```

This shows how to migrate from old workflows to the new reusable ones.

## 📝 Common Use Cases

### 1. Single Environment Deployment
Use the basic example from Step 2 above.

### 2. Multi-Environment (dev, staging, prod)
```yaml
jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      environment: ${{ github.ref_name }}
      # ... other params ...

  deploy:
    needs: build
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      environment: ${{ github.ref_name }}
      eks_cluster_name: ${{ github.ref_name }}-cluster
      # ... other params ...
```

### 3. Different AWS Accounts
```yaml
jobs:
  build:
    uses: wearevolt/devops-workflows/.github/workflows/build.yml@v1
    with:
      aws_account_id: '345594573454'  # Build account
      # ... other params ...

  deploy:
    uses: wearevolt/devops-workflows/.github/workflows/deployment.yml@v1
    with:
      aws_account_id: '905418253345'  # Deploy account
      # ... other params ...
```

## ⚡ Benefits

✅ **Centralized maintenance** - Update once, apply everywhere
✅ **Version control** - Pin specific versions for stability
✅ **Consistency** - Same process across all apps
✅ **Easy testing** - Test changes before rolling out
✅ **Better documentation** - Single source of truth

## 🔄 Updating Workflows

When you need to update:

1. Make changes in a feature branch
2. Test thoroughly
3. Merge to main
4. Create new version:
   ```bash
   git tag -a v1.0.1 -m "Description"
   git push origin v1.0.1
   
   # Update major version tag
   git tag -fa v1 -m "Update to v1.0.1"
   git push origin v1 --force
   ```
5. Update CHANGELOG.md

## 🆘 Troubleshooting

### "workflow is not accessible"
→ Check Step 1 above (repository access settings)

### "role cannot be assumed"
→ Check AWS OIDC configuration and trust policies

### "image not found"
→ Ensure `image_tag: ${{ needs.build.outputs.image_tag }}`

### "helmfile not found"
→ Verify `working_directory` and `helm_file_path` paths

## 📞 Getting Help

- Review [README.md](./README.md) for full documentation
- Check [docs/usage.md](./docs/usage.md) for detailed examples
- See [docs/setup-instructions.md](./docs/setup-instructions.md) for setup help
- Contact DevOps team for specific questions

## ✨ What's Next?

1. ✅ Configure repository access (Step 1)
2. ✅ Test with one application
3. ✅ Migrate existing workflows
4. ✅ Train team members
5. ✅ Set up monitoring

## 🎉 You're Ready!

Start using the workflows in your projects. The more projects use them, the more value you get from centralized maintenance!

---

**Repository**: https://github.com/wearevolt/devops-workflows
**Issues**: https://github.com/wearevolt/devops-workflows/issues
**License**: MIT

