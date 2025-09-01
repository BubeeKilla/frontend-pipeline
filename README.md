# Frontend Pipeline 🚀

This repository (**frontend-pipeline**) contains the **frontend code** and a **CI/CD pipeline** that automatically builds and deploys a Next.js static website to an existing AWS S3 bucket.

---

## 📌 Overview
- The application is written with **Next.js**.  
- On each push to `main`:  
  1. Install dependencies with `npm install`.  
  2. Build the app with `npm run build`.  
  3. Export static files (`out/` directory).  
  4. Sync the build output to the S3 bucket using `aws s3 sync`.  

The pipeline ensures the website is always up-to-date with the latest frontend code.

---

## 🏗 Infrastructure
- The S3 bucket and IAM configuration are **not created here**.  
- They are provisioned separately using Terraform in a dedicated **infra repository**.  
- This repo only manages the **frontend application + deployment pipeline**.  

---

## ⚙️ GitHub Actions Workflow
Location: `.github/workflows/deploy.yml`

```yaml
name: Deploy Frontend to S3

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm install

      - name: Build Next.js (static export)
        run: npm run build

      - name: Deploy to S3
        run: aws s3 sync ./out s3://YOUR_BUCKET_NAME --delete
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ secrets.AWS_REGION }}
```

---

## 🔑 GitHub Secrets
The pipeline requires the following repository secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`

---

## 🌍 Live Deployment
After each push to `main`, the site is rebuilt and deployed to the configured S3 bucket.  
By visiting the bucket’s **static website endpoint**, you’ll always see the latest version of the Next.js app.  

Example URL format:  
```
http://<bucket-name>.s3-website-<region>.amazonaws.com/
```

---

## 📝 Notes
- This pipeline uses **`--delete`** to keep the S3 bucket in sync with the `out/` folder. Old files not in the latest build are automatically removed.  
- Next.js is configured with `"output": "export"` in `next.config.mjs` to support static site generation suitable for S3 hosting.  
- Infrastructure (bucket, IAM, etc.) is managed in a **separate Terraform repo**.  

---

## ✅ Demo Value
This repo demonstrates a clean DevOps separation:
- **Infra repo**: manages cloud resources (Terraform).  
- **Frontend repo**: manages app code + CI/CD deployment.  

Together, they form a production-like workflow that automatically delivers frontend code to AWS.
