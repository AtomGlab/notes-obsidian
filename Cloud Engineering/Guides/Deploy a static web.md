

# Deploying a Static Portfolio on AWS

## Overview

This document describes the complete process used to deploy the `atomlab.cloud` portfolio using AWS services.

The final architecture uses:

- **Amazon S3** — stores and serves the static website files.
    
- **Amazon CloudFront** — CDN and HTTPS delivery.
    
- **AWS Certificate Manager (ACM)** — TLS/SSL certificate.
    
- **Amazon Route 53** — DNS management.
    
- **Namecheap** — domain registrar.
    

The domain remains registered with Namecheap, while DNS management is delegated to Route 53.

---

## Final Architecture

```text
                         Internet
                            │
                            │
                     https://atomlab.cloud
                            │
                            ▼
                     ┌──────────────┐
                     │  Namecheap   │
                     │  Registrar   │
                     └──────┬───────┘
                            │
                     Nameserver delegation
                            │
                            ▼
                     ┌──────────────┐
                     │  Route 53    │
                     │     DNS      │
                     └──────┬───────┘
                            │
                     Alias A Record
                            │
                            ▼
                     ┌──────────────┐
                     │  CloudFront  │
                     │     CDN      │
                     └──────┬───────┘
                            │
                     S3 Website Endpoint
                            │
                            ▼
                  ┌─────────────────────┐
                  │  S3 Bucket          │
                  │  atomlab-portfolio  │
                  └──────────┬──────────┘
                             │
                             ▼
                         index.html
```

---

# 1. Domain Registration

The domain used for the portfolio is:

```text
atomlab.cloud
```

The domain is registered with **Namecheap**.

The domain was **not transferred to AWS**. Instead, the domain remains registered with Namecheap while its DNS is managed by Route 53.

This separation is useful because:

- Namecheap remains the registrar.
    
- AWS Route 53 manages DNS records.
    
- AWS services can be connected directly through Route 53.
    
- The domain can remain registered with the existing provider.
    

---

# 2. Create a Route 53 Hosted Zone

A public hosted zone was created in Route 53:

```text
Domain:
atomlab.cloud

Type:
Public hosted zone
```

Route 53 automatically created the following records:

```text
NS
SOA
```

The Route 53 nameservers were:

```text
ns-856.awsdns-43.net
ns-1455.awsdns-53.org
ns-203.awsdns-25.com
ns-1961.awsdns-53.co.uk
```

These nameservers identify Route 53 as the authoritative DNS provider for the domain.

---

# 3. Delegate DNS from Namecheap to Route 53

The domain was still registered at Namecheap.

Inside:

```text
Namecheap
→ Domain List
→ atomlab.cloud
→ Manage
→ Nameservers
```

the nameserver configuration was changed from:

```text
Namecheap BasicDNS
```

to:

```text
Custom DNS
```

The four Route 53 nameservers were entered:

```text
ns-856.awsdns-43.net
ns-1455.awsdns-53.org
ns-203.awsdns-25.com
ns-1961.awsdns-53.co.uk
```

After this change, Namecheap continued to own the domain registration, but Route 53 became responsible for DNS resolution.

---

# 4. Create the S3 Website

The website was stored in an S3 bucket:

```text
atomlab-portfolio
```

AWS Region:

```text
Europe (Ireland)
eu-west-1
```

The main website file was:

```text
index.html
```

Its S3 URI is:

```text
s3://atomlab-portfolio/index.html
```

The object was configured with:

```text
Content-Type: text/html
```

---

# 5. Enable S3 Static Website Hosting

Inside:

```text
S3
→ atomlab-portfolio
→ Properties
→ Static website hosting
```

static website hosting was enabled.

Configuration:

```text
Static website hosting: Enabled
Hosting type: Bucket hosting
Index document: index.html
```

AWS provided the website endpoint:

```text
http://atomlab-portfolio.s3-website-eu-west-1.amazonaws.com
```

This endpoint became the origin used by CloudFront.

---

# 6. Create the CloudFront Distribution

A CloudFront distribution was created for the website.

Distribution name:

```text
atomlab-portfolio-cloudfront
```

Distribution type:

```text
Single website configuration
```

The S3 website endpoint was selected as the origin.

The origin was configured to use the S3 website endpoint because the bucket had static website hosting enabled.

CloudFront distribution domain:

```text
d1esojqeivbbto.cloudfront.net
```

---

# 7. Configure the CloudFront Origin

The origin was configured as:

```text
Amazon S3
```

The S3 bucket:

```text
atomlab-portfolio
```

was selected.

Because S3 static website hosting was enabled, CloudFront was configured to use the **website endpoint** rather than the standard S3 bucket endpoint.

The origin path was left empty.

Recommended CloudFront origin settings were used.

Recommended cache settings for S3 content were also used.

---

# 8. Configure HTTPS with AWS Certificate Manager

CloudFront requires a TLS certificate to serve the website over HTTPS.

A certificate was created through **AWS Certificate Manager (ACM)**.

Important:

> CloudFront requires ACM certificates to be created in the `us-east-1` region.

The certificate was therefore created in:

```text
US East (N. Virginia)
us-east-1
```

The certificate covers:

