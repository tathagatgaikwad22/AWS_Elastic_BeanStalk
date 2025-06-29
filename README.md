# 🚀 AWS Elastic Beanstalk CI/CD with GitHub Actions

Automate deployment to **AWS Elastic Beanstalk** whenever you push your code—no manual steps needed. Just configure, push, and let GitHub Actions handle the rest.

---

## 🧩 Table of Contents

1. [Project Overview](#project-overview)  
2. [Features](#features)  
3. [Prerequisites](#prerequisites)  
4. [Setup Guide](#setup-guide)  
   - [AWS Configuration](#aws-configuration)  
   - [GitHub Secrets](#github-secrets)  
   - [Workflow Configuration](#workflow-configuration)  
5. [How It Works](#how-it-works)  
6. [Customizations](#customizations)  
7. [Troubleshooting](#troubleshooting)  
8. [Contributing](#contributing)  
9. [License](#license)

---

## 📌 Project Overview

This repository configures a workflow that:

- Builds and packages your app
- Uploads it to S3
- Creates a new version in Elastic Beanstalk
- Rolls out the latest version to your running environment

Every push to `main` (or a configured branch) is automatically deployed—making deployment fast, repeatable, and consistent.

---

## 🌟 Features

- **Full automation** of deployment process  
- **Versioning** with Git SHA labels  
- **S3 integration** for application package storage  
- **AWS CLI via GitHub Actions** for seamless application versioning  
- **Modular**: easily adapt to Docker, Java, Python, .NET, etc.

---

## ✅ Prerequisites

- An AWS account with:
  - An **Elastic Beanstalk application & environment**
  - An **S3 bucket** for storing deployment packages

- A GitHub repository with:
  - This project’s files
  - A `.github/workflows/deploy.yml` workflow

- GitHub repository secrets:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
  - Optional: `AWS_REGION` (default is `us-east-1`)

---

## 🛠️ Setup Guide

### AWS Configuration

1. Create or use an existing Elastic Beanstalk application & environment (e.g., `my-app-prod-env`)  
2. Create an S3 bucket (e.g., `my-app-deploys`) to store application versions

---

### GitHub Secrets

Go to **Settings → Secrets and Variables → Actions** in your repo and add:

- `AWS_ACCESS_KEY_ID`: IAM user with permissions for EB and S3  
- `AWS_SECRET_ACCESS_KEY`  
- `AWS_REGION`: (optional) region of your EB environment (e.g., `us-east-1`)

---

### Workflow Configuration

Use this `deploy.yml` inside `.github/workflows/`:

```yaml
name: deploy-to-eb

on:
  push:
    branches:
      - main  # Change this if needed

jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Build & package
        run: |
          # Update this for your project's build commands
          zip -r deploy_package.zip ./

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION || 'us-east-1' }}

      - name: Upload to S3
        run: |
          aws s3 cp deploy_package.zip s3://YOUR_S3_BUCKET/

      - name: Create EB application version
        run: |
          aws elasticbeanstalk create-application-version \
            --application-name YOUR_EB_APP \
            --source-bundle S3Bucket="YOUR_S3_BUCKET",S3Key="deploy_package.zip" \
            --version-label "ver-${{ github.sha }}" \
            --description "Commit ${{ github.sha }}"

      - name: Deploy to Elastic Beanstalk
        run: |
          aws elasticbeanstalk update-environment \
            --environment-name YOUR_EB_ENV \
            --version-label "ver-${{ github.sha }}"

Make sure to replace:

YOUR_S3_BUCKET

YOUR_EB_APP

YOUR_EB_ENV

⚙️ How It Works
You push code to main.

GitHub Actions triggers deploy-to-eb job.

App files are zipped.

Uploads the zip to your S3 bucket.

Creates a new Elastic Beanstalk application version labeled with the Git commit SHA.

Updates your Elastic Beanstalk environment with that version, triggering deployment.

🔧 Customizations
Docker apps: use action einaregilsson/beanstalk-deploy@v22

Language-specific builds:

Node.js: npm install && npm run build

Python: pip install -r requirements.txt

Java: add actions/setup-java and build JAR/WAR

Multiple environments: create separate workflows for staging & production

Rollback support: extend workflows with error handling and cleanup steps

🐞 Troubleshooting
Problem	Potential Fix
Permission errors	Verify IAM user has policies: elasticbeanstalk:*, s3:*
S3 errors	Make sure YOUR_S3_BUCKET exists in the correct region
EB version issues	Ensure version-label is unique on each run
Deployment fails in EB	Check EB logs in AWS Console → Logs → Last 100 lines

🤝 Contributing
Contributions are welcome! Some ideas:

Add rollback steps on failure

Support for Docker, multi-stage workflows, more languages

Add tests to validate that deployment succeeded

Submit a pull request or open an issue—happy to collaborate!

📝 License
Distributed under the MIT License. See LICENSE for details.
