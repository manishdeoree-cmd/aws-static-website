# 🚀 Static Website Hosting on AWS

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws)
![S3](https://img.shields.io/badge/Amazon_S3-Storage-green?logo=amazon-s3)
![CloudFront](https://img.shields.io/badge/CloudFront-CDN-blue?logo=amazon-aws)
![Route53](https://img.shields.io/badge/Route_53-DNS-purple?logo=amazon-aws)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-black?logo=github-actions)
![Status](https://img.shields.io/badge/Status-Live-brightgreen)

A production-grade static website hosted on AWS using **S3**, **CloudFront**, **Route 53**, **ACM**, and automated deployment via **GitHub Actions CI/CD pipeline**.

> 🌐 **Live Demo:** [file:///F:/aws-static-website/aws-static-website/src/index.html)

---

## 📌 Project Overview

This project demonstrates how to host a static website on AWS with a fully automated CI/CD pipeline. Every time code is pushed to the `main` branch, GitHub Actions automatically builds and deploys the site to S3 and invalidates the CloudFront cache — so changes go live in seconds.

---

## 🏗️ Architecture

```
User (Browser)
      │
      ▼
 ┌─────────────┐
 │  Route 53   │  ← Custom domain DNS routing
 └──────┬──────┘
        │
        ▼
 ┌─────────────────────────┐
 │      CloudFront CDN     │  ← HTTPS, caching, global edge locations
 │   + ACM SSL Certificate │
 └──────────┬──────────────┘
            │
            ▼
     ┌─────────────┐
     │  S3 Bucket  │  ← Static files (HTML, CSS, JS, images)
     └─────────────┘

━━━━━━━━━━ CI/CD Pipeline ━━━━━━━━━━

 GitHub Repo
      │  git push
      ▼
 GitHub Actions
      │  Build + Test
      ▼
 S3 Sync (aws s3 sync)
      │
      ▼
 CloudFront Invalidation
```

---

## ☁️ AWS Services Used

| Service | Purpose |
|---------|---------|
| **Amazon S3** | Stores and serves static website files (HTML, CSS, JS) |
| **Amazon CloudFront** | CDN for fast global delivery + HTTPS termination |
| **Amazon Route 53** | Custom domain DNS management |
| **AWS Certificate Manager (ACM)** | Free SSL/TLS certificate for HTTPS |
| **AWS IAM** | Least-privilege user for GitHub Actions deployments |

---

## 🔄 CI/CD Pipeline

Every push to `main` branch triggers the GitHub Actions workflow:

```
git push → GitHub Actions → Build → S3 Sync → CloudFront Invalidation → Live ✅
```

**Workflow steps:**
1. Checkout source code
2. Build the static site (if using React/Hugo/Jekyll)
3. Configure AWS credentials via GitHub Secrets
4. Sync files to S3 bucket
5. Invalidate CloudFront cache so changes are live immediately

---

## 📁 Project Structure

```
├── .github/
│   └── workflows/
│       └── deploy.yml        # GitHub Actions CI/CD pipeline
├── src/
│   ├── index.html            # Main HTML file
│   ├── style.css             # Stylesheet
│   └── script.js             # JavaScript
├── assets/
│   └── images/               # Images and static assets
└── README.md
```

---

## ⚙️ GitHub Actions Workflow

`.github/workflows/deploy.yml`

```yaml
name: Deploy to AWS S3

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to S3
        run: |
          aws s3 sync ./src s3://${{ secrets.S3_BUCKET_NAME }} \
            --delete \
            --cache-control max-age=86400

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

---

## 🔐 IAM Policy (Least Privilege)

The GitHub Actions IAM user only has the minimum permissions required:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 🚀 Setup Guide

### Step 1 — Create S3 Bucket

```bash
# Create bucket
aws s3api create-bucket \
  --bucket your-bucket-name \
  --region us-east-1

# Enable static website hosting
aws s3 website s3://your-bucket-name/ \
  --index-document index.html \
  --error-document error.html

# Upload files
aws s3 sync ./src s3://your-bucket-name
```

### Step 2 — Create CloudFront Distribution

1. Go to **AWS Console → CloudFront → Create Distribution**
2. Set **Origin Domain** to your S3 bucket
3. Enable **Redirect HTTP to HTTPS**
4. Set **Default Root Object** to `index.html`
5. Attach your **ACM SSL Certificate**

### Step 3 — Request SSL Certificate (ACM)

```bash
aws acm request-certificate \
  --domain-name your-domain.com \
  --validation-method DNS \
  --region us-east-1
```

### Step 4 — Configure Route 53

1. Go to **Route 53 → Hosted Zones → Create Record**
2. Set **Record type** to `A`
3. Enable **Alias** → point to your CloudFront distribution

### Step 5 — Add GitHub Secrets

Go to your GitHub repo → **Settings → Secrets and variables → Actions** and add:

| Secret Name | Value |
|-------------|-------|
| `AWS_ACCESS_KEY_ID` | Your IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | Your IAM user secret key |
| `S3_BUCKET_NAME` | Your S3 bucket name |
| `CLOUDFRONT_DISTRIBUTION_ID` | Your CloudFront distribution ID |

### Step 6 — Push and Deploy

```bash
git add .
git commit -m "Deploy static website"
git push origin main
# GitHub Actions will automatically deploy 🚀
```

---

## 💰 Estimated AWS Cost

| Service | Free Tier | Estimated Cost |
|---------|-----------|----------------|
| S3 Storage | 5 GB free | ~$0.023/GB/month |
| CloudFront | 1 TB free | ~$0.0085/GB after |
| Route 53 | — | $0.50/hosted zone/month |
| ACM Certificate | Free | $0 |
| **Total (small site)** | | **~$0.50–$1/month** |

---

## 📊 Key Features

- ✅ **HTTPS** enabled with free ACM SSL certificate
- ✅ **Global CDN** via CloudFront edge locations
- ✅ **Custom domain** with Route 53 DNS
- ✅ **Automated CI/CD** — deploy on every git push
- ✅ **Cache invalidation** — changes go live instantly
- ✅ **IAM least privilege** — secure deployment credentials
- ✅ **Cost optimized** — nearly free for small sites

---

## 🛠️ Tech Stack

- **Cloud:** AWS (S3, CloudFront, Route 53, ACM, IAM)
- **CI/CD:** GitHub Actions
- **Frontend:** HTML, CSS, JavaScript
- **IaC:** AWS CLI / AWS Console

---

## 👨‍💻 Author

**Manish Deore**

DevOps Engineer | AWS | Docker | Kubernetes | Terraform | Linux

[![LinkedIn](https://linkedin.com/in/manish-deore )
[![GitHub](https://github.com/manishdeoree-cmd
)

---

⭐ **Star this repository if it helped you!**
