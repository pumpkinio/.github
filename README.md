# pumpkinio/.github

Shared GitHub Actions reusable workflows and composite actions for the Pumpkin platform.

## Versioning

Workflows are versioned with semver tags. Pin to a major version for stability:

```yaml
uses: pumpkinio/.github/.github/workflows/node-ci.yml@v1
```

## Reusable Workflows

| Workflow | Purpose | Key Inputs |
|----------|---------|------------|
| `node-ci.yml` | Lint, typecheck, test Node.js projects | `node-version`, `test-command`, `lint-command` |
| `security-scan.yml` | npm audit + TruffleHog secret scanning | `audit-level`, `scan-secrets` |
| `terraform-deploy.yml` | OpenTofu plan/apply with OIDC | `environment`, `working-directory`, `plan-only` |
| `db-migrate.yml` | Run SQL migrations via Aurora Data API | `environment`, `migrations-directory` |
| `lambda-deploy.yml` | Build and deploy Lambda functions | `environment`, `functions-directory`, `function-prefix` |
| `s3-cloudfront-deploy.yml` | Build and deploy frontend to S3 + CloudFront | `environment`, `working-directory`, `build-command` |

## Composite Actions

| Action | Purpose | Key Inputs |
|--------|---------|------------|
| `actions/setup-node` | Setup Node.js with npm cache | `node-version`, `working-directory` |
| `actions/configure-aws` | OIDC-based AWS authentication | `role-arn`, `aws-region` |

## Usage Examples

### CI Pipeline (caller workflow)

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

jobs:
  ci:
    uses: pumpkinio/.github/.github/workflows/node-ci.yml@v1
    with:
      node-version: '20'
      test-command: 'npm test'
      lint-command: 'npm run lint'

  security:
    uses: pumpkinio/.github/.github/workflows/security-scan.yml@v1
    with:
      audit-level: 'high'
```

### Deploy Pipeline (caller workflow)

```yaml
name: Deploy
on:
  workflow_dispatch:

jobs:
  deploy-api:
    uses: pumpkinio/.github/.github/workflows/lambda-deploy.yml@v1
    with:
      environment: prod
      functions-directory: services/api
      function-prefix: 'myapp-prod-'
    secrets:
      aws-role-arn: ${{ secrets.PROD_AWS_ROLE_ARN }}
```

### Infrastructure (caller workflow)

```yaml
name: Infrastructure
on:
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    uses: pumpkinio/.github/.github/workflows/terraform-deploy.yml@v1
    with:
      environment: prod
      working-directory: terraform/prod
    secrets:
      aws-role-arn: ${{ secrets.PROD_AWS_ROLE_ARN }}
      tf-sensitive-vars: ${{ secrets.PROD_TF_SENSITIVE_VARS }}
```

## Security

- All third-party actions are pinned to commit SHAs
- AWS authentication uses OIDC (no long-lived credentials)
- Prod deployments are gated by GitHub Environment protection rules

## Contributing

1. Create a branch
2. Make changes
3. PR to `main` — requires CODEOWNERS review
4. After merge, create a release via the Release workflow dispatch
