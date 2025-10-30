# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2025-10-30

### Added
- Initial release of reusable workflows
- Build workflow for Docker image building and ECR push
- Deployment workflow for EKS deployment using Helmfile
- Comprehensive documentation and usage examples
- Support for OIDC authentication with AWS
- GitHub Actions cache for Docker layer caching
- Build and deployment summaries in workflow runs
- Flexible input parameters with sensible defaults
- Support for multiple environments and AWS accounts

### Features
- **Build Workflow**:
  - Builds Docker images from any directory
  - Pushes to Amazon ECR
  - Generates standardized image tags
  - Uses GitHub Actions cache for faster builds
  - Outputs image tag for downstream jobs

- **Deployment Workflow**:
  - Deploys to EKS using Helmfile
  - Supports SOPS-encrypted secrets
  - Performs helm diff before deployment
  - Verifies deployment rollout
  - Configurable timeout
  - Multi-region support

### Documentation
- Complete README with quick start guide
- Detailed usage documentation
- Multiple example patterns
- Migration guide from old workflows
- Troubleshooting section

[Unreleased]: https://github.com/wearevolt/devops-workflows/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/wearevolt/devops-workflows/releases/tag/v1.0.0

