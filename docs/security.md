# Security

## Private S3 origin

Current design:

- S3 Block Public Access: **enabled**
- S3 static website hosting: **disabled**
- Public bucket access: **disabled**
- CloudFront Origin Access Control: **enabled**
- Public content delivery: **through CloudFront**

CloudFront is the public layer. S3 remains private.

## Origin Access Control

The bucket policy follows this model:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "cloudfront.amazonaws.com"
  },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::BUCKET_NAME/*",
  "Condition": {
    "StringEquals": {
      "AWS:SourceArn": "CLOUDFRONT_DISTRIBUTION_ARN"
    }
  }
}
```

The important parts are:

1. CloudFront is the principal.
2. The site only requires object reads.
3. The source ARN condition restricts access to the intended distribution.

## TLS

The CloudFront distribution uses an ACM certificate for:

```text
dague.vip
*.dague.vip
```

The certificate is in `us-east-1`.

## Cloudflare

Cloudflare is authoritative for DNS, but the website records are not proxied. This keeps the delivery path:

```text
Viewer -> CloudFront -> private S3
```
