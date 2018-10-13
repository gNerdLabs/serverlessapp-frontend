---
layout: post
title:  "Part 8 - Speedy Delivery! - INCOMPLETE"
date:   2018-10-11 14:49:08 -0400
categories: Serverless Frontend
---

<img style="float: right; padding:10px 0 10px 10px" src="/images/speedy.jpg">

next we add dns and cloudfront

We're going to be using cloudfront to serve the contents of our s3 bucket over ssl... so this is the part that really ties it all together...

first thing we need to do is add an array of domains to our config file. These are the aliases our cloudfront distribution will respond to.

in the config file, add:

```
cfAliases:
  - serverlessapp.net
  - www.serverlessapp.net
```

obviously using your own domain name.

now, create a new file in your respurces directory called `cf_distro.yml` and insert the following code:

```
---
Type: AWS::CloudFront::Distribution
Properties:
  DistributionConfig:
    Origins:
      - DomainName:
          Fn::Join: [
            "", [
              { "Ref": "AppS3Bucket" },
              ".s3.amazonaws.com"
            ]
          ]
        Id:
          Ref: AppS3Bucket
        CustomOriginConfig:
          HTTPPort: 80
          HTTPSPort: 443
          OriginProtocolPolicy: https-only
    Enabled: 'true'
    Aliases: ${file(config.${self:provider.stage}.yml):cfAliases}
    DefaultRootObject: index.html
    CustomErrorResponses:
      - ErrorCode: 404
        ResponseCode: 200
        ResponsePagePath: /index.html
    DefaultCacheBehavior:
      AllowedMethods:
        - GET
        - HEAD
      TargetOriginId:
        Ref: AppS3Bucket
      ForwardedValues:
        QueryString: 'false'
        Cookies:
          Forward: none
      ViewerProtocolPolicy: redirect-to-https
    ViewerCertificate:
      # AcmCertificateArn: ${file(config.${self:provider.stage}.yml):sslCertArn} # use this if using an existing cert
      AcmCertificateArn:
        Ref: StaticSiteCert
      SslSupportMethod: sni-only
DependsOn:
  - AppS3BucketPolicy
```

--- if time allows, add a breakdown of whats going on here ---

note: if using a manually created aws ssl cert as mentioned above, you'll need to swap:

```
AcmCertificateArn:
  Ref: StaticSiteCert
```

for 

```
AcmCertificateArn: ${file(config.${self:provider.stage}.yml):sslCertArn}
```

then add `sslCertArn` to your config file where the value is that of your certificates arn.

e.g.

```
sslCertArn: arn:aws:acm:region:000000000000:certificate/0000000e-0e00-000d-ad0c-a0c000000000
```

otherwise... the reference to our own resource should suffice.

note about cloudfront distibution changes in aws... they take forever to update so we want to do as few updates as possible here. mistakes suck because removing them and rebuilding is a really long wait between tests. 

so, hopefully we have everything right in the first try ;)

<p align="center"><a href="/serverless/frontend/2018/10/11/09-DNS.html">Continue to Part 9</a></p>