# Deploying Angular Application to Firebase

This guide will walk you through the process of deploying an Angular application to Firebase Hosting.

## Prerequisites

Before you begin, make sure you have the following installed:
- Node.js and npm
- Angular CLI (`npm install -g @angular/cli`)
- Firebase CLI (`npm install -g firebase-tools`)

## Step 1: Build Your Angular Application

First, build your Angular application for production:

```bash
ng build --prod
```

This will create a `dist` folder with your production-ready application.

## Step 2: Initialize Firebase in Your Project

1. Login to Firebase:
```bash
firebase login
```

2. Initialize Firebase in your project:
```bash
firebase init
```

3. During initialization, select the following options:
   - Select "Hosting: Configure files for Firebase Hosting"
   - Select your Firebase project (or create a new one)
   - For the public directory, enter: `dist/[your-app-name]`
   - Configure as a single-page app: Yes
   - Set up automatic builds and deploys: No (unless you want to use GitHub integration)

## Step 3: Configure Firebase Hosting

The `firebase.json` file will be created in your project root. Make sure it looks like this:

```json
{
  "hosting": {
    "public": "dist/[your-app-name]",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

## Step 4: Deploy to Firebase

To deploy your application, run:

```bash
firebase deploy
```

After successful deployment, Firebase will provide you with a hosting URL where your application is live.

## Additional Configuration

### Environment Variables

If you need to use environment variables:

1. Create a `.env` file in your project root
2. Add your environment variables
3. Use them in your `environment.ts` and `environment.prod.ts` files

### Custom Domain

To set up a custom domain:

1. Go to Firebase Console
2. Navigate to Hosting
3. Click "Add custom domain"
4. Follow the verification steps

## Troubleshooting

Common issues and solutions:

1. **404 Errors**: Make sure your `firebase.json` has the correct rewrite rules
2. **Build Errors**: Check your Angular build configuration
3. **Deployment Errors**: Ensure you're logged in to Firebase and have the correct permissions

## Continuous Deployment (Optional)

To set up continuous deployment with GitHub:

1. Connect your GitHub repository in Firebase Console
2. Configure build settings
3. Enable automatic deployments

## Security Rules

For additional security, you can add security rules in your `firebase.json`:

```json
{
  "hosting": {
    "public": "dist/[your-app-name]",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(eot|otf|ttf|ttc|woff|font.css)",
        "headers": [
          {
            "key": "Access-Control-Allow-Origin",
            "value": "*"
          }
        ]
      }
    ]
  }
}
```

## Additional Resources

- [Firebase Hosting Documentation](https://firebase.google.com/docs/hosting)
- [Angular Deployment Guide](https://angular.io/guide/deployment)
- [Firebase CLI Reference](https://firebase.google.com/docs/cli)
