Assume AWS account exists...
Make new IAM role for sreverless
Set up/configure aws/serverless

build the frontend first (no app just a static page or something)



installing serverless framework, node and aws cli

link to install node:
https://nodejs.org/en/download/package-manager/
or https://nodejs.org/en/download/

link to get sreverless:
https://serverless.com

* npm install -g serverless

IAM user role

go to IAM
click users
add user
serverless-admin
programatic access
Next
attach existing
AdministratorAccess
Next
Create user
download csv file.

serverless config credentials --provider aws --key ___ --secret ___ --profile serverless-admin

create project directory

create frontend app:

serverless create --template aws-nodejs --path frontend

go into frontend app and open in editor

3 files, a base .gitignore, jandler.js and the serverless.ym file

delete the handler as we wont need it for this one 

open up the yml file

if youre not familliar with yml it may be a good idea to read up on it or look at some yaml specific tutorials (see if you can find some)


go ahead and delete all the commented commented

should now look something liek 

```
---
service: frontend 

provider:
  name: aws
  runtime: nodejs8.10

functions:
  hello:
    handler: handler.hello
```
we dont need the functions section so get rid of it too.

change the service name to something appropriate (I also moved it below provider out of personal preference)

```
---
provider:
  name: aws
  runtime: nodejs8.10

service: serverlessapp-frontend
```

before we go too much further, it may be a good idea to set up a config file... to make changes easier

create a new file in your frontend directory named `config.dev.yml'  we're going to add a bunch of custom configuration values in here.

we're setting up this config for the "dev" environment so lets start using a stage value. Add this to the provider block:

```
stage: ${opt:stage, 'dev'}
```

if a stage is provided in the command line it will use that, it will default to dev when not provided.

now we need to tell serverless to load our file.

add the following to your main yml file:

```
custom: ${file(config.${self:provider.stage}.yml)}
```

I make this the first line of mine so it now looks something like this:

```
---
custom: ${file(config.${self:provider.stage}.yml)}

provider:
  name: aws
  runtime: nodejs8.10
  stage: ${opt:stage, 'dev'}

service: serverlessapp-frontend

...
```

you can now see how the file import works. ${self:provider.stage} is set to `dev` by default so it loads config.dev.yml in as the value for the `custom` block. 

(note: Scope for `self` gats a little odd the way I have this set up. its easiest to think of `self` as being at the main serverless.yml file root.)

so, lets set up our config now

add the following to your config.dev.yml:

```
---
rev: 001
domainName: serverlessapp.net
cleanName: serverlessapp-frontend
appName: ${self:custom.cleanName}-app-rev${self:custom.rev}
runtime: nodejs8.10
profile: serverless-admin

```
`rev` will be a somewhat arbitrary value we can use to tell what revision we are working on. Sometimes if a build or a remove fails it can leave remnantes that block us from reusing the exact app name again (at least until the system clears them out... which can take a while)  this is a habndy way to change the name of our next deploy so we dont have to wait.

`domainName` should be pretty self explanatory

`cleanName` is the base name for our app that all other names will be based off of, in this example we will use the vaule we previously had for `service` in the main yml file: serverlessapp-frontend

More on the note about scope from above... 

since we are accessing these config values from within the main file as `custom`, we do `self:custom` and then whichever value we want. This is instead of what might seem more intuitive to some... ${self:cleanName} or somethin glike that. 

so to reference our custon values we use the following: ${self:custom.cleanName} even when `custom` does not exist in the immediate file.

this allows us to create the following:

`appName` comes out (in this example) as `serverlessapp-frontend-app-rev001`

how this is used will be more evident later on in the tutorials

`runtime` should be set to whatever you previously had in your main file. For this particular frontend application, we wont even be using this so it just needs to be set to something valid.

`profile` while not strictly necessary is a way to specify which credentials you use for authenticating with aws... if you have multiple sets of credentials for different projects this comes in quite handy 

Now that we have out config set up we can swap out the parts of our main yml that we have set up.

update your serverless.yml so it looks like this:

```
---
custom: ${file(config.${self:provider.stage}.yml)}

provider:
  name: aws
  runtime: ${self:custom.runtime}
  stage: ${opt:stage, 'dev'}
  profile: ${self:custom.profile}

service: ${self:custom.appName}
```

the only addition is the `profile` entry.

Now we can move on to actually creating the s3 bucket. 

the s3 bucket is where we'll store all of our frontend html and whatnot.

lets add that now.

start with naming the bucket... 

add a new value to your config file:

```
s3Bucket: ${self:custom.cleanName}-www-rev${self:custom.rev}
```

much like `appName`, the value of `s3Bucket` is computed as `serverlessapp-frontend-www-rev001`

we can now ause that value in out serverless.yml file.  add the following block there:

```
resources:
  Resources:
    AppS3Bucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: ${self:custom.appName}
        AccessControl: PublicRead
        WebsiteConfiguration:
          IndexDocument: index.html
          ErrorDocument: index.html
```

this is our resources block, it's where we will put all the definition for the components of our app

we have defined a resource clled `AppS3Bucket`  the type is standard AWS naming for their S3 service: `AWS::S3::Bucket`

We've defined three properties for this resource, BucketName, AccessControl and WebsiteConfiguration. 

`BucketName` wil be out custom name: `serverlessapp-frontend-www-rev001`

We are allowing public read since we want this to serve our www assets directly to users. As static www root.

lastly we need to tell S3 what files to serve by default. 

lets test this out!

to dploy the project so far use the following command:

`serverless deploy -v`

the -v is going to give you more verbose output so you can see more of whats happening.

assuming everything worked out you should see something like this:

```
D:\serverlessapp-tut\frontend>serverless deploy -v
Serverless: WARNING: Missing "tenant" and "app" properties in serverless.yml. Without these properties, you can not publish the service to the Serverless Platform.
Serverless: Packaging service...
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
CloudFormation - CREATE_IN_PROGRESS - AWS::CloudFormation::Stack - serverlessapp-frontend-app-rev1-dev
CloudFormation - CREATE_IN_PROGRESS - AWS::S3::Bucket - ServerlessDeploymentBucket
CloudFormation - CREATE_IN_PROGRESS - AWS::S3::Bucket - ServerlessDeploymentBucket
CloudFormation - CREATE_COMPLETE - AWS::S3::Bucket - ServerlessDeploymentBucket
CloudFormation - CREATE_COMPLETE - AWS::CloudFormation::Stack - serverlessapp-frontend-app-rev1-dev
Serverless: Stack create finished...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading artifacts...
Serverless: Validating template...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
CloudFormation - CREATE_COMPLETE - AWS::CloudFormation::Stack - serverlessapp-frontend-app-rev1-dev
CloudFormation - UPDATE_IN_PROGRESS - AWS::CloudFormation::Stack - serverlessapp-frontend-app-rev1-dev
CloudFormation - CREATE_IN_PROGRESS - AWS::S3::Bucket - AppS3Bucket
CloudFormation - CREATE_IN_PROGRESS - AWS::S3::Bucket - AppS3Bucket
CloudFormation - CREATE_COMPLETE - AWS::S3::Bucket - AppS3Bucket
CloudFormation - UPDATE_COMPLETE_CLEANUP_IN_PROGRESS - AWS::CloudFormation::Stack - serverlessapp-frontend-app-rev1-dev
CloudFormation - UPDATE_COMPLETE - AWS::CloudFormation::Stack - serverlessapp-frontend-app-rev1-dev
Serverless: Stack update finished...
Service Information
service: serverlessapp-frontend-app-rev1
stage: dev
region: us-east-1
stack: serverlessapp-frontend-app-rev1-dev
api keys:
  None
endpoints:
  None
functions:
  None

Stack Outputs
ServerlessDeploymentBucketName: serverlessapp-frontend-a-serverlessdeploymentbuck-9kfom02mrohh
```

If you got no errors we can go back to aws to check out the bucket we created.

navigate to Services -> S3 from your aws console.

you will see two new buckets.

one of them will be named to match whatever string the server returned at the end of the deploy output `ServerlessDeploymentBucketName`  in my case `serverlessapp-frontend-a-serverlessdeploymentbuck-9kfom02mrohh`  this is where it puts the cloudformation code that is built by serverless framework. In other applications it may have python code to run on lambda, etc. if you tunnel in a bit you will see where it uses the value you set for `appName` in your config as well. (e.g. serverlessapp-frontend-app-rev001 for my example)

you should also see the s3 bucket you defined... mine is called `serverlessapp-frontend-www-rev001` as expected. it is empty and rather boring... just a basic s3 bucket for now.

once we've verified it works, we want to remove both buckets.  this is another simple serveless copmmand:

```
sls remove
```

once done, you can verify it worked by refreshing your s3 console.

now back to the yml...

to make the s3 bucket work as we want, we need to set up a bucket policy:

in your resources section, we're gonna add a new itrem called `AppS3BucketPolicy`:

this is the new item we're adding:
```
AppS3BucketPolicy:
  Type: AWS::S3::BucketPolicy
  Properties:
    Bucket:
      Ref: AppS3Bucket
    PolicyDocument:
      Statement:
      - Sid: PublicReadGetObject
        Effect: Allow
        Principal: '*'
        Action:
          - s3:GetObject
        Resource:
          Fn::Join: [
            "", [
              "arn:aws:s3:::",
              {
                "Ref": "AppS3Bucket"
              },
              "/*"
            ]
          ]
```
your resources block should now look like this:

```
resources:
  Resources:
    AppS3Bucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: ${self:custom.s3Bucket}
        AccessControl: PublicRead
        WebsiteConfiguration:
          IndexDocument: index.html
          ErrorDocument: index.html
    AppS3BucketPolicy:
      Type: AWS::S3::BucketPolicy
      Properties:
        Bucket:
          Ref: AppS3Bucket
        PolicyDocument:
          Statement:
            - Sid: PublicReadGetObject
              Effect: Allow
              Principal: '*'
              Action:
                - s3:GetObject
              Resource:
                Fn::Join: [
                  "", [
                    "arn:aws:s3:::",
                    {
                      "Ref": "AppS3Bucket"
                    },
                    "/*"
                  ]
                ]
```

Just like the S3 bucket we have a new item with a Type and some properties.

Without going into the `PolicyDocument` because aws policies are pretty far off track for this tutorial, I will say that if you dont understand what is going on in that section you shoul dprobably do some reading on policies. Long story short, we are defining a policy here that allows anyone to read any content from the bucket this policy is attached to.

That is the real important takeaway from this item... you can see that the `Bucket` property of the new policy if a `Ref` ... we simplu use the name of the resource we are referencing and it will figure out all the details behind the scenes. In this case we called our bucket `AppS3Bucket` so our `AppS3BucketPolicy` is attached to it. 


At this point we should do a littl emore oganization. This app is pretty small but will still benefit from a better code structure. Lets set up all out resources as includes, like we did our configuration...

create a new folder called `resources`
within that folder create a file named `app_bucket.yml` or something clear like that.

we're going to copy and paste our definition of `AppS3Bucket` into that file:

```
---
Type: AWS::S3::Bucket
Properties:
  BucketName: ${self:custom.s3Bucket}
  AccessControl: PublicRead
  WebsiteConfiguration:
    IndexDocument: index.html
    ErrorDocument: index.html
```

and do the same for `AppS3BucketPolicy` in a file called `app_bucket_policy.yml`

```
---
Type: AWS::S3::BucketPolicy
Properties:
  Bucket:
    Ref: AppS3Bucket
  PolicyDocument:
    Statement:
      - Sid: PublicReadGetObject
        Effect: Allow
        Principal: '*'
        Action:
          - s3:GetObject
        Resource:
          Fn::Join: [
            "", [
              "arn:aws:s3:::",
              {
                "Ref": "AppS3Bucket"
              },
              "/*"
            ]
          ]
```

remember to be careful and clean up your indentation as needed. yml is dependent on indentation.

now we can update the resources section of our main file to look like this:

```
resources:
  Resources:
    AppS3Bucket: ${file(resources/app_bucket.yml)}
    AppS3BucketPolicy: ${file(resources/app_bucket_policy.yml)}
```
which has eliminated ~25 lines of noisy code from our main document. 

There is one last thing we need to change before we can dest the deploy again.

our `BucketName` value in the properties for `AppS3Bucket` could be more direct. Rather than using a relative reference that in my opinion is counterintuitive, lets get that value directly from the source.  change the value of `BucketName` in 'app_bucket.yml' like so:

```
BucketName: ${file(config.${self:provider.stage}.yml):s3Bucket}
```

Now, Im not sure this is strictly necessary. It would probably work fine without doing this... I havent tested it with the relative variable reference so I dont know. My personal preference is to do it this way but its also an opportunity to point out one more method of referencing data from other files.

we are targeting the file the same way we do when including the whole  thing for each of our resources... in this case though we are telling it the name of a single variable, `s3Bucket`, we want to retrieve. 

Its also a chance to point out that the file path is relative to the file path is relative to the main serverlass.yml file... not to the file the code is actually in. i.e. we access it via `config.dev.yml` as opposed to something like `../config.dev.yml` because we are inside the `resources` directory.

go ahead and deploy again, verify it still works and then remove it.

`sls daploy -v` and `sls remove`

one of the nice things about using aws is that we can use free ssl for public sites. lets set up a new ssl certificate as part of our project here.

first we need to make sure we  have a hosted zone on route53. this is still poaaible if your dns is hosted elsewhere but for this tutorial Im keeping it all in house. 

--- section on dns zone ---

once your zone is all set up, create a new file in your resources directory: `ssl_certificate.yml' and place the following yml in it:

```
---
Type: AWS::CertificateManager::Certificate
Properties:
  DomainName: ${file(config.${self:provider.stage}.yml):domainName}
  SubjectAlternativeNames:
    - www.serverlessapp.net
  ValidationMethod: DNS
  DomainValidationOptions:
  - DomainName: ${file(config.${self:provider.stage}.yml):domainName}
    ValidationDomain: ${file(config.${self:provider.stage}.yml):hostedZoneName}
  - DomainName: www.serverlessapp.net
    ValidationDomain: ${file(config.${self:provider.stage}.yml):hostedZoneName}
```

By now you are familiar with the Type and Properties concept. You can see we're using the domain name we set earlier in our config file. I have opted to include a www subdomain as an alternate here (`SubjectAlternativeNames`) as well, so that is hardcoded.  In a future verison of this I may set it up to be more dynamic. Since Im not doing that today, I will probably be hardcoding my default stage to prod from here on out. 

`ValidationMethod` is set to `DNS`... this will allow for really easy verification of the domain so the ssl certificate can be issued. Another option is `Email` ... as I mentioned above, you can do the certificate another way if you dont have a hosted zon for your domain in route53

you will also see we are referencing a new config variable: `hostedZoneName`  this should be added to your config file like so:

```
hostedZoneName:serverlessapp.net
```

as shown above when creating the zone, we end up with the domain name value as the zone name.

each domain name in the certificate will need to be verified independently so will need its own entry in the `DomainValidationOptions` array.

`DomainName` is the actual domain name that is added to the ssl certificate and secured, `ValidationDomain` will, in cases like this, be the same value for all of them. 

It's not difficult to manually create the certificate. If you do that, I believe you can use a wildcard if you choose to do so. I dont believe serverless framework currently supports that option out of the box. I'll note the changes later on if you are using a manually created certificate. IF so, just ignore this whole section.

now that we have the definition, we need to add the cert to our resources in serverless.yml:

StaticSiteCert: ${file(resources/ssl_certificate.yml)}

Lets deploy again. I'll need to make a video of this process probably...

-- do deploy stuff here ---

note about how we're not gonna remove stuff anymore unless something goes wrong.


lets get some html in the bucket so we can look at it...

create a new folder called `app`

create in the folder an `index.html` file

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>ServerlessApp.net tutorial -- html boilerplate</title>
	<link rel="stylesheet" href="css/style.css">
	<!--[if IE]>
		<script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
	<![endif]-->
</head>

<body id="home">

	<h1>This is just a placeholder!</h1>

</body>
</html>
```

we can sync our app dir with the s3 bucket really easily using a serverless framework plugin...

this will add some utility commands for us... one does the file sync, the other will come in handy later...

easy install:

```
npm install git+https://github.com/gNerdLabs/serverless-single-page-app-plugin.git
```

and add it to serverless.yml:

```
plugins:
  - serverless-single-page-app-plugin
```


run the command:

```
sls syncToS3
```

depending on your environment you may get an error here:

if you see "Serverless: fatal error: Unable to locate credentials"  try the following:

run the command `aws configure` and enter the credentials from before.  They should be in `~/.aws/credentials`

just hit `enter` to use default values for the rest of the prompts

this will make a new "default" profile for you and that is the one that the plugin will use.

The plugin works by simply executing commands for you in the aws CLI. Without the `default` profile it cannot authenticate.

--- maybe use default above to avoid this? ---

once you have it working, the command

```
sls syncToS3
```

should show something like:

```
upload: app\index.html to s3://serverlessapp-frontend-www-rev1/index.htmlining

Serverless: Successfully synced to the S3 bucket
```

Now go to your S3 bucket in the aws console.

click on the properties tab, click the "Static website hosting" box and you should see an `endpoint` link... clicking it should show you the "This is just a placeholder!" text from our index.html file.





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

before we deploy again we should add out dns records to point or domain names to the cf distribution. 

we're gonna do one resource for each domain.

first, we also need to add one more config variable:

```
hostedZoneId: Z2FDTNDATAQYW2
```

that is the zoneID for cloudfront... this is a static value provided by amazon. 

now, create the first file in the resources directory:

`dns_record.yml`

should contain:

```
---
Type: "AWS::Route53::RecordSet"
Properties:
  AliasTarget:
    DNSName:
      Fn::GetAtt:
        - CloudFrontDistribution
        - DomainName
    HostedZoneId: ${file(config.${self:provider.stage}.yml):hostedZoneId}
  HostedZoneName: ${file(config.${self:provider.stage}.yml):hostedZoneName}.
  Name: ${file(config.${self:provider.stage}.yml):hostedZoneName}.
  Type: 'A'
```

then the second file:

`dns_record2.yml`

should contain:

```
---
Type: "AWS::Route53::RecordSet"
Properties:
  AliasTarget:
    DNSName:
      Fn::GetAtt:
        - CloudFrontDistribution
        - DomainName
    HostedZoneId: ${file(config.${self:provider.stage}.yml):hostedZoneId}
  HostedZoneName: ${file(config.${self:provider.stage}.yml):hostedZoneName}.
  Name: www.serverlessapp.net.
  Type: 'A'
```

you'll see that the `Name` value is really the only difference here.

link it all up in the serverless.yml file by adding these three lines to the resources block:

```
CloudFrontDistribution: ${file(resources/cf_distro.yml)}
DnsRecord: ${file(resources/dns_record.yml)}
DnsRecord2: ${file(resources/dns_record.2.yml)}
```

deploy now...


---------------------------

create backend app:

sls create -t aws-python3 -p api



npm install git+https://github.com/gNerdLabs/serverless-single-page-app-plugin.git