# Deploy Hugo To S3 Action

This GitHub action makes it easy to build and deploy any Hugo static website to AWS S3 with just one step.

## Example

```
name: Hugo Build and Deploy to S3

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
      - name: Check out master
        uses: actions/checkout@master
      
      - name: Build and deploy
        uses: AlbertMorenoDEV/deploy-hugo-to-s3-action@v0.0.3
        with:
          hugo-version: 0.85.0
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```