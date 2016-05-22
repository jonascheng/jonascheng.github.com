---
layout: post
title: "cognito with unauthenticated identities in iOS"
image: '/assets/img/'
description: "In this post, I will demo you how to use Cognito Identity Pool to authorize unauthenticated clients to invoke API Gateway in iOS"
main-class: aws
tags:
- "identify pool"
- "api gateway"
- "cognito"
- "ios"
categories: aws
twitter_text: "Cognito with Unauthenticated Identities in iOS"
introduction: "Use Cognito Identity Pool to authorize unauthenticated clients to invoke API Gateway in iOS"
published: false
---

# Preface 

As I posted the pain point in [Cognito with Unauthenticated Identities in Javascript]({% post_url 2016-05-19-cognito-with-unauthenticated-identities %}), I'd like to share how you achieve in iOS.

If you didn't read previous post, I'd recommend you reading it first before you move on.

### Invoke MOCK API Gateway with Cognito SDK in iOS

* Login `AWS Management Console` and open `Amazon API Gateway`
* Under `APIs\MOCK\Stages`, click on `dev`
* Select tab `SDK Generation`
* Set `Platform` as `Javascript`
* Click `Generate SDK` and download an archived file
* Decompress the download file to a folder, say `~\foo` 
* Download [aws-sdk](https://github.com/aws/aws-sdk-js/releases) from Github
* Decompress the download file to the previous folder
* Create a static index.html with the following content in the previous folder


* Create a static script.js with the following content in the previous folder

    Replace [region] and [identity-pool-id] first 

> You ONLY need to invoke `AWS.CognitoIdentity.getId()` first time or as you receive `ResourceNotFoundException: Identity 'identity-id' not found.`
> 

* Open index.html in browser, and you should be able to invoke the API successfully and read a prompt message `Hello foo!`

> To use an API key with the API Gateway-generated SDK, you can pass the API key as a parameter to the Factory object by using code similar to the following. If you use an API key, it is specified as part of the x-api-key header and all requests to the API will be signed.
>
> var apigClient = apigClientFactory.newClient({
  apiKey: 'API_KEY'
});


# References

* [AWS Serverless Multi-Tier Architectures](https://d0.awsstatic.com/whitepapers/AWS_Serverless_Multi-Tier_Architectures.pdf)
* [Enable CORS for a Resource through a Method in API Gateway](http://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)
* [Identify Pool](http://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html)
* [Generate an SDK for an API in API Gateway](http://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-generate-sdk.html)