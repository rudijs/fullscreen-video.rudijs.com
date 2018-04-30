## AWS resources:
1. Route53
2. Certificate Manager
3. S3 static website
4. Cloudfront


## Register a Domain name with Route53
Register the domain name.

Create a *. wildcard and fqdn (ex: rudijs.com & *.rudijs.com) SSL cert at AWS Certficate Manager

https://medium.com/@victor.leong.17/cheap-wildcard-ssl-certificate-with-aws-route-53-and-certificate-manager-ac922a4af5af

## S3 Static Web Site (non https)
- `aws --profile default --region ap-southeast-1 s3 mb s3://fullscreen-video.rudijs.com`
- `aws --profile default --region ap-southeast-1 s3 website s3://fullscreen-video.rudijs.com --index-document index.html --error-document index.html`
- `aws --profile default --region ap-southeast-1 s3api put-bucket-policy --bucket fullscreen-video.rudijs.com --policy file://s3-fullscreen-video.rudijs.com-policy.json`

## Deploy application
- `aws --profile default --region ap-southeast-1 s3 sync src/ s3://fullscreen-video.rudijs.com`
- `google-chrome fullscreen-video.rudijs.com.s3-website-ap-southeast-1.amazonaws.com`

## Cloudfront with HTTPS
- `aws --profile default cloudfront create-distribution --origin-domain-name fullscreen-video.rudijs.com.s3.amazonaws.com --default-root-object index.html`

Now open up the web UI and make manual edits.

Manual is quicker and easier for now, rather than creating and editing Cloudformation json.

General:
- Alternate Domain Names (CNAMEs): fullscreen-video.rudijs.com
- Custom SSL Certificate (example.com): rudijs.com

Behaviours:
- Viewer Protocol Policy: Redirect HTTP to HTTPS
- Allowed HTTP Methods: (Leave for now, may need to review later)

## Route53 DNS
Get the URL to the cloudfront distribution just created:

- `aws cloudfront list-distributions | jq '.DistributionList.Items[] | .Id + " " + .DomainName + " " + .DefaultCacheBehavior.TargetOriginId' | grep fullscreen-video`

Update the *Comment*, *Name* and *DNSName* in the file `dns-fullscreen-video.rudijs.com.json`

Get the hosted-zone-id for the *rudijs.com* domain, use any of these commands (the last one is the most direct):

- `aws route53 list-hosted-zones-by-name`
- `aws route53 list-hosted-zones`
- `aws route53 list-hosted-zones | jq '.HostedZones[].Id'`
- `aws route53 list-hosted-zones | jq '.HostedZones[].Name + " " + .HostedZones[].Id'`

Create a new DNS A record that is an aliaes pointing to the cloudfront distribution URL

- `aws route53 change-resource-record-sets --hosted-zone-id Z1RD0SSFONJ4KR --change-batch file://dns-fullscreen-video.rudijs.com.json`
- `google-chrome fullscreen-video.rudijs.com`

## Deploy
- `aws --profile default --region ap-southeast-1 s3 sync src/ s3://fullscreen-video.rudijs.com`
- `aws cloudfront list-distributions`
- `aws cloudfront list-distributions | jq '.DistributionList.Items[] | .Id + " " + .DefaultCacheBehavior.TargetOriginId'`
- `aws cloudfront list-distributions | jq '.DistributionList.Items[] | .Id + " " + .DefaultCacheBehavior.TargetOriginId' | grep fullscreen-video`
- `aws cloudfront create-invalidation --distribution-id XXXXXXXXXXXXXXXXXXXX --paths "/index.html"`
- `google-chrome fullscreen-video.rudijs.com`