```text
atomlab.cloud
www.atomlab.cloud
www.portfolio.atomlab.cloud
```

The primary domains required for the website are:

```text
atomlab.cloud
www.atomlab.cloud
```

The certificate was then associated with the CloudFront distribution.

TLS security policy:

```text
TLSv1.2_2021
```

---

# 9. Configure the CloudFront Default Root Object

CloudFront needs to know which file should be returned when a user visits the root of the domain.

The default root object was configured as:

```text
index.html
```

Therefore:

```text
https://atomlab.cloud/
```

is resolved to:

```text
index.html
```

through CloudFront.

---

# 10. Configure Route 53

After CloudFront was created, Route 53 was configured to route the domain to the CloudFront distribution.

The first record was created for the root domain.

Configuration:

```text
Record name:
(empty)

Record type:
A

Alias:
Enabled

Routing policy:
Simple

Target:
CloudFront distribution
d1esojqeivbbto.cloudfront.net
```

The record uses an **Alias** instead of manually entering an IP address.

This is important because CloudFront does not provide a fixed IP address that should be manually configured in DNS.

---

# 11. Configure the WWW Subdomain

A second DNS record was configured for:

```text
www.atomlab.cloud
```

Configuration:

```text
Record name:
www

Record type:
A

Alias:
Enabled

Routing policy:
Simple

Target:
d1esojqeivbbto.cloudfront.net
```

This allows both:

```text
https://atomlab.cloud
```

and:

```text
https://www.atomlab.cloud
```

to use the same CloudFront distribution.

---

# 12. Final Request Flow

When a user enters:

```text
https://atomlab.cloud
```

the request follows this path:

```text
Browser
   │
   ▼
DNS resolution
   │
   ▼
Route 53
   │
   ▼
CloudFront
   │
   ▼
S3 Website Endpoint
   │
   ▼
atomlab-portfolio
   │
   ▼
index.html
```

CloudFront provides:

- HTTPS
    
- Global edge caching
    
- CDN delivery
    
- TLS termination
    
- Integration with AWS Certificate Manager
    

S3 provides:

- Static file storage
    
- Website content
    
- High durability
    
- Serverless hosting for the static assets
    

Route 53 provides:

- DNS resolution
    
- Alias records
    
- Integration with CloudFront
    

---

# 13. Final AWS Resources

The resulting infrastructure consists of:

## Amazon S3

```text
Bucket:
atomlab-portfolio

Region:
eu-west-1

Website:
index.html
```

## CloudFront

```text
Distribution:
atomlab-portfolio-cloudfront

Distribution domain:
d1esojqeivbbto.cloudfront.net
```

## Route 53

```text
Hosted zone:
atomlab.cloud

Type:
Public hosted zone
```

## ACM

```text
Region:
us-east-1

Certificate:
atomlab.cloud
```

## Domain Registrar

```text
Namecheap

Domain:
atomlab.cloud
```

---

# 14. Why This Architecture?

This architecture separates the responsibilities of each AWS service.

### S3

Responsible for storing the static website.

```text
HTML
CSS
JavaScript
Images
Assets
```

### CloudFront

Responsible for delivering the website globally.

It provides:

- CDN caching
    
- HTTPS
    
- TLS termination
    
- Edge locations
    
- Reduced latency
    

### Route 53

Responsible for DNS.

It maps:

```text
atomlab.cloud
```

to:

```text
CloudFront
```

### ACM

Responsible for the TLS certificate.

It allows the website to use:

```text
https://
```

instead of:

```text
http://
```

### Namecheap

Remains responsible for domain registration.

---

# 15. Result

The final portfolio is accessible through:

```text
https://atomlab.cloud
```

and:

```text
https://www.atomlab.cloud
```

The domain remains registered with Namecheap, while AWS manages the infrastructure and DNS required to deliver the website.

---

# 16. Architecture Summary

```text
┌──────────────────────┐
│      Namecheap       │
│   Domain Registrar   │
└──────────┬───────────┘
           │
           │ Nameserver delegation
           ▼
┌──────────────────────┐
│      Route 53        │
│        DNS           │
└──────────┬───────────┘
           │
           │ Alias A
           ▼
┌──────────────────────┐
│     CloudFront       │
│        CDN           │
│       HTTPS          │
└──────────┬───────────┘
           │
           │ Origin
           ▼
┌──────────────────────┐
│         S3           │
│  atomlab-portfolio   │
│                      │
│     index.html       │
└──────────────────────┘

          ▲
          │
     ACM Certificate
       (us-east-1)
```

---

## Key Concepts Learned

This deployment demonstrates several fundamental AWS concepts relevant to the **AWS Solutions Architect – Associate (SAA)** certification:

- DNS and domain delegation
    
- Route 53 hosted zones
    
- DNS records and Alias records
    
- Amazon S3 static website hosting
    
- CloudFront distributions
    
- CDN architecture
    
- Origin configuration
    
- HTTPS and TLS
    
- AWS Certificate Manager
    
- AWS regions
    
- Separation between domain registration and DNS hosting
    
- Serverless static website architecture
    
- Content caching and edge delivery
    

The resulting architecture is a simple example of a highly available, serverless static website delivered through AWS infrastructure.