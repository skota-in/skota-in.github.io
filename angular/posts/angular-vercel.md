# Deploying Angular Application to Vercel

*Published on February 8, 2024*

This guide will walk you through the process of deploying an Angular application to Vercel.

## Prerequisites

Before you begin, make sure you have the following installed:
- Node.js and npm
- Angular CLI (`npm install -g @angular/cli`)
- Vercel CLI (`npm install -g vercel`)

## Step 1: Build Your Angular Application

First, build your Angular application for production:

```bash
ng build --prod
```

This will create a `dist` folder with your production-ready application.

## Step 2: Initialize Vercel in Your Project

1. Login to Vercel:
```bash
vercel login
```

2. Initialize Vercel in your project:
```bash
vercel
```

3. During initialization, you'll be prompted to:
   - Link to an existing project or create a new one
   - Set up the project name
   - Configure the build settings

## Step 3: Configure Vercel Deployment

Create a `vercel.json` file in your project root with the following configuration:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "dist/[your-app-name]/*",
      "use": "@vercel/static"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ]
}
```

## Step 4: Deploy to Vercel

To deploy your application, run:

```bash
vercel --prod
```

After successful deployment, Vercel will provide you with a hosting URL where your application is live.

## Additional Configuration

### Environment Variables

To set up environment variables:

1. Go to your project settings in the Vercel dashboard
2. Navigate to the "Environment Variables" section
3. Add your environment variables
4. Make sure to add the same variables to your `environment.ts` and `environment.prod.ts` files

### Custom Domain

To set up a custom domain:

1. Go to your project in the Vercel dashboard
2. Click on "Settings"
3. Navigate to "Domains"
4. Add your custom domain
5. Follow the DNS configuration instructions

## Continuous Deployment

Vercel automatically sets up continuous deployment when you:

1. Connect your GitHub repository
2. Push changes to your main branch
3. Vercel will automatically build and deploy your application

## Build Configuration

You can customize the build process by adding a `vercel.json` configuration:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "dist/[your-app-name]"
      }
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ]
}
```

## Troubleshooting

Common issues and solutions:

1. **404 Errors**: Ensure your `vercel.json` has the correct routes configuration
2. **Build Errors**: Check your Angular build configuration and dependencies
3. **Environment Variables**: Make sure all required environment variables are set in the Vercel dashboard

## Additional Resources

- [Vercel Documentation](https://vercel.com/docs)
- [Angular Deployment Guide](https://angular.io/guide/deployment)
- [Vercel CLI Reference](https://vercel.com/docs/cli)
