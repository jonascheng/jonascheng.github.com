---
layout: post
title: "got 500 error as generating API Gateway SDK"
image: '/assets/img/'
description:
main-class: trap
tags:
- "api gateway"
categories: trap
twitter_text: "Got 500 error as generating API Gateway SDK"
introduction: "The issue has been confirmed by AWS support team, you may work around if you encounter the same symptom"
published: true
---

# Symptom

As I click `Generate SDK` in `SDK Generation` tab under **API Gateway**, I always encounter **500** error in screenshot below.

![]({{site.url}}{{ page.image }}2016-05-25-got-500-error-as-generating-api-gateway-sdk-1.png)

I found the issue could be reproduced after I define **204** in `Method Response` below.

![]({{site.url}}{{ page.image }}2016-05-25-got-500-error-as-generating-api-gateway-sdk-2.png)

After contacting AWS support and API Gateway team, they have indicated that while you can have/deploy an API with multiple **2XX** responses, the SDK generation only supports a single **2xx** response.

AWS team have therefore opened this as a feature request (refer to BPL-4495) and has been added to their backlog.


