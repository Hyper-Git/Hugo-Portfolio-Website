# My Portfolio Website

A simple, fast portfolio website built with Hugo and deployed on AWS S3 + CloudFront.

## 🚀 Live Site

[cornelcloud.net](https://cornelcloud.net)

## 🛠 Tech Stack

- **Hugo** - Static site generator
- **AWS S3** - Website hosting (private bucket)
- **CloudFront** - Content delivery network with OAC
- **CloudFront Functions** - URL rewriting and path normalization
- **Route 53** - DNS management
- **Certificate Manager** - SSL/TLS certificate

## 🏗 How to Build This Project

### Architecture Diagram

![Deployment Diagram](./static/images/enhanced_static_website_architecture_with_github_actions_(a3).png)

### Step 1: Install Hugo

**Windows:**

```bash
# Using Chocolatey
choco install hugo-extended

# Or download from https://github.com/gohugoio/hugo/releases
```

**Mac:**

```bash
brew install hugo
```

**Linux:**

```bash
sudo apt install hugo
```

### Step 2: Create Your Hugo Site

```bash
# Create new site
hugo new site my-portfolio
cd my-portfolio

# Add Gugo Narrow Theme
git init
git submodule add https://github.com/tom2almighty/hugo-narrow.git themes/hugo-narrow
git submodule update --init --recursive --remote
echo "theme = 'hugo-narrow'" >> config.toml

# Create your first page
hugo new posts/my-first-post.md

# Start local server
hugo server -D
```

Visit `http://localhost:1313` to see your site!

### Step 3: Customize Your Site

**Edit `config.toml`:**

```toml
baseURL = 'https://cornelcloud.net'
languageCode = 'en-us'
title = 'Your Name - Portfolio'
theme = 'hugo-narrow'

[params]
  description = "Web Developer Portfolio"
  author = "Your Name"
```

**Add content:**

```bash
# Create about page
hugo new about.md

# Create project pages
hugo new projects/project-1.md
hugo new projects/project-2.md
```

**Test locally:**

```bash
hugo server -D
```

### Step 4: Build for Production

```bash
# Build the site
hugo --minify

# This creates a 'public' folder with your website
```

## ☁️ AWS Deployment Guide

### Step 1: Create S3 Bucket

1. **Go to AWS Console → S3**
2. **Click "Create bucket"**
3. **Bucket name:** `your-website-bucket` (must be globally unique)
4. **Region:** Choose closest to your users
5. **Keep "Block all public access" checked** (we'll use CloudFront OAC)
6. **Click "Create bucket"**

### Step 2: Configure S3 Bucket for CloudFront Access

1. **Go to "Permissions" tab**
2. **Keep "Block all public access" enabled** (we'll use CloudFront with OAC instead)
3. **Click "Save changes"**

> Note: We'll no longer use a public bucket policy as we'll secure the bucket with Origin Access Control (OAC)

### Step 3: Upload Your Website

1. **Go to "Objects" tab**
2. **Click "Upload"**
3. **Select all files from your `public/` folder**
4. **Click "Upload"**

### Step 4: Set Up Route 53 for Custom Domain

1. **Go to AWS Console → Route 53**
2. **Click "Hosted zones"**
3. **Click "Create hosted zone"**
4. **Domain name:** Enter your domain (e.g., `cornelcloud.net`)
5. **Type:** Select "Public hosted zone"
6. **Click "Create hosted zone"**
7. **Note the NS (Name Server) records** - You'll need to update these at your domain registrar

### Step 5: Update Domain Registrar

1. **Log in to your domain registrar** (where you purchased your domain)
2. **Find DNS/Nameserver settings**
3. **Replace the nameservers with the four NS values from Route 53**
4. **Save changes** (may take 24-48 hours to propagate)

### Step 6: Request SSL Certificate

1. **Go to AWS Console → Certificate Manager**
2. **Ensure you're in the US East (N. Virginia) region** for CloudFront compatibility
3. **Click "Request certificate"**
4. **Select "Request a public certificate"**
5. **Domain names:** Enter your domain and `*.yourdomain.com` (e.g., `cornelcloud.net` and `*.cornelcloud.net`)
6. **Validation method:** DNS validation
7. **Click "Request"**
8. **Click "Create records in Route 53"** to automatically validate
9. **Wait for certificate status to change to "Issued"**

### Step 7: Create CloudFront Function

1. **Go to AWS Console → CloudFront → Functions**
2. **Click "Create function"**
3. **Name:** `url-rewriter`
4. **Runtime:** JavaScript
5. **Copy this code:**

```javascript
function handler(event) {
    var request = event.request;
    var uri = request.uri;
    
    // Check whether the URI is missing a file name.
    if (uri.endsWith('/')) {
        request.uri += 'index.html';
    } 
    // Check whether the URI is missing a file extension.
    else if (!uri.includes('.')) {
        request.uri += '/index.html';
    }
    
    return request;
}
```

6.**Click "Save changes"**
7.**Click "Publish"**

### Step 8: Create CloudFront Distribution with OAC

1. **Go to AWS Console → CloudFront**
2. **Click "Create distribution"**
3. **Origin domain:** Select your S3 bucket
4. **Origin access:** Select "Origin access control settings (recommended)"**
5. **Create new OAC:**
   - **Name:** `s3-oac-settings`
   - **Sign requests:** Yes
   - **Click "Create"**
6. **Origin path:** Leave empty
7. **Viewer protocol policy:** "Redirect HTTP to HTTPS"
8. **Allowed HTTP methods:** GET, HEAD, OPTIONS
9. **Cache policy:** "Managed-CachingOptimized"
10. **Function associations:**
    - **Viewer request:** Select the `url-rewriter` function
11. **Price class:** Choose based on your needs
12. **Alternate domain names:** Add your custom domain (e.g., `cornelcloud.net`)
13. **Custom SSL certificate:** Select your ACM certificate
14. **Default root object:** `index.html`
15. **Click "Create distribution"**

**Wait 10-15 minutes** for deployment to complete.

### Step 9: Update S3 Bucket Policy

After creating the CloudFront distribution with OAC, you'll see a policy warning in the CloudFront console. Click "Copy Policy" and:

1. **Go to S3 Console → your bucket**
2. **Go to "Permissions" tab**
3. **Click "Bucket policy"**
4. **Paste the policy** (it will look similar to this):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-website-bucket/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::your-account-id:distribution/your-distribution-id"
                }
            }
        }
    ]
}
```

5.**Click "Save changes"**

### Step 10: Create Route 53 Record for CloudFront

1. **Go to Route 53 → Hosted zones → Your domain**
2. **Click "Create record"**
3. **Record name:** Leave empty for root domain
4. **Record type:** A
5. **Route traffic to:** "Alias to CloudFront distribution"
6. **Choose your distribution** from the dropdown
7. **Click "Create records"**
8. **Repeat for www subdomain** (if desired)
   - **Record name:** `www`
   - Follow the same steps as above

### Step 11: Update Your Site

When you make changes:

1. **Build your site:**

   ```bash
   hugo --minify
   ```

2. **Upload to S3:**
   - Go to S3 Console
   - Select your bucket
   - Upload new files (overwrite existing)
   Or use AWS CLI for faster uploads:

```bash
   aws s3 sync public/ s3://your-website-bucket/ --delete
   ```

3.**Invalidate CloudFront cache:**

- Go to CloudFront Console
- Select your distribution
- Go to "Invalidations" tab
- Click "Create invalidation"
- Enter `/*` to invalidate everything
- Click "Create invalidation"
   Or use AWS CLI:

```bash
  aws cloudfront create-invalidation --distribution-id YOUR_DISTRIBUTION_ID --paths "/*"
   ```

## 🔧 Development Workflow

### Daily Development

```bash
# Start local server
hugo server -D

# Make changes to content or layouts
# Test in browser at localhost:1313

# When ready to deploy
hugo --minify
# Upload public/ folder to S3
# Create CloudFront invalidation
```

### Adding New Content

```bash
# New blog post
hugo new posts/my-new-post.md

# New project
hugo new projects/my-project.md

# Edit the created markdown file
# Test locally, then deploy
```

## 📊 Performance & Costs

### Performance Benefits

- **Fast loading** - Static files served from global CDN
- **Scalable** - Handles traffic spikes automatically
- **Secure** - HTTPS by default with CloudFront

### Monthly Costs (Estimated)

- **S3 Storage:** $1-3
- **CloudFront:** $1-5 (depending on traffic)
- **Route 53:** $0.50 per hosted zone + $0.40 per million queries
- **Certificate Manager:** Free for CloudFront distributions
- **CloudFront Functions:** ~$0.10 (first 2M requests/month free)
- **Total:** ~$3-10/month

## 🔗 Useful Commands

```bash
# Create new content
hugo new posts/title.md

# Start development server
hugo server -D

# Build for production
hugo --minify

# Check Hugo version
hugo version
```

## 📞 Contact

**Your Name**  
Email: <cornelbacanu@gmail.com>  
Portfolio: [cornelcloud.net](https://cornelcloud.net)

---

## 💡 What This Project Demonstrates

- **Static Site Generation** with Hugo
- **AWS Cloud Services** (S3, CloudFront, Route 53, Certificate Manager)
- **Modern Web Development** practices
- **Performance Optimization** through CDN
- **Security Best Practices** with private S3 buckets and OAC
- **URL Rewriting** with CloudFront Functions
- **Cost-Effective Hosting** solution

This project shows how to build and deploy a professional website using modern tools and cloud infrastructure with enhanced security and performance.

---

## 🤖 Automate Deployment with GitHub Actions (CI/CD) using OIDC

Automate your deployment process so that every time you push a change to your `main` branch, your website is automatically built and deployed to AWS. This guide uses OpenID Connect (OIDC) to securely authenticate your GitHub Actions workflow with AWS, eliminating the need for long-lived access keys.

### Step 1: Create an OIDC Identity Provider in AWS IAM

1. **Go to AWS Console → IAM → Identity providers.**
2. **Click "Add provider".**
3. **Provider type:** Select "OpenID Connect".
4. **Provider URL:** `https://token.actions.githubusercontent.com`
5. **Audience:** `sts.amazonaws.com`
6. **Click "Get thumbprint"** to verify the provider.
7. **Click "Add provider".**

### Step 2: Create an IAM Role for GitHub Actions

1. **Go to IAM → Roles → Create role.**
2. **Trusted entity type:** Select "Web identity".
3. **Identity provider:** Choose the `token.actions.githubusercontent.com` provider you just created.
4. **Audience:** Select `sts.amazonaws.com`.
5. **GitHub organization/repository:** To restrict access, you can specify your GitHub organization or repository here (e.g., `my-github-username/my-repo`). For this guide, we'll allow any repository in your account, but you can tighten this later.
6. **Click "Next".**
7. **Permissions policies:** Click "Create policy" and use the JSON editor to paste the following policy. Replace `your-website-bucket` with your actual S3 bucket name.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowS3Sync",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:ListBucket",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::your-website-bucket",
                "arn:aws:s3:::your-website-bucket/*"
            ]
        },
        {
            "Sid": "AllowCloudFrontInvalidation",
            "Effect": "Allow",
            "Action": "cloudfront:CreateInvalidation",
            "Resource": "*"
        }
    ]
}
```

8.**Finish creating the policy, then attach it to the role.**
9.  **Role name:** `github-actions-deployer-role` (or a name of your choice).
10. **Click "Create role".**
11. **Copy the ARN of the role you just created.** You'll need it for your GitHub workflow.

### Step 3: Add Secrets to GitHub

1. In your GitHub repository, go to **Settings > Secrets and variables > Actions**.
2. Click **"New repository secret"** for each of the following:
    - `AWS_ROLE_TO_ASSUME`: The ARN of the IAM role you created (e.g., `arn:aws:iam::123456789012:role/github-actions-deployer-role`).
    - `AWS_S3_BUCKET`: The name of your S3 bucket (e.g., `your-website-bucket`).
    - `CLOUDFRONT_DISTRIBUTION_ID`: The ID of your CloudFront distribution.

### Step 4: Create the GitHub Actions Workflow File

1. In your project's root directory, create a new folder structure: `.github/workflows/`.
2. Inside the `workflows` folder, create a new file named `deploy.yml`.
3. Paste the following code into `deploy.yml`:

```yaml
name: Deploy Hugo Site to AWS S3 (OIDC)

on:
  push:
    branches:
      - main # Or your default branch

permissions:
  id-token: write # Required for OIDC
  contents: read

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: 'recursive' # Important for themes

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build Hugo site
        run: hugo --minify

      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1 # Change to your bucket's region

      - name: Deploy to S3
        run: |
          aws s3 sync public/ s3://${{ secrets.AWS_S3_BUCKET }}/ --delete

      - name: Invalidate CloudFront Cache
        run: |
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```

Now, every time you push changes to your `main` branch, this workflow will securely authenticate with AWS using OIDC, build your Hugo site, upload the files to S3, and invalidate the CloudFront cache.
