# Using Multiple index.html Files in Angular for Different Environments

*Published on February 12, 2024*

When working with Angular applications, you might need different `index.html` files for development, testing, and production environments. This guide will show you how to set this up effectively.

## Why Use Multiple index.html Files?

Different environments often require different configurations:
- Development: Debugging tools, mock services
- Testing: Test-specific configurations, analytics
- Production: Optimized builds, production analytics

## Setup Instructions

### 1. Create Environment-Specific index.html Files

Create separate `index.html` files in your `src` directory:

```
src/
  ├── index.html           # Default/development
  ├── index.test.html     # Testing environment
  └── index.prod.html     # Production environment
```

### 2. Configure angular.json

Update your `angular.json` file to use different index files based on the build configuration:

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "build": {
          "configurations": {
            "development": {
              "index": "src/index.html"
            },
            "test": {
              "index": "src/index.test.html"
            },
            "production": {
              "index": "src/index.prod.html"
            }
          }
        }
      }
    }
  }
}
```

### 3. Create Environment-Specific Content

Each `index.html` file can have environment-specific content:

#### Development (index.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>My App - Development</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
  <!-- Development specific scripts -->
  <script src="assets/dev-tools.js"></script>
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

#### Testing (index.test.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>My App - Testing</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
  <!-- Testing specific scripts -->
  <script src="assets/test-analytics.js"></script>
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

#### Production (index.prod.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>My App</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
  <!-- Production specific scripts -->
  <script src="assets/prod-analytics.js"></script>
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

### 4. Build Commands

Use the appropriate configuration when building:

```bash
# Development build (default)
ng build

# Testing build
ng build --configuration=test

# Production build
ng build --configuration=production
```

## Best Practices

1. **Keep Common Elements**: Maintain common elements across all index files to ensure consistency
2. **Environment Variables**: Use environment variables for dynamic content
3. **Version Control**: Keep all index files in version control
4. **Documentation**: Document the differences between environments
5. **Testing**: Test each environment's build process

## Common Use Cases

- Different analytics tracking codes
- Environment-specific meta tags
- Different CDN configurations
- Debugging tools in development
- Performance monitoring in production

## Troubleshooting

If you encounter issues:
1. Verify the file paths in `angular.json`
2. Check for syntax errors in HTML files
3. Ensure the build configuration names match
4. Clear the build cache if needed: `ng build --configuration=production --delete-output-path`

## Conclusion

Using multiple `index.html` files allows for better environment-specific customization while maintaining a clean and organized project structure. This approach is particularly useful for large applications with different deployment requirements across environments.
