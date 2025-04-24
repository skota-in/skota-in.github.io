# AWS ACM Certificate Creation and Validation

## Creating ACM Certificate with Terraform

Here's how to create and validate an ACM certificate using Terraform:

```hcl
resource "aws_acm_certificate" "example" {
  domain_name       = "example.com"
  validation_method = "DNS"

  subject_alternative_names = [
    "*.example.com",
    "api.example.com"
  ]

  tags = {
    Environment = "production"
    Name        = "example-certificate"
  }
}

resource "aws_route53_record" "example_validation" {
  for_each = {
    for dvo in aws_acm_certificate.example.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = aws_route53_zone.example.zone_id
}

resource "aws_acm_certificate_validation" "example" {
  certificate_arn         = aws_acm_certificate.example.arn
  validation_record_fqdns = [for record in aws_route53_record.example_validation : record.fqdn]
}

output "certificate_arn" {
  value = aws_acm_certificate.example.arn
}
```

### Explanation of Terraform Resources

1. **aws_acm_certificate**: 
   - Creates the ACM certificate
   - Supports multiple domains through `subject_alternative_names`
   - Uses DNS validation method
   - Optional tags for better resource management

2. **aws_route53_record**:
   - Creates the DNS validation records
   - Uses `for_each` to handle multiple domain validations
   - Automatically creates the correct record type (CNAME or TXT)
   - Allows overwriting existing records if needed

3. **aws_acm_certificate_validation**:
   - Waits for the certificate to be validated
   - Uses the created DNS records to verify domain ownership
   - Ensures the certificate is fully validated before proceeding

### Using the Certificate

Once the certificate is validated, you can use it with other AWS services:

```hcl
resource "aws_lb_listener" "example" {
  load_balancer_arn = aws_lb.example.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-2016-08"
  certificate_arn   = aws_acm_certificate.example.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.example.arn
  }
}
```

### Important Terraform Notes

- The certificate validation process can take several minutes
- Make sure your Route 53 zone is properly configured
- The `aws_acm_certificate_validation` resource will wait until validation is complete
- You can use the certificate ARN in other resources once validation is complete

### Troubleshooting Terraform

If you encounter issues:
1. Check the AWS Console for certificate status
2. Verify DNS records are created correctly
3. Ensure Route 53 zone ID is correct
4. Check IAM permissions for ACM and Route 53

## AWS ACM Certificate Creation and Validation from Console

This guide explains how to create and validate an AWS ACM (Amazon Certificate Manager) certificate, including DNS validation steps.

## Creating an ACM Certificate

1. **Request a Certificate**:
   - Go to AWS Certificate Manager in the AWS Console
   - Click "Request a certificate"
   - Choose "Request a public certificate"
   - Enter your domain name (e.g., example.com)
   - For wildcard certificates, use *.example.com
   - Click "Next"

2. **Select Validation Method**:
   - Choose "DNS validation" (recommended)
   - Click "Request"

## DNS Validation

After requesting the certificate, AWS will provide CNAME records that need to be added to your DNS configuration. There are two types of records you might need to add:

### CNAME Record
This is the most common validation method. AWS will provide a CNAME record that looks like:
```
Name: _acm-validation.example.com
Value: _acm-validation.xxxxx.acm-validations.aws
```

### TXT Record
In some cases, you might need to add a TXT record instead:
```
Name: _acm-validation.example.com
Value: "xxxxx-xxxxx-xxxxx-xxxxx"
```

## Adding DNS Records

1. **For Route 53**:
   - Go to Route 53 in AWS Console
   - Select your hosted zone
   - Click "Create record"
   - Choose "CNAME" or "TXT" based on the validation method
   - Enter the name and value provided by ACM
   - Click "Create records"

2. **For Other DNS Providers**:
   - Log in to your DNS provider's console
   - Navigate to DNS management
   - Add a new CNAME or TXT record
   - Enter the name and value provided by ACM
   - Save the changes

## Validation Process

1. After adding the DNS records:
   - Wait for DNS propagation (can take up to 72 hours, but usually much faster)
   - AWS will automatically check the DNS records
   - The certificate status will change to "Issued" when validated

2. **Check Validation Status**:
   - Go back to AWS Certificate Manager
   - Look for your certificate
   - Status should show "Issued" when validation is complete

## Important Notes

- DNS propagation can take time (usually 5-30 minutes)
- Keep your DNS records until the certificate is issued
- You can use the same certificate for multiple AWS services
- Certificates are valid for 13 months
- AWS will automatically attempt to renew certificates before expiration

## Troubleshooting

If validation is taking too long:
1. Verify the DNS records are correctly entered
2. Check for typos in the record values
3. Use DNS lookup tools to verify the records are propagated
4. Ensure there are no conflicting records