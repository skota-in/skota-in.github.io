# Deploying Angular Application to AWS Amplify

*Published on February 5, 2024*

This guide will walk you through the process of deploying an Angular application to AWS Amplify.

## Prerequisites

1. An AWS account
2. Node.js and npm installed
3. Angular CLI installed (`npm install -g @angular/cli`)
4. AWS CLI installed and configured
5. Git installed and configured

## Step 1: Create an Angular Application

If you don't have an existing Angular application, create one:

```bash
ng new my-angular-app
cd my-angular-app
```

## Step 2: Initialize Git Repository

If not already initialized:

```bash
git init
git add .
git commit -m "Initial commit"
```

## Step 3: Push to GitHub

Create a new repository on GitHub and push your code:

```bash
git remote add origin <your-github-repo-url>
git push -u origin main
```

## Step 4: Set Up AWS Amplify

1. Sign in to the AWS Management Console
2. Navigate to AWS Amplify
3. Click "New app" → "Host web app"
4. Select "GitHub" as your repository service
5. Authorize AWS Amplify to access your GitHub account
6. Select your repository and branch
7. Configure build settings:
   - Build command: `npm run build`
   - Output directory: `dist/<your-app-name>`
8. Review and deploy

## Step 5: Configure Environment Variables

If your application requires environment variables:

1. In the Amplify Console, go to your app
2. Navigate to "Environment variables"
3. Add your environment variables
4. Save and redeploy

## Step 6: Custom Domain (Optional)

To add a custom domain:

1. In the Amplify Console, go to "Domain management"
2. Click "Add domain"
3. Follow the instructions to verify domain ownership
4. Configure DNS settings with your domain provider

## Step 7: Continuous Deployment

AWS Amplify automatically sets up continuous deployment:
- Any push to your main branch will trigger a new deployment
- Pull requests will create preview deployments

## Best Practices

1. **Environment Configuration**
   - Use environment.ts for development
   - Use environment.prod.ts for production
   - Configure different environment variables in Amplify

2. **Build Optimization**
   - Enable production mode: `ng build --prod`
   - Use Ahead-of-Time (AOT) compilation
   - Enable build optimization

3. **Caching**
   - Configure appropriate cache headers
   - Use AWS CloudFront for CDN

4. **Security**
   - Enable HTTPS
   - Configure appropriate CORS settings
   - Use AWS WAF if needed

## Troubleshooting

Common issues and solutions:

1. **Build Failures**
   - Check build logs in Amplify Console
   - Verify node version compatibility
   - Ensure all dependencies are in package.json

2. **Deployment Issues**
   - Verify build settings
   - Check environment variables
   - Ensure proper output directory configuration

3. **Performance Issues**
   - Enable compression
   - Configure proper cache headers
   - Optimize assets

## Additional Resources

- [AWS Amplify Documentation](https://docs.aws.amazon.com/amplify/)
- [Angular Deployment Guide](https://angular.io/guide/deployment)
- [AWS Amplify Console](https://console.aws.amazon.com/amplify/home)
