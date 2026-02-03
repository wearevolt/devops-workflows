# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.9.0] - 2026-02-03

### Added
- **Custom image tag support**: New optional `image_tag` input for `gitops-deploy.yml` workflow
  - Allows specifying custom Docker image tags (e.g., semver tags like `v1.0.0`)
  - If not provided, falls back to default tag format: `{application}.{namespace}.{environment}.{sha}`
  - Useful for release-based deployments with semantic versioning

- **Image promotion/rollback support**: New optional `skip_build` input for `gitops-deploy.yml` workflow
  - When `true`, skips Docker build and uses existing image from ECR
  - Requires `image_tag` to be set when using promotion mode
  - Validates that the specified image exists in ECR before proceeding
  - Shows available tags if image not found
  - Enables "Build Once, Deploy Everywhere" pattern for consistent deployments
  - Use cases: promote QA image to STG/PROD, rollback to previous version

## [1.3.0] - 2025-12-22

### Added
- **Auto-create ECR repositories**: Build workflow now automatically creates ECR repositories if they don't exist
  - Repositories are created with image scanning enabled (scanOnPush=true)
  - Uses AES256 encryption by default
  - Shows "Created automatically" in build summary when repository was created

### Changed
- Build workflow now includes "Ensure ECR repository exists" step before building

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

[Unreleased]: https://github.com/wearevolt/devops-workflows/compare/v1.9.0...HEAD
[1.9.0]: https://github.com/wearevolt/devops-workflows/compare/v1.8.1...v1.9.0
[1.3.0]: https://github.com/wearevolt/devops-workflows/compare/v1.0.0...v1.3.0
[1.0.0]: https://github.com/wearevolt/devops-workflows/releases/tag/v1.0.0

