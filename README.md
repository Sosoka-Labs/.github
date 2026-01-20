# Sosoka-Labs

Organization-wide GitHub configuration and reusable workflows.

## Reusable Workflows

Shared CI/CD workflows for Sosoka-Labs repositories.

### Available Workflows

| Workflow | Description |
|----------|-------------|
| `build-nextjs.yml` | Build Next.js static sites |
| `deploy-static-site.yml` | Deploy to S3 + CloudFront |

### Usage

```yaml
jobs:
  build:
    uses: Sosoka-Labs/.github/.github/workflows/build-nextjs.yml@main
    with:
      working-directory: website

  deploy:
    needs: build
    uses: Sosoka-Labs/.github/.github/workflows/deploy-static-site.yml@main
    with:
      environment: staging
      s3-bucket: my-app-staging
      cloudfront-alias: staging.myapp.sosoka.io
    secrets: inherit
```

See [workflow documentation](.github/workflows/) for full input options.

## Branching Model

- `develop` → staging
- `main` → production
