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
published: true
---

# Preface 

As I posted the pain point in [Cognito with Unauthenticated Identities in Javascript]({% post_url 2016-05-19-cognito-with-unauthenticated-identities %}), I'd like to share how you achieve in iOS.

If you didn't read previous post, I'd recommend you reading it first before you move on.

# Overview

In this sample code, I'd like to invoke **MOCK** API Gateway with Cognito SDK in iOS and the service which would only response to client (mobile app) with signing requests. 

* Generate an SDK for an API with the API Gateway Console
* Integrate an API Gateway-Generated iOS SDK into Your iOS Project
*Use the SDK in your project

# Steps by Steps

## Generate an SDK for an API with the API Gateway Console

* Sign in to the [API Gateway console](https://console.aws.amazon.com/apigateway)
* Under `APIs\MOCK\Stages`, click on `dev`
* On the `SDK Generation` tab, for `Platform`, choose the platform `iOS`.
* In the Prefix box, type the unique prefix `CLI` for the generated classes.
* Choose `Generate SDK`, and then follow the on-screen directions to download the API Gateway-generated SDK.

## Integrate an API Gateway-Generated iOS SDK into Your iOS Project

The generated SDK depends on the [AWS Mobile SDK for iOS](http://docs.aws.amazon.com/mobile/sdkforios/developerguide/setup.html). There are two ways to import it into your project:

* CocoaPods
* Frameworks

You should use one of these two ways to import the AWS Mobile SDK but not both. Importing both ways loads two copies of the SDK into the project and causes compiler errors.

### With CocoaPods

* The AWS Mobile SDK for iOS is available through [CocoaPods](https://cocoapods.org/). If you have not installed CocoaPods, install it by running the command:

        $ sudo gem install cocoapods

* Move the generated `Podfile` to the same directory as your Xcode project file. If your project already has `Podfile`, you can add the following line to the existing `Podfile`.

        pod 'AWSAPIGateway', '~> 2.3.2'

* Then run the following command:

        $ pod install

* Open up `*.xcworkspace` with Xcode.
* Add all files (`*.h` and `*.m` files) under the `generated-src` directory to your Xcode project.

### With Frameworks

* Download the SDK from [http://aws.amazon.com/mobile/sdk](http://aws.amazon.com/mobile/sdk). Amazon API Gateway is supported with the version 2.2.1 and later.
* With your project open in Xcode, <kbd>Control</kbd>+click **Frameworks** and then click **Add files to "\<project name\>"...**.
* In Finder, navigate to the `AWSCore.framework` and `AWSAPIGateway.framework` files, select them, and click **Add**.
* Open a target for your project, select **Build Phases**, expand **Link Binary With Libraries**, click the **+** button, and add `libsqlite3.dylib`, `libz.dylib`, and `SystemConfiguration.framework`.

## Use the SDK in your project

First import the AWSCore and the generated header files

```objc
#import <AWSCore/AWSCore.h>
#import "CLIMOCKClient.h"
```

To use AWS IAM to authorize API calls you should set an `AWSCognitoCredentialsProvider` object as the default provider for the SDK.

> Replace [AWSRegionAPNortheast1] and [identity-pool-id] first

```objc
AWSCognitoCredentialsProvider *credentialsProvider = [[AWSCognitoCredentialsProvider alloc] 
        initWithRegionType:AWSRegionAPNortheast1    // Replace with the region you deploy
        identityPoolId:@"identity-pool-id"];
AWSServiceConfiguration *configuration = [[AWSServiceConfiguration alloc]
        initWithRegion:AWSRegionAPNortheast1 
        credentialsProvider:credentialsProvider];
AWSServiceManager.defaultServiceManager.defaultServiceConfiguration = configuration;

```

Get the identity id for this provider. If an identity id is already set on this provider, no remote call is made and the identity will be returned as a result of the AWSTask (the identityId is also available as a property). If no identityId is set on this provider, one will be retrieved from the service.

```objc
// Retrieve your Amazon Cognito ID
[[credentialsProvider getIdentityId] continueWithBlock:^id(AWSTask *task) {
    if (task.error) {
        NSLog(@"Error: %@", task.error);
    }
    else {
        // the task result will contain the identity id
        NSString *cognitoId = task.result;
    }
    return nil;
}];
```

Then grab the `defaultClient` from your code

```objc
CLIMOCKClient *client = [CLIMOCKClient defaultClient];
```

You can now call your method using the client SDK

```objc
[[client demoGet] continueWithBlock:^id(AWSTask *task) {
    if (task.error) {
        NSLog(@"Error: %@", task.error);
        return nil;
    }
    if (task.result) {
       CLIEmpty * output = task.result;
       //Do something with output
    }
    return nil;
}];
```

> To use an API key with the API Gateway-generated SDK, you can set the `apiKey` property of the generated SDK to send API Keys in your requests.  If you use an API key, it is specified as part of the x-api-key header and all requests to the API will be signed.
>
> client.APIKey = @"api-key";

# References

* [Set Up the SDK for iOS](http://docs.aws.amazon.com/mobile/sdkforios/developerguide/setup.html)
* [Getting Credentials](http://docs.aws.amazon.com/cognito/latest/developerguide/getting-credentials.html)
* [AWS Mobile SDK for iOS v2.4.2 Reference](http://docs.aws.amazon.com/AWSiOSSDK/latest/index.html)