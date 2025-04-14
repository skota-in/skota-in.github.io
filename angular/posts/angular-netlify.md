# Deploying Angular Application to Netlify

This guide will walk you through the process of deploying an Angular application to Netlify.

## Prerequisites

Before you begin, make sure you have the following installed:
- Node.js and npm
- Angular CLI (`npm install -g @angular/cli`)
- Netlify CLI (`npm install -g netlify-cli`)

## Step 1: Build Your Angular Application

First, build your Angular application for production:

```bash
ng build --prod
```

This will create a `dist` folder with your production-ready application.

## Step 2: Initialize Netlify in Your Project

1. Login to Netlify:
```bash
netlify login
```

2. Initialize Netlify in your project:
```bash
netlify init
```

3. During initialization, you'll be prompted to:
   - Link to an existing project or create a new one
   - Set up the project name
   - Configure the build settings

## Step 3: Configure Netlify Deployment

Create a `netlify.toml` file in your project root with the following configuration:

```toml
[build]
  command = "ng build --prod"
  publish = "dist/[your-app-name]"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

## Step 4: Deploy to Netlify

To deploy your application, run:

```bash
netlify deploy --prod
```

After successful deployment, Netlify will provide you with a hosting URL where your application is live.

## Additional Configuration

### Environment Variables

To set up environment variables:

1. Go to your project settings in the Netlify dashboard
2. Navigate to "Build & deploy" > "Environment"
3. Add your environment variables
4. Make sure to add the same variables to your `environment.ts` and `environment.prod.ts` files

### Custom Domain

To set up a custom domain:

1. Go to your project in the Netlify dashboard
2. Click on "Domain settings"
3. Click "Add custom domain"
4. Follow the DNS configuration instructions

## Continuous Deployment

Netlify automatically sets up continuous deployment when you:

1. Connect your GitHub repository
2. Push changes to your main branch
3. Netlify will automatically build and deploy your application

## Build Configuration

You can customize the build process by adding more configuration to your `netlify.toml`:

```toml
[build]
  command = "ng build --prod"
  publish = "dist/[your-app-name]"
  environment = { NODE_VERSION = "16" }

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
    [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
```

## Troubleshooting

Common issues and solutions:

1. **404 Errors**: Ensure your `netlify.toml` has the correct redirects configuration
2. **Build Errors**: Check your Angular build configuration and dependencies
3. **Environment Variables**: Make sure all required environment variables are set in the Netlify dashboard

## Additional Resources

- [Netlify Documentation](https://docs.netlify.com/)
- [Angular Deployment Guide](https://angular.io/guide/deployment)
- [Netlify CLI Reference](https://docs.netlify.com/cli/get-started/)
