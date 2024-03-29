# Static website hosting with S3, Cloudfront and SSL

* Creates an S3 bucket (with website enabled) in a region of your choice using your domain as name.
* Creates an SSL certificate with ACM in us-east-1 for your site domain.
* Creates a Cloudfront distribution with your site domain as alias, using the above certificate.

## Why such a complicated template for something so basic?
Because AWS... Sadly AWS allows you to create a Cloudfront distribution from any region, but the certificate needed for your distribution needs to be created in us-east-1.

Since you have good reasons to create the resources of your stack in your preferred region, you need to jump through hoops just for an SSL certificate. If you think this is crazy, [your are not alone](https://github.com/aws-cloudformation/cloudformation-coverage-roadmap/issues/523).

## Prerequisites
* AWSCloudFormationStackSetAdministrationRole and AWSCloudFormationStackSetExecutionRole exists

  If not, install them from [StackSetsRoles](StackSetsRoles).
* The DNS for your domain needs to be defined with Route53
* You need the required access rights to deploy the resources in the stack and stackSet

## Create a parameter file
_parameters.json_
```json
[
  {
    "ParameterKey": "DomainName",
    "ParameterValue": "FQDN for your site (blog.example.com)"
  },
  {
    "ParameterKey": "HostedZoneId",
    "ParameterValue": "HostedZoneId from Route53"
  }
]
```
### Get HostedZoneId
```bash
export AWS_PROFILE=myProfile
domain=example.com
site=myblog
hostedZoneId=$(aws route53 list-hosted-zones-by-name \
--dns-name $domain \
--query "HostedZones[0].Id" \
--output text | \
cut -d/ -f3)
site=$site domain=$domain hostedZoneId=$hostedZoneId envsubst < parameters.template.json  > paramteres.json
```
## Deploy
```bash
export AWS_PROFILE=myProfile
export AWS_REGION=eu-central-1
aws cloudformation create-stack \
  --stack-name myBlog \
  --template-body file://cf-s3-static.yml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters file://parameters.json
```
This will create the main stack in eu-central-1 and a stack in us-east-1 to generate the SSL certificate for Cloudfront.

### Optional
Upload your index.html and 404.html
```bash
export AWS_PROFILE=myProfile
fqdn=blog.example.com
aws s3 sync public/* --cache-control max-age=3600 s3://${fqdn}/
```
