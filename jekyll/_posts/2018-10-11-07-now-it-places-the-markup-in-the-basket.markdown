---
layout: post
title:  "Part 7 - now it places the markup in the basket... - INCOMPLETE"
date:   2018-10-11 14:49:09 -0400
categories: Serverless Frontend
---

<img style="float: right; padding:10px 0 10px 10px" src="/images/basket.jpg">


add a note about how we're not gonna `remove` deployed stuff anymore unless something goes wrong.


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


<p align="center"><a href="/serverless/frontend/2018/10/11/08-speedy-delivery.html">Continue to Part 8</a></p>