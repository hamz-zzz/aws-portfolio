# 4. AWS Content Delivery and DNS

![Content delivery and DNS architecture](architecture/04-cloudfront-and-route-53.png)

## Overview

A static website is hosted in a private S3 bucket and delivered through CloudFront. Route 53 directs `systemsbyhamza.com` to the CloudFront distribution.

The S3 bucket blocks public access. CloudFront uses Origin Access Control (OAC) to sign requests to the S3 origin, and the bucket policy allows requests from the distribution only. An ACM certificate provides HTTPS, and CloudFront caching is enabled.

Validation covered DNS resolution over IPv4 and IPv6, direct access to the S3 bucket, CloudFront cache hits and misses, and cache invalidation.

## Architecture

1. Route 53 directs requests for the custom domain to CloudFront.
2. CloudFront serves cached objects or requests content from S3 through OAC.
3. S3 stores the website files. The bucket policy restricts access to the CloudFront distribution.

CloudFront uses the S3 bucket's regional REST endpoint rather than the S3 static website endpoint. The distribution uses `index.html` as its default root object.

## Route 53 and custom domain

`systemsbyhamza.com` was registered through Route 53, and a public hosted zone was created for the domain.

| Record | Name | Target |
|---|---|---|
| A (Alias) | `systemsbyhamza.com` | CloudFront distribution |
| AAAA (Alias) | `systemsbyhamza.com` | CloudFront distribution |
| CNAME | `www.systemsbyhamza.com` | `systemsbyhamza.com` |

The root domain has A and AAAA alias records for IPv4 and IPv6. The `www` CNAME points to `systemsbyhamza.com`.

**Domain registration**

![Route 53 domain registration](screenshots/1-1-domain-registered.png)

**Public hosted zone**

![Route 53 hosted zone overview](screenshots/1-2-hosted-zone-overview.png)

**Root A alias record**

![Route 53 root A alias record](screenshots/1-3-root-alias-a-record.png)

**Root AAAA alias record**

![Route 53 root AAAA alias record](screenshots/1-4-root-alias-aaaa-record.png)

**WWW CNAME record**

![Route 53 WWW CNAME record](screenshots/1-5-www-cname-record.png)

## Private S3 origin

The S3 bucket contains `index.html` and the page's image assets. Block Public Access and default encryption are enabled.

**S3 bucket**

![S3 bucket overview](screenshots/2-1-s3-bucket-overview.png)

**Block Public Access**

![S3 Block Public Access settings](screenshots/2-2-s3-block-public-access.png)

**Website files**

![S3 website object structure](screenshots/2-3-s3-objects-structure.png)

**Image assets**

![S3 image assets](screenshots/2-4-s3-images-folder.png)

### Origin Access Control

CloudFront uses the S3 REST endpoint as its origin instead of the S3 website endpoint. OAC signs requests from CloudFront, and the bucket policy allows access from the distribution.

![S3 bucket policy for CloudFront OAC](screenshots/2-5-s3-bucket-policy.png)

A direct request to the S3 object URL returned `AccessDenied`, as expected.

![Direct S3 object access denied](screenshots/2-6-s3-access-denied-test.png)

## CloudFront distribution

The distribution uses `systemsbyhamza.com` and `www.systemsbyhamza.com` as alternate domain names.

| Setting | Configuration |
|---|---|
| Origin | Private S3 bucket REST endpoint |
| Origin access | Origin Access Control (OAC) |
| Default root object | `index.html` |
| Viewer protocol policy | Redirect HTTP to HTTPS |
| Allowed methods | GET and HEAD |
| Custom domains | `systemsbyhamza.com`, `www.systemsbyhamza.com` |

**Distribution overview**

![CloudFront distribution overview](screenshots/3-1-cloudfront-distribution-overview.png)

**Origin configuration**

![CloudFront origin configuration](screenshots/3-2-cloudfront-origin-config.png)

**Cache behavior settings**

![CloudFront behavior settings](screenshots/3-3-cloudfront-behavior-settings.png)

The first request fetched the object from S3. A repeated request returned a cache hit. An invalidation was also tested to remove the cached object.

## HTTPS with ACM

An ACM certificate was requested in `us-east-1` for `systemsbyhamza.com` and `www.systemsbyhamza.com`.

The certificate was validated through Route 53 DNS records and attached to the CloudFront distribution. Both custom domains use HTTPS.

![Issued ACM certificate](screenshots/3-4-acm-certificate-issued.png)

## Validation

The website was tested through the custom domain, the CloudFront distribution domain, and the S3 object URL.

### Website access

The homepage loaded through `systemsbyhamza.com`, and the browser showed a valid HTTPS certificate.

**Website homepage**

![Website homepage through the custom domain](screenshots/4-1-site-homepage.png)

**HTTPS certificate**

![HTTPS certificate in the browser](screenshots/4-2-site-https-cert.png)

The page's image also loads through CloudFront.

![Static image loaded through CloudFront](screenshots/4-3-image-direct-load.png)

The website also loaded through the CloudFront distribution domain and `www.systemsbyhamza.com`. The distribution domain worked independently of the custom Route 53 records.

| CloudFront distribution domain | `www` custom domain |
| --- | --- |
| ![Website access through the CloudFront domain](screenshots/7-1-cloudfront-domain-access.png) | ![Website access through the WWW subdomain](screenshots/7-2-www-domain-access.png) |

### Cache behavior

Testing covered a cache miss, a cache hit, and a request after invalidation.

| Test | Expected behavior |
| --- | --- |
| Cache miss | CloudFront fetches the object from S3. |
| Cache hit | CloudFront serves the cached object. |
| Invalidation | An invalidation for `/*` removes the cached objects. |
| Request after invalidation | The next request returns a cache miss. |

**Cache miss**

![CloudFront cache miss](screenshots/5-1-cache-miss.png)

**Cache hit**

![CloudFront cache hit](screenshots/5-2-cache-hit.png)

**Invalidation**

![CloudFront cache invalidation](screenshots/5-3-cache-invalidation.png)

**Request after invalidation**

![CloudFront cache status after invalidation](screenshots/5-4-cache-after-invalidation.png)

### DNS resolution

`nslookup` and `dig` were used to check DNS resolution. Both commands returned CloudFront IPv4 and IPv6 addresses for the root domain.

| `nslookup` | `dig` |
| --- | --- |
| ![nslookup terminal output](screenshots/6-1-nslookup-output.png) | ![dig terminal output](screenshots/6-2-dig-output.png) |

The results matched the A and AAAA alias records in Route 53.

## Project files

- **[architecture/](./architecture/)** — PNG architecture diagram and editable draw.io source.
- **[configs/](./configs/)** — `index.html` and the static image assets.
- **[screenshots/](./screenshots/)** — Route 53, S3, CloudFront, ACM, DNS resolution, and validation evidence.