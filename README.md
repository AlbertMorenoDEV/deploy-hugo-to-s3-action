# Deploy Hugo To S3 Action

This GitHub action makes it easy to build and deploy any Hugo static website to AWS S3 with just one step.

## Authentication

This action supports two authentication methods:

1.  **OpenID Connect (OIDC):** This is the recommended authentication method, as it is more secure and does not require storing long-lived access keys as secrets.
2.  **AWS Access Keys:** This method is less secure, but it is easier to set up.

### OpenID Connect (OIDC)

To use OIDC, you need to create an IAM role that can be assumed by GitHub Actions.

#### 1. Create an IAM Role

Create an IAM role in your AWS account. In the "Select trusted entity" step, choose "Web identity" and configure it as follows:

- **Identity provider:** Choose the identity provider for your GitHub organization or user. If you don't have one, you can create one by following the instructions in the [AWS documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html).
- **Audience:** `sts.amazonaws.com`

#### 2. Attach a Policy to the Role

Attach a policy to the role that grants the necessary permissions to deploy to your S3 bucket. Here is an example policy:

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
      - master
  pull_request:
    branches:
      - master

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Check out master
        uses: actions/checkout@master

      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.0.11
        with:
          hugo-version: 0.139.0
          aws-role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
          aws-region: us-east-1
```

### AWS Access Keys

To use AWS access keys, you need to create an IAM user with the required permissions.

#### 1. Create an IAM User

Create an IAM user in your AWS account. Attach the same policy as for the OIDC role to this user.

#### 2. Configure the GitHub Workflow

In your GitHub workflow, add the `aws-access-key-id` and `aws-secret-access-key` as secrets and pass them to the action.

```yaml
---
name: Hugo Build and Deploy to S3

on:
  push:
    branches:
      - master
  pull_request:
    branches:
      - master

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
      - name: Check out master
        uses: actions/checkout@master

      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.0.11
        with:
          hugo-version: 0.139.0
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
```

## Custom Config File

Optional `config` to specify a custom config file. Handy for Hugo sites that have more than one config file.

### Example

#### GitHub Action

```yaml
---
name: Hugo Build and Deploy to S3

on:
  push:
    branches:
      - master
  pull_request:
    branches:
      - master

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Check out master
        uses: actions/checkout@master

      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.0.11
        with:
          hugo-version: 0.139.0
          config: path/config.toml
          aws-role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
          aws-region: us-east-1
```

## Targets

Optional `input` to specify a deployment target. Handy for Hugo sites that have multiple environments (e.g. "staging" and "production").

If `target` is not specified in your GitHub Action, the first deployment target is used by default.

### Example

#### GitHub Action

```yaml
name: Hugo Build and Deploy to S3

on:
  push:
    branches:
      - master
  pull_request:
    branches:
      - master

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Check out master
        uses: actions/checkout@master

      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.0.11
        with:
          hugo-version: 0.139.0
          target: production
          aws-role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
          aws-region: us-east-1
```

#### Hugo Config

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
