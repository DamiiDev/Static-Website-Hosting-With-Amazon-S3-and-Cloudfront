# RegPro — Static Website Hosting with Amazon S3 & CloudFront

**A hands-on AWS Cloud Engineering Project by Damilare Festus**

## Overview

I built this project to strengthen my practical cloud engineering skills by deploying a static website for **RegPro**, a fictional company registration service, using Amazon Web Services (AWS).

The project demonstrates how to store website files in Amazon S3, distribute content globally through Amazon CloudFront, and secure a private S3 bucket using Origin Access Control (OAC).

My objective was to gain hands-on experience with cloud storage, content delivery, access control, HTTPS, and deployment automation using the AWS CLI and Bash scripting.

## Architecture

```text
                    Website Visitor
                           |
                           | HTTPS
                           v
                   Amazon CloudFront
                    (Global CDN)
                           |
                           | OAC-signed requests
                           v
                     Private S3 Bucket
                           |
               +-----------+-----------+
               |           |           |
           index.html  contact.html  css/ and js/
```

**Security approach:** The S3 bucket blocks public access. CloudFront uses Origin Access Control to retrieve website objects through an authorized bucket policy.

## AWS Services and Technologies

- **Amazon S3:** Stores the website's HTML, CSS, and JavaScript files.
- **Amazon CloudFront:** Distributes website content globally and supports HTTPS.
- **CloudFront Origin Access Control (OAC):** Secures access between CloudFront and the private S3 bucket.
- **AWS IAM:** Manages permissions for AWS resource creation and deployment.
- **AWS CLI:** Automates resource provisioning, file uploads, and configuration.
- **Bash scripting:** Automates deployment tasks and reduces repetitive manual work.
- **Git and GitHub:** Support version control and project documentation.

## Project Structure

```text
RegPro/
├── index.html
├── contact.html
├── css/
│   └── styles.css
├── js/
│   └── main.js
├── deploy-website.sh
├── cleanup-website.sh
├── deployment-info.txt
└── README.md
```

- `index.html` — Homepage.
- `contact.html` — Contact page.
- `css/styles.css` — Website styling and responsive layout.
- `js/main.js` — Client-side JavaScript functionality.
- `deploy-website.sh` — Automated deployment script.
- `cleanup-website.sh` — Resource cleanup script.
- `deployment-info.txt` — Generated deployment information.

## Implementation Steps

### 1. Create a Private S3 Bucket

I used the AWS CLI to create a uniquely named S3 bucket in the `us-east-1` region.

The deployment script enables all four S3 Block Public Access settings to prevent direct public access to bucket objects.

### 2. Upload Website Files

The script uses `aws s3 sync` to upload website files while excluding deployment scripts, Markdown files, Git metadata, and the generated deployment information file.

### 3. Configure CloudFront Origin Access Control

I configured OAC using AWS Signature Version 4 signing. This enables CloudFront to make authenticated requests to the private S3 origin.

### 4. Create a CloudFront Distribution

The deployment script creates a CloudFront distribution configured with:

- `index.html` as the default root object.
- HTTP-to-HTTPS redirection.
- Compression enabled.
- GET and HEAD as the allowed HTTP methods.
- Caching for static website content.
- A custom error response for HTTP 404 errors.

### 5. Apply the S3 Bucket Policy

The bucket policy grants the CloudFront service principal permission to retrieve S3 objects using `s3:GetObject`. Access is restricted to the specific CloudFront distribution ARN.

This approach avoids granting public read access to the bucket.

### 6. Access the Website Over HTTPS

CloudFront provides a default domain in this format:

`https://dxxxxxxxxxxxxx.cloudfront.net`

This allows the website to be accessed over HTTPS without purchasing a custom domain or requesting a separate ACM certificate.

A custom domain and AWS Certificate Manager (ACM) certificate can be added later if required.

## Automated Deployment

### Prerequisites

- An AWS account.
- AWS CLI v2 installed.
- AWS credentials configured with the required permissions.
- Bash terminal, such as Ubuntu on WSL.
- Website files available in the project root directory.

### Verify AWS CLI Installation

```bash
aws --version
```

### Verify AWS Authentication

```bash
aws sts get-caller-identity
```

### Deploy the Website

From the project directory, run:

```bash
bash deploy-website.sh
```

The script creates the S3 bucket, uploads website files, creates the OAC and CloudFront distribution, and applies the required bucket policy.

Deployment details are saved to `deployment-info.txt`.

**Note:** CloudFront distribution creation is asynchronous. Wait until the distribution status is `Deployed` before expecting the website to be available.

## Updating Website Content

After making changes to the local HTML, CSS, or JavaScript files, synchronize the updated content with S3:

```bash
aws s3 sync . s3://YOUR_BUCKET_NAME \
  --exclude "*.sh" \
  --exclude "*.md" \
  --exclude ".git/*" \
  --exclude "deployment-info.txt"
```

Replace `YOUR_BUCKET_NAME` with your actual bucket name.

### Invalidate the CloudFront Cache

If the updated content is not immediately visible, create a cache invalidation:

```bash
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/index.html" "/contact.html" "/css/*" "/js/*"
```

Replace `YOUR_DISTRIBUTION_ID` with your CloudFront distribution ID.

## Testing and Validation

| Test | Expected Result |
|---|---|
| CloudFront HTTPS URL | Website loads over HTTPS |
| Homepage | `index.html` displays correctly |
| Contact page | `contact.html` opens correctly |
| CSS and JavaScript | Styling and functionality load correctly |
| Direct S3 object access | Public access is blocked |
| CloudFront access to S3 | Website objects are served successfully |
| Cache invalidation | Updated content becomes available after cache refresh |

## Security Considerations

- Keep S3 Block Public Access enabled.
- Use OAC and a distribution-specific bucket policy.
- Apply least-privilege IAM permissions.
- Never commit AWS access keys or secret keys to GitHub.
- Monitor AWS billing and remove unused resources.

## Cleanup

To remove the resources created for this project, review and run the cleanup script:

```bash
bash cleanup-website.sh
```

Before executing the script, verify that it correctly handles the CloudFront distribution, S3 objects, bucket, and associated resources.

CloudFront distributions generally must be disabled and fully deployed in that state before deletion. Ensure that the bucket is emptied before attempting to delete it.

## Learning Outcomes

Through this project, I gained practical experience with:

- Static website hosting using Amazon S3.
- Private S3 buckets and bucket policies.
- Global content delivery using Amazon CloudFront.
- Origin Access Control for securing an S3 origin.
- HTTPS delivery through CloudFront.
- AWS resource provisioning using Bash and the AWS CLI.
- File synchronization and CloudFront cache invalidation.
- Cloud deployment security and cost management.

## Future Improvements

- Configure a custom domain using Amazon Route 53.
- Request an ACM public certificate for the custom domain in `us-east-1`.
- Implement automated deployment using GitHub Actions.
- Improve deployment validation and error handling.
- Add monitoring and logging for troubleshooting.

## Author

**Damilare Festus**

Aspiring Cloud Engineer | IT Support | AWS & Cloud Infrastructure

- **GitHub:** https://github.com/DamiiDev
- **LinkedIn:** https://linkedin.com/in/damilare-festus-915386202/

---

*This project is part of my hands-on cloud engineering learning journey, focused on applying AWS concepts through practical implementation.*
