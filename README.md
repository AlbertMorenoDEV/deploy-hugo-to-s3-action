# Deploy Hugo To S3 Action

This GitHub Action makes it easy to build and deploy any Hugo static website to AWS S3 with just one step.

## Features

- **Portable:** Works on Linux, Windows, and macOS runners with `x64` and `ARM64` architectures (supporting `.tar.gz`, `.zip`, and macOS universal `.pkg` releases).
- **Secure:** Supports OpenID Connect (OIDC) for authentication without storing long-lived AWS access keys, and hardens all shell steps against script injection.
- **Flexible:** Supports both OIDC and static AWS Access Keys, custom `working-directory` for monorepos, Hugo Modules (`go.mod`) and Git submodules, and `hugo-version: latest`.
- **Fast:** Automatic caching of the resolved Hugo binary (`actions/cache@v6`) speeds up workflow runs.
- **Observable:** Exports action outputs (`hugo-version-installed`, `build-path`) and publishes a clean deployment table to the GitHub Actions Step Summary.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `hugo-version` | Hugo version to use (e.g., `0.169.0` or `latest`) | No | `latest` |
| `aws-access-key-id` | AWS Access Key ID | No | |
| `aws-secret-access-key` | AWS Secret Access Key | No | |
| `aws-role-to-assume` | AWS Role to Assume (OIDC) | No | |
| `aws-region` | AWS Region (required when using `aws-role-to-assume`) | No | |
| `target` | Hugo deployment target | No | `""` |
| `config` | Custom Hugo config file path | No | `""` |
| `environment` | Hugo environment (e.g., `production`) | No | `""` |
| `skip-deployment` | Skip the deployment step (`true` / `false`) | No | `false` |
| `build-args` | Additional arguments for `hugo` build | No | `""` |
| `deploy-args` | Additional arguments for `hugo deploy` | No | `""` |
| `destination` | Build output directory | No | `public` |
| `working-directory` | Working directory for the Hugo site | No | `.` |

## Outputs

| Output | Description |
|--------|-------------|
| `hugo-version-installed` | The resolved Hugo version that was installed and executed (e.g., `0.169.0`) |
| `build-path` | Absolute path to the built Hugo site output directory |

## Authentication

This action supports two authentication methods:

1. **OpenID Connect (OIDC):** Recommended, as it uses short-lived credentials and does not require storing long-lived access keys as secrets.
2. **AWS Access Keys:** Uses static `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` secrets.

### OpenID Connect (OIDC)

To use OIDC, create an IAM role that can be assumed by GitHub Actions.

#### 1. Create an IAM Role

Create an IAM role in your AWS account. In the "Select trusted entity" step, choose "Web identity" and configure it as follows:

- **Identity provider:** Choose the identity provider for your GitHub organization or user (`token.actions.githubusercontent.com`). If you don't have one, follow the [AWS documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html).
- **Audience:** `sts.amazonaws.com`

#### 2. Attach a Policy to the Role

Attach a policy to the role that grants the necessary permissions to deploy to your S3 bucket (and optionally invalidate CloudFront):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DeployToS3",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:DeleteObject",
                "s3:GetBucketLocation"
            ],
            "Resource": [
                "arn:aws:s3:::your-bucket-name",
                "arn:aws:s3:::your-bucket-name/*"
            ]
        },
        {
            "Sid": "InvalidateCloudFront",
            "Effect": "Allow",
            "Action": [
                "cloudfront:CreateInvalidation"
            ],
            "Resource": "arn:aws:cloudfront::123456789012:distribution/your-distribution-id"
        }
    ]
}
```

**Note:** Replace `your-bucket-name` and `your-distribution-id` with your actual bucket name and CloudFront distribution ID.

#### 3. Configure the GitHub Workflow

In your GitHub workflow, add the `permissions` block to the job and configure the `aws-role-to-assume` and `aws-region` inputs in the action.

```yaml
---
name: Hugo Build and Deploy to S3

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Check out repository
        uses: actions/checkout@v6

      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.1.0
        with:
          hugo-version: latest
          aws-role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
          aws-region: us-east-1
```

### AWS Access Keys

To use AWS access keys, create an IAM user with the same permissions policy shown above, store `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in your repository secrets, and pass them to the action:

```yaml
---
name: Hugo Build and Deploy to S3

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v6

      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.1.0
        with:
          hugo-version: 0.169.0
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## Custom Config File & Working Directory

Use `config` to specify a custom config file, or `working-directory` when your Hugo project lives in a subdirectory.

```yaml
      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.1.0
        with:
          hugo-version: 0.169.0
          working-directory: site
          config: config/production.toml
          aws-role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
          aws-region: us-east-1
```

## Targets

Optional `target` input to specify a deployment target defined in your Hugo configuration (e.g. `staging` and `production`). If `target` is omitted, Hugo uses the first target by default.

### GitHub Workflow

```yaml
      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.1.0
        with:
          hugo-version: latest
          target: production
          aws-role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
          aws-region: us-east-1
```

### Hugo Config

```yaml
---
baseURL: /
languageCode: en-us
title: Example Site

# -------------- AWS S3 Deployment Configs -------------- #

deployment:
  targets:
    - name: "staging"
      URL: "s3://stg.example.com?region=us-east-2"
    - name: "production"
      URL: "s3://example.com?region=us-east-1"
```
