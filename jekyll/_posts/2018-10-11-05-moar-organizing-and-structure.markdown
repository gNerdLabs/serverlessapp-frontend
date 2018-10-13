---
layout: post
title:  "Frontend 05 - Moar organizing and structure!"
date:   2018-10-11 14:49:11 -0400
categories: Serverless Frontend
---
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


<p align="center"><a href="/serverless/frontend/2018/10/11/06-free.ssl.html">Continue to Part 6</a></p>