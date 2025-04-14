# Deploying Angular Application to GitHub Pages

This guide will walk you through the process of deploying an Angular application to GitHub Pages.

## Prerequisites

Before you begin, make sure you have the following installed:
- Node.js and npm
- Angular CLI (`npm install -g @angular/cli`)
- Git

## Step 1: Create a GitHub Repository

1. Create a new repository on GitHub
2. Clone the repository to your local machine:
```bash
git clone https://github.com/your-username/your-repo-name.git
```

## Step 2: Configure Angular Project

1. Create a new Angular project or use an existing one
2. Update the `angular.json` file to set the base href for GitHub Pages:

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "build": {
          "options": {
            "baseHref": "/your-repo-name/"
          }
        }
      }
    }
  }
}
```

## Step 3: Build Your Angular Application

Build your Angular application for production:

```bash
ng build --prod
```

This will create a `dist` folder with your production-ready application.

## Step 4: Deploy to GitHub Pages

### Method 1: Using Angular CLI (Recommended)

1. Install the `angular-cli-ghpages` package:
```bash
npm install -g angular-cli-ghpages
```

2. Deploy your application:
```bash
ng deploy --base-href=/your-repo-name/
```

### Method 2: Manual Deployment

1. Create a new branch called `gh-pages`:
```bash
git checkout -b gh-pages
```

2. Copy the contents of the `dist/[your-app-name]` folder to the root of your repository

3. Commit and push the changes:
```bash
git add .
git commit -m "Deploy to GitHub Pages"
git push origin gh-pages
```

## Step 5: Configure GitHub Pages

1. Go to your repository on GitHub
2. Click on "Settings"
3. Scroll down to "GitHub Pages" section
4. Under "Source", select the `gh-pages` branch
5. Click "Save"

Your application will be available at: `https://your-username.github.io/your-repo-name/`

## Additional Configuration

### Custom Domain

To set up a custom domain:

1. Create a `CNAME` file in your repository root with your domain name
2. Update your DNS settings to point to GitHub Pages
3. In GitHub repository settings, add your custom domain

### Environment Variables

For environment variables:

1. Create separate environment files for development and production
2. Use Angular's environment system to manage different configurations

## Continuous Deployment

To set up continuous deployment:

1. Create a GitHub Actions workflow file (`.github/workflows/deploy.yml`):

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'

      - name: Install dependencies
        run: npm install

      - name: Build
        run: ng build --prod

      - name: Deploy
        uses: JamesIves/github-pages-deploy-action@4.1.4
        with:
          branch: gh-pages
          folder: dist/[your-app-name]
```

## Troubleshooting

Common issues and solutions:

1. **404 Errors**: 
   - Make sure the base href is correctly set
   - Check if the repository name in the URL matches your actual repository name

2. **Build Errors**:
   - Ensure all dependencies are properly installed
   - Check Angular version compatibility

3. **Deployment Errors**:
   - Verify GitHub Pages is enabled in repository settings
   - Check if the `gh-pages` branch exists

## Additional Resources

- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Angular Deployment Guide](https://angular.io/guide/deployment)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
