# Sosoka-Labs

Organization-wide GitHub configuration and reusable workflows.

## Reusable Workflows

Shared CI/CD workflows for Sosoka-Labs repositories.

### Available Workflows

| Workflow | Description |
|----------|-------------|
| `test-python.yml` | Test Python projects (lint, type check, pytest) |
| `build-lambda.yml` | Build Python Lambda packages |
| `deploy-lambda.yml` | Deploy Lambda packages to AWS |
| `build-nextjs.yml` | Build Next.js static sites |
| `deploy-static-site.yml` | Deploy to S3 + CloudFront |

## Python Lambda Workflows

### Test Python

Runs linting (ruff), type checking (mypy), and tests (pytest) with coverage.

```yaml
jobs:
  test:
    uses: Sosoka-Labs/.github/.github/workflows/test-python.yml@main
    with:
      python-version: '3.12'
      run-mypy: true
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `python-version` | No | `'3.12'` | Python version |
| `working-directory` | No | `'.'` | Working directory |
| `run-mypy` | No | `true` | Run mypy type checking |

### Build Lambda

Builds Lambda deployment packages (zip files with dependencies).

```yaml
jobs:
  build:
    uses: Sosoka-Labs/.github/.github/workflows/build-lambda.yml@main
    with:
      lambdas: 'contact-api,contact-processor'
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `python-version` | No | `'3.12'` | Python version |
| `working-directory` | No | `'.'` | Working directory |
| `lambdas` | Yes | - | Comma-separated Lambda names |

**Outputs:**
| Output | Description |
|--------|-------------|
| `artifact-name` | Name of uploaded artifact (`lambda-packages`) |

### Deploy Lambda

Deploys Lambda packages to AWS via S3 and updates function code.

```yaml
jobs:
  deploy:
    needs: build
    uses: Sosoka-Labs/.github/.github/workflows/deploy-lambda.yml@main
    with:
      environment: staging
      lambdas: 'contact-api,contact-processor'
      s3-bucket: my-api-artifacts-staging
      aws-region: us-west-2
    secrets: inherit
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `environment` | Yes | - | Deployment environment |
| `lambdas` | Yes | - | Comma-separated Lambda names |
| `s3-bucket` | Yes | - | S3 bucket for artifacts |
| `aws-region` | No | `'us-west-2'` | AWS region |
| `artifact-name` | No | `'lambda-packages'` | Artifact name from build |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `AWS_ROLE_ARN` | Yes | IAM Role ARN for OIDC auth |

## Full Lambda Pipeline Example

```yaml
name: CI/CD

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [develop, main]

jobs:
  test:
    uses: Sosoka-Labs/.github/.github/workflows/test-python.yml@main
    with:
      python-version: '3.12'

  build:
    needs: test
    if: github.event_name == 'push'
    uses: Sosoka-Labs/.github/.github/workflows/build-lambda.yml@main
    with:
      lambdas: 'contact-api,contact-processor'

  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    uses: Sosoka-Labs/.github/.github/workflows/deploy-lambda.yml@main
    with:
      environment: staging
      lambdas: 'contact-api,contact-processor'
      s3-bucket: my-api-artifacts-staging
    secrets: inherit

  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    uses: Sosoka-Labs/.github/.github/workflows/deploy-lambda.yml@main
    with:
      environment: production
      lambdas: 'contact-api,contact-processor'
      s3-bucket: my-api-artifacts-production
    secrets: inherit
```

## Static Site Workflows

### Build Next.js

```yaml
jobs:
  build:
    uses: Sosoka-Labs/.github/.github/workflows/build-nextjs.yml@main
    with:
      working-directory: website
```

### Deploy Static Site

```yaml
jobs:
  deploy:
    needs: build
    uses: Sosoka-Labs/.github/.github/workflows/deploy-static-site.yml@main
    with:
      environment: staging
      s3-bucket: my-app-staging
      cloudfront-alias: staging.myapp.sosoka.io
    secrets: inherit
```

## Branching Model

| Branch | Environment | Deployment |
|--------|-------------|------------|
| `develop` | staging | Automatic on push |
| `main` | production | Automatic on push |

Pull requests to `develop` and `main` trigger test workflows only.

## AWS OIDC Authentication

All deploy workflows use OIDC for secure, keyless authentication. Configure your repository with:

1. **Environment secrets:** Set `AWS_ROLE_ARN` for each environment (staging, production)
2. **IAM Role:** Create IAM role with trust policy for GitHub OIDC provider
3. **Permissions:** Role needs S3 write and Lambda update permissions

See [AWS OIDC docs](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) for setup.
