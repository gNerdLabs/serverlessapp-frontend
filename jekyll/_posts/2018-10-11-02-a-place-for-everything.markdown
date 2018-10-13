---
layout: post
title:  "Frontend 02 - A place for everything and everything in its place"
date:   2018-10-11 14:49:14 -0400
categories: Serverless Frontend
---
Before we go too much further, it may be a good idea to set up an app configuration scheme... to make changes easier.

Start by creating a new file in your frontend directory named `config.dev.yml' we're going to add a bunch of custom configuration values in here.

We'll also introduce the `stage` concept here... We wont get too much into that in this tutorial as we're essentially building this straight to production. In the future as we do more it will be easier to have multiple stages if we build it in now. 

We're setting up this config for the "dev" environment so add this to the provider block:

```yml
stage: ${opt:stage, 'dev'}
```

If a stage is provided to serverless in the command line options, it will use the one provided, it will default to `dev` when nothing is provided.

Now we need to tell serverless to load our configuration file.

Add the following to your main yml file:

```
custom: ${file(config.${self:provider.stage}.yml)}
```

I usually make this the first line of my `serverless.yml` file so it now looks something like this:

```
custom: ${file(config.${self:provider.stage}.yml)}

provider:
  name: aws
  runtime: nodejs8.10
  stage: ${opt:stage, 'dev'}

service: serverlessapp-frontend

...
```

You shoul dbe able to see how the file import works. ${self:provider.stage} is set to `dev` by default so it loads `config.dev.yml` in as the value for the `custom` block. 

_*note: Scope for `self` gets a little odd the way I have this set up. It's easiest to just think of `self` as being at the main `serverless.yml` file root._

So, lets set up our config now...

Add the following to your config.dev.yml:

```
rev: 001
domainName: serverlessapp.net
cleanName: serverlessapp-frontend
appName: ${self:custom.cleanName}-app-rev${self:custom.rev}
runtime: nodejs8.10
profile: serverless-admin

```

- `rev` will be a somewhat arbitrary value we can use to tell what revision we are working on. Sometimes if a build or a remove fails it can leave remnantes that block us from reusing the exact app name again (at least until the system clears them out... which can take a while)  this is a habndy way to change the name of our next deploy so we dont have to wait.

- `domainName` should be pretty self explanatory

- `cleanName` is the base name for our app that all other names will be based off of, in this example we will use the vaule we previously had for `service` in the main yml file: serverlessapp-frontend

_*More on the note about scope from above..._

_Since we are accessing these config values from within the main file as `custom`, we do `self:custom` and then reference whichever value we want. This is instead of what might seem more intuitive to some... ${self:cleanName} or something like that._

_So to reference our custom values we use the following: `${self:custom.cleanName}` even when `custom` does not exist in the immediate file._

This allows us to create the following:

- `appName` comes out (in this example) as `serverlessapp-frontend-app-rev001` (how this is used will be more evident later on in the tutorials)

- `runtime` should be set to whatever you previously had in your main file. For this particular frontend application, we wont even be using this so it just needs to be set to something valid.

- `profile` while not strictly necessary is a way to specify which credentials you use for authenticating with aws... if you have multiple sets of credentials for different projects this comes in quite handy 

Now that we have out config set up we can swap out the parts of our main yml.

Update your `serverless.yml` so it looks like this:

```
custom: ${file(config.${self:provider.stage}.yml)}

provider:
  name: aws
  runtime: ${self:custom.runtime}
  stage: ${opt:stage, 'dev'}
  profile: ${self:custom.profile}

service: ${self:custom.appName}
```

_*note that the `profile` entry is a new line_


<p align="center"><a href="/serverless/frontend/2018/10/11/03-its-this-your-bucket.html">Continue to Part 3</a></p>


<!-- "Before, however, Lucy had been an hour in the house she had contrived a place for everything and put everything in its place."

- The Naughty Girl Won : Religious Tract Society in 1799 -->