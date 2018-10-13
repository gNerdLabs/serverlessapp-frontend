---
layout: post
title:  "Frontend 00 - WTF is this?"
date:   2018-10-11 14:49:16 -0400
categories: Serverless Frontend
---

If you want to skip ahead, the repo for this tutorial and application is available here: [https://github.com/gNerdLabs/serverlessapp-frontend](https://github.com/gNerdLabs/serverlessapp-frontend){:target="_blank"}

Otherwise...

**First of all, What is "serverless" anyway?**
<img style="float: right; padding:10px 0 10px 10px" src="/images/nerd-shit.jpg">
Serverless is a way to build and run virtually any application by using just the required services instead of a traditional server. Provisioning and scaling resources is either automatic or as easy to do as updating a configuration. There is no software to install or maintain, no updates or patches to be applied. Because the systems are all high availability and fault tolerant by default, there is no need to architect for those capabilities. So what you get is Highly available, massively scalable fault tolerant applications with virtually no maintenance or administration needs for the infrastructure.

**Ok... so what does that actually mean in practice?**

We'll start simple. What we'll build here is a small static site. No database integration or anything like that. Just a simple site served over SSL. We want it fast and available as well as secure and easy to update.

Instead of the standard approach of installing, updating and configuring a new server with all the bells and whistles. We are just going to select a service to provide each of the components of our application.

In this case:

- S3 for storage
- Cloudfront for distribution

Additionally we will use Route53 for DNS and we'll have Amazon generate our SSL certificate for us. We'll secure and limit access using AWS policies.

**What we'll cover**

This is not intended to be an in-depth guide to every aspect of the tech were using. While it is somewhat detailed in areas I may just link to another resource for things I cannot cover here at this time. Chances are when it comes to something like Yaml, someone else has already covered it and they've probably done a better job than I would anyway.

Given time I will come back and add information to areas where I feel it is lacking or could be better. Please email me and let me know if anything stands out to you.

All that said, I intend to walk through the creation of all the various bits mentioned above. I will go over the basic of how each resource is configured as well as doing some code organization that keeps each file as clear as possible. This is *my* preferred way of doing this... not necessarily why everyone would agree is *best practice* but not likely something anyone should hate on. If you dont like, it, feel free to restructure however you see fit. You should be able to take whatever you want from these tutorials and leave/change the things you dont need.

In the next section we'll briefly cover the installation and configuration of the tools and accounts you'll be using.

<p align="center"><a href="/serverless/frontend/2018/10/11/01-install-all-the-things.html">Continue to Part 1</a></p>

