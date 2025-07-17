# My Portfolio Website

A simple, fast portfolio website built with Hugo and deployed on AWS S3 + CloudFront.

## 🚀 Live Site

[your-domain.com](https://your-domain.com)

## 🛠 Tech Stack

- **Hugo** - Static site generator
- **AWS S3** - Website hosting
- **CloudFront** - Content delivery network
- **Route 53** - DNS management

## 🏗 How to Build This Project

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
echo "theme = 'ananke'" >> config.toml

# Create your first page
hugo new posts/my-first-post.md

# Start local server
hugo server -D
```

Visit `http://localhost:1313` to see your site!

### Step 3: Customize Your Site

**Edit `config.toml`:**

```toml
baseURL = 'https://your-domain.com'
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
5. **Uncheck "Block all public access"**
6. **Click "Create bucket"**

### Step 2: Configure S3 for Website Hosting

1. **Click on your bucket**
2. **Go to "Properties" tab**
3. **Scroll to "Static website hosting"**
4. **Click "Edit"**
5. **Enable static website hosting**
6. **Index document:** `index.html`
7. **Error document:** `404.html`
8. **Click "Save changes"**

### Step 3: Set Bucket Policy

1. **Go to "Permissions" tab**
2. **Click "Bucket policy"**
3. **Add this policy** (replace `your-website-bucket`):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-website-bucket/*"
        }
    ]
}
```

### Step 4: Upload Your Website

1. **Go to "Objects" tab**
2. **Click "Upload"**
3. **Select all files from your `public/` folder**
4. **Click "Upload"**

Your site is now live at the S3 website URL!

### Step 5: Create CloudFront Distribution

1. **Go to AWS Console → CloudFront**
2. **Click "Create distribution"**
3. **Origin domain:** Select your S3 bucket
4. **Origin path:** Leave empty
5. **Viewer protocol policy:** "Redirect HTTP to HTTPS"
6. **Allowed HTTP methods:** GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
7. **Cache policy:** "Managed-CachingOptimized"
8. **Price class:** Choose based on your needs
9. **Alternate domain names:** Add your custom domain (optional)
10. **Click "Create distribution"**

**Wait 10-15 minutes** for deployment to complete.

### Step 6: Update Your Site

When you make changes:

1. **Build your site:**

   ```bash
   hugo --minify
   ```

2. **Upload to S3:**
   - Go to S3 Console
   - Select your bucket
   - Upload new files (overwrite existing)

3. **Invalidate CloudFront cache:**
   - Go to CloudFront Console
   - Select your distribution
   - Go to "Invalidations" tab
   - Click "Create invalidation"
   - Enter `/*` to invalidate everything
   - Click "Create invalidation"

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
- **Route 53:** $0.50 (if using custom domain)
- **Total:** ~$3-9/month

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
Email: <your.email@example.com>  
Portfolio: [your-domain.com](https://your-domain.com)

---

## 💡 What This Project Demonstrates

- **Static Site Generation** with Hugo
- **AWS Cloud Services** (S3, CloudFront, Route 53)
- **Modern Web Development** practices
- **Performance Optimization** through CDN
- **Cost-Effective Hosting** solution

This project shows how to build and deploy a professional website using modern tools and cloud infrastructure.
