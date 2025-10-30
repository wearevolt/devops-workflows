# Setup Instructions

## GitHub Repository Configuration

After creating the repository, follow these steps to make it work properly.

### 1. Enable Workflow Permissions

1. Go to repository **Settings** → **Actions** → **General**
2. Under **Workflow permissions**, select:
   - ✅ **Read and write permissions**
3. Under **Allow GitHub Actions to create and approve pull requests**:
   - ✅ Check this box
4. Click **Save**

### 2. Configure Access from Other Repositories

1. Go to repository **Settings** → **Actions** → **General**
2. Under **Access**, select one of:
   - ✅ **Accessible from repositories in the 'wearevolt' organization** (Recommended)
   - Or: **Accessible from repositories owned by the user 'wearevolt'**
3. Click **Save**

This allows other repositories in your organization to use these workflows.

### 3. Create GitHub Release (Optional but Recommended)

Create a release on GitHub for better version tracking:

1. Go to **Releases** → **Draft a new release**
2. Choose tag: `v1.0.0`
3. Release title: `v1.0.0 - Initial Release`
4. Description: Copy from CHANGELOG.md
5. Click **Publish release**

### 4. Protect Main Branch (Recommended)

1. Go to **Settings** → **Branches**
2. Click **Add rule**
3. Branch name pattern: `main`
4. Enable:
   - ✅ Require a pull request before merging
   - ✅ Require approvals (1)
   - ✅ Require status checks to pass
5. Click **Create**

### 5. Set Up Branch Protection for Tags (Recommended)

To prevent accidental tag deletion:

1. Go to **Settings** → **Tags** → **Protected tags**
2. Add pattern: `v*`
3. This protects all version tags

## AWS OIDC Configuration

### For Build Workflow

The build workflow needs an IAM role in the **build AWS account** (default: 345594573454).

**IAM Role Name**: `github-oidc-provider-aws`

**Trust Policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::345594573454:oidc-provider/token.actions.githubusercontent.com"
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

**Permissions Policy**:
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

### For Deployment Workflow

The deployment workflow needs an IAM role in **each target AWS account**.

**IAM Role Name**: `iam-github-oidc-role` (or custom name)

**Trust Policy** (same as above, but in target account):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::TARGET_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
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

**Permissions Policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EKSAccess",
      "Effect": "Allow",
      "Action": [
        "eks:DescribeCluster",
        "eks:ListClusters"
      ],
      "Resource": "*"
    },
    {
      "Sid": "ECRReadAccess",
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

### Create OIDC Provider (if not exists)

If the OIDC provider doesn't exist in your AWS account:

```bash
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
```

## Testing the Workflows

### Test in a Sample Repository

1. Create a test repository or use an existing one
2. Add the example workflow from `docs/example-ci-cd.yml`
3. Update the inputs with your values
4. Commit and push
5. Go to Actions tab and run the workflow manually
6. Check the logs for any errors

### Common Issues

**Issue**: "Resource not accessible by integration"
**Solution**: Check step 2 above - ensure the repository has access configured

**Issue**: "Role cannot be assumed"
**Solution**: Check AWS OIDC configuration and trust policies

**Issue**: "ECR repository not found"
**Solution**: Create the ECR repository first or check the repository name

## Updating the Workflows

When you need to update the workflows:

1. Make changes in a feature branch
2. Test thoroughly
3. Merge to main
4. Create a new version tag:
   ```bash
   git tag -a v1.0.1 -m "Release v1.0.1: Description"
   git push origin v1.0.1
   ```
5. Update the major version tag:
   ```bash
   git tag -fa v1 -m "Update v1 to v1.0.1"
   git push origin v1 --force
   ```
6. Update CHANGELOG.md

### Versioning Strategy

- **Major version (v1, v2)**: Breaking changes
- **Minor version (v1.1, v1.2)**: New features, backward compatible
- **Patch version (v1.0.1, v1.0.2)**: Bug fixes

## Monitoring Usage

To see which repositories are using these workflows:

1. Go to **Insights** → **Dependency graph** → **Dependents**
2. You'll see repositories that reference this workflow repository

## Getting Help

If you encounter issues:
1. Check the [Usage Documentation](./usage.md)
2. Review the [README](../README.md)
3. Check GitHub Actions logs for detailed error messages
4. Contact DevOps team

## Next Steps

After setup is complete:

1. ✅ Test workflows with a sample application
2. ✅ Update existing repositories to use these workflows
3. ✅ Document any custom configurations
4. ✅ Train team members on usage
5. ✅ Set up monitoring for workflow failures

