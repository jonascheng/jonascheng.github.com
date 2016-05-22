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
---

# Preface 

As I posted the pain point in [Cognito with Unauthenticated Identities in Javascript]({% post_url 2016-05-19-cognito-with-unauthenticated-identities %}), I'd like to share how you achieve in iOS.

If you didn't read previous post, I'd recommend you reading it first before you move on.

# Overview

In this sample code, I'd like to invoke **MOCK** API Gateway with Cognito SDK in iOS and the service which would only response to client (mobile app) with signing requests. 

* Generate an SDK for an API with the API Gateway Console
* Integrate an API Gateway-Generated iOS SDK into Your iOS Project

# Steps by Steps

### Generate an SDK for an API with the API Gateway Console

* Sign in to the [API Gateway console](https://console.aws.amazon.com/apigateway)
* Under `APIs\MOCK\Stages`, click on `dev`
* On the `SDK Generation` tab, for `Platform`, choose the platform `iOS`.
* In the Prefix box, type the unique prefix `CLI` for the generated classes.
* Choose `Generate SDK`, and then follow the on-screen directions to download the API Gateway-generated SDK.

### Integrate an API Gateway-Generated iOS SDK into Your iOS Project

The generated SDK depends on the AWS Mobile SDK for iOS. There are two ways to import it into your project:

* CocoaPods
* Frameworks

You should use one of these two ways to import the AWS Mobile SDK but not both. Importing both ways loads two copies of the SDK into the project and causes compiler errors.

#### With CocoaPods

* The AWS Mobile SDK for iOS is available through [CocoaPods](https://cocoapods.org/). If you have not installed CocoaPods, install it by running the command:

        $ sudo gem install cocoapods

* Move the generated `Podfile` to the same directory as your Xcode project file. If your project already has `Podfile`, you can add the following line to the existing `Podfile`.

        pod 'AWSAPIGateway', '~> 2.3.2'

* Then run the following command:

        $ pod install

* Open up `*.xcworkspace` with Xcode.
* Add all files (`*.h` and `*.m` files) under the `generated-src` directory to your Xcode project.

#### With Frameworks

* Download the SDK from [http://aws.amazon.com/mobile/sdk](http://aws.amazon.com/mobile/sdk). Amazon API Gateway is supported with the version 2.2.1 and later.
* With your project open in Xcode, <kbd>Control</kbd>+click **Frameworks** and then click **Add files to "\<project name\>"...**.
* In Finder, navigate to the `AWSCore.framework` and `AWSAPIGateway.framework` files, select them, and click **Add**.
* Open a target for your project, select **Build Phases**, expand **Link Binary With Libraries**, click the **+** button, and add `libsqlite3.dylib`, `libz.dylib`, and `SystemConfiguration.framework`.

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