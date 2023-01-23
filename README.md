# [Amazon CloudFront Secure Static Website](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/getting-started-secure-static-website-cloudformation-template.html)

Review the documentation on the AWS link above. Then follow the instructions on this README to deploy the solution.

## CloudFront Response Header Policy  
In `templates/cloudfront-site.yaml` edit the Content Security Policy for the response headers for the CloudFront distribution, if needed. Examples shown in this fork:
- Your app needs to load a script from a CDN on a different domain.  
- Your app needs to connect to an API or load images from another domain.
- You need to load your app's manifest file.  

<details> 

<summary>More details</summary>

The CloudFront Response Header Policy adds security headers to every response served by CloudFront.

The security headers can help mitigate some attacks, as explained in the [Amazon CloudFront - Understanding response header policies documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/understanding-response-headers-policies.html#understanding-response-headers-policies-security). Security headers are a group of headers in the web server response that tell web browsers to take extra security precautions. This solution adds the following headers to each response:

- [Strict-Transport-Security](https://infosec.mozilla.org/guidelines/web_security#http-strict-transport-security)
- [Content-Security-Policy](https://infosec.mozilla.org/guidelines/web_security#content-security-policy)
- [X-Content-Type-Options](https://infosec.mozilla.org/guidelines/web_security#x-content-type-options)
- [X-Frame-Options](https://infosec.mozilla.org/guidelines/web_security#x-frame-options)
- [X-XSS-Protection](https://infosec.mozilla.org/guidelines/web_security#x-xss-protection)
- [Referrer-Policy](https://infosec.mozilla.org/guidelines/web_security#referrer-policy)

For more information, see [Mozilla’s web security guidelines](https://infosec.mozilla.org/guidelines/web_security).

</details>  

<p>

## CloudFront Price Class

In `templates/cloudfront-site.yaml` edit the CloudFront distribution [`PriceClass`](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PriceClass.html?icmpid=docs_cf_help_panel) with one of [these options](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-cloudfront-distribution-distributionconfig.html#cfn-cloudfront-distribution-distributionconfig-priceclass) to match your needs.  

## Deployment  

> ⚠️ This template can only be deployed in the `us-east-1` region.  

**CreateApex:** Optionally create an Alias to the domain apex (example.com) in your CloudFront configuration.  Default is `no`.

1. Create or empty a folder for your site content with `mkdir www` or `rm -r ./www/*`, respectively.  
2. Deploy the solution with these commands:
```shell
make package-static

cp -r /path/to/my-site/build/* ./www/

ARTIFACTS=solution-artifacts
STACK=static-site-stack
DOMAIN=example.com
SUBDOMAIN=www
HOSTEDZONE=XXXXXXXXXXXXXXXXXXXXX
CREATEAPEX=no

aws s3 mb s3://$ARTIFACTS --region us-east-1

aws cloudformation package \
    --region us-east-1 \
    --template-file templates/main.yaml \
    --s3-bucket $ARTIFACTS \
    --output-template-file packaged.template

aws cloudformation deploy \
    --region us-east-1 \
    --stack-name $STACK \
    --template-file packaged.template \
    --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
    --parameter-overrides DomainName=$DOMAIN SubDomain=$SUBDOMAIN HostedZoneId=$HOSTEDZONE CreateApex=$CREATEAPEX
```

After your site content has been uploaded to its S3 bucket it will not be updated on subsequent runs of `aws cloudformation deploy`. You'll need to upload it manually to the bucket with the name contaning `s3bucketroot`.
```shell
aws s3 sync /path/to/my-site/build s3://my-bucket/path --delete
```
If you need to remove a file from CloudFront edge caches before it expires, follow [these intructions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html).