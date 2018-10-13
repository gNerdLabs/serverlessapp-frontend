---
layout: post
title:  "Frontend 04 - Something something security"
date:   2018-10-11 14:49:12 -0400
categories: Serverless Frontend
---
<img style="float: right; padding:10px 0 10px 10px" src="/images/homer.jpg">
To make our S3 bucket work the way we want it to while being as secure as possible, we'll need to set up a proper bucket policy.

I intend to create some sort of addon to this tutorial or perhaps a standalone thing that really goes into what the policy is doing, how its structured, what each part means. (This one is not very complex but others will be) ... also just exactly how Serverless Framework and Cloudformation are generating the policies from the little bit of input we give them. Hopefully that addition will be forthcoming sooner rather than later. For now... perhaps this will be a resource to go deeper on this topic: (Understanding Permissions Granted by a Policy)[https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_understand.html]{:target="_blank"}

Back to you your resources section in `serverless.yml` ... we're going to add a new item called `AppS3BucketPolicy`:


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

As mentioned above, AWS policies are pretty far off track for this tutorial, I _will_ reiterate that if you don't understand what is going on in that section you should probably do some reading on policies. Long story short, we are defining a policy that allows anyone to read (only) any content from the bucket this policy is attached to.

Perhaps the real important takeaway from this section: you can see that the `Bucket` property of the new policy if a `Ref` ... we simply use the name of the resource we are referencing and it will figure out all the details behind the scenes. In this case we called our bucket `AppS3Bucket` so our `AppS3BucketPolicy` is attached to it.

We will use this reference technique quite a bit as we go deeper into Serverless Framework and Cloudformation.


<p align="center"><a href="/serverless/frontend/2018/10/11/05-moar-organizing-and-structure.html">Continue to Part 5</a></p>