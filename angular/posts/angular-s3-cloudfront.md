# Deploying Angular Application to S3 and CloudFront

This guide will walk you through the process of deploying an Angular application to Amazon S3 and configuring CloudFront for content delivery.

## Prerequisites

- AWS Account with appropriate permissions
- AWS CLI installed and configured
- Angular application ready for production
- Node.js and npm installed

## Step 1: Build Your Angular Application

First, build your Angular application for production:

```bash
ng build --configuration production
```

This will create a `dist` folder with your production-ready files.

## Step 2: Create an S3 Bucket

1. Log in to the AWS Management Console
2. Navigate to S3 service
3. Click "Create bucket"
4. Configure the bucket:
   - Choose a unique bucket name
   - Select your preferred region
   - Uncheck "Block all public access" (we'll need public access for the website)
   - Acknowledge the warning about public access
   - Click "Create bucket"

## Step 3: Configure S3 Bucket for Static Website Hosting

1. Select your bucket in the S3 console
2. Go to the "Properties" tab
3. Scroll down to "Static website hosting"
4. Click "Edit"
5. Enable static website hosting
6. Set:
   - Index document: `index.html`
   - Error document: `index.html` (for SPA routing)
7. Save changes

## Step 4: Configure Bucket Policy

1. Go to the "Permissions" tab
2. Click "Edit" under "Bucket policy"
3. Add the following policy (replace `your-bucket-name` with your actual bucket name):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
    ]
}
```

## Step 5: Upload Your Application

You can upload your files using the AWS CLI:

```bash
aws s3 sync dist/your-app-name s3://your-bucket-name --delete
```

Or manually through the S3 console:
1. Navigate to your bucket
2. Click "Upload"
3. Select all files from your `dist` folder
4. Click "Upload"

## Step 6: Create CloudFront Distribution

1. Navigate to CloudFront in AWS Console
2. Click "Create distribution"
3. Configure the distribution:
   - Origin domain: Select your S3 bucket
   - Origin path: Leave empty
   - Viewer protocol policy: Redirect HTTP to HTTPS
   - Cache policy: CachingOptimized
   - Default root object: index.html
   - Price class: Select based on your needs
4. Click "Create distribution"

## Step 7: Configure CloudFront for SPA Routing

1. Go to your CloudFront distribution
2. Click "Error pages" tab
3. Click "Create custom error response"
4. Configure:
   - HTTP error code: 403
   - Custom error response: Yes
   - Response page path: /index.html
   - HTTP status code: 200
5. Repeat for error code 404

## Step 8: Update DNS (Optional)

If you want to use a custom domain:
1. Go to your domain registrar
2. Create a CNAME record pointing to your CloudFront distribution domain
3. In CloudFront, add your custom domain to the "Alternate domain names" list
4. Request or upload an SSL certificate for your domain

## Step 9: Test Your Deployment

1. Wait for CloudFront distribution to deploy (can take 5-15 minutes)
2. Access your application using the CloudFront domain name
3. Test all routes and functionality

## Additional Tips

- Enable compression in CloudFront for better performance
- Set up proper cache headers for static assets
- Consider implementing CI/CD pipeline for automated deployments
- Monitor your CloudFront distribution metrics
- Set up proper logging for debugging

## Troubleshooting

- If you see 403 errors, check your S3 bucket policy
- If routing doesn't work, verify your CloudFront error page configuration
- If assets aren't loading, check the paths in your Angular application
- If you see CORS issues, configure CORS on your S3 bucket

## Security Considerations

- Regularly update your Angular application
- Implement proper security headers
- Consider using AWS WAF for additional protection
- Monitor your CloudFront logs for suspicious activity
- Implement proper access controls for your S3 bucket
