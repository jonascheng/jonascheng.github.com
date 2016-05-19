---
layout: post
title: "Cognito with Unauthenticated Identities"
image: '/assets/img/'
description:
main-class: aws
tags:
- "identify pool"
- "api gateway"
- "lambda"
- "serverless"
categories: aws
twitter_text: "Cognito with Unauthenticated Identities"
introduction: "Cognito with Unauthenticated Identities"
---

# Background

I intent to create a REST API to handle request from unauthenticated mobile app(s), but the API should not be involked by other unrecognized end points.

For access control, I could not rely on providing client applications with static API key strings; these can be extracted from clients and used elsewhere.The Amazon API Gateway allows to generate client SDKs to integrate with your APIs. That SDK also manages the signing of requests when APIs require authentication.
Each resource/method combination that you create as part of your API is granted its own specific Amazon Resource Name (ARN) that can be referenced in AWS Identity and Access Management (IAM) policies.API access is enforced by the IAM policies that you create outside the context of your application code. This means that you do not have to write any code to be aware of or enforce those access levels.Authorizing clients using [AWS Signature version 4 (SigV4)](http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) authorization and IAM policies for API access allows those same credentials to restrict or permit access to other AWS services and resources as needed.However there is no comprehensive sample to walk through the concept as I document the post, I hope this could be of some help to whom would like to have same intention.

# Summary

In this sample, I'd like to create an **MOCK** service which would only response client (mobile app) with signing of requests. 

* Create Mock API Gateway and Enable CORS
* Change Authorization Settings to AWS_IAM


# Steps by Steps

## Create Mock API Gateway and Enable CORS

* Login `AWS Management Console`
* Open `Amazon API Gateway` and click on `Create API`
* Name API with `MOCK`
* Click on `Actions` > `Create Resource`
* Name resource with `demo`
* Under `\demo`, click on `Actions` > `Create Method`
* Select `GET` with `Mock Integration` type and click `Save`
* Under `\demo`, click on `Actions` > `Enable CORS`
* Click `Enable CORS and replace existing CORS headers`
* Click `Yes, replace existing values`
* Click on `Actions` > `Deploy API` and `[New Stage]` if this is firsty time deployment, otherwise select `dev`
* Name stage with `dev` and click `Deploy`
* You shoul be able to invoke the API successfully by [PostMan](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop) or browser

    GET https://[API-ID].execute-api.[Region].amazonaws.com/dev/demo
    
## Change Authorization Settings to AWS_IAM

* Under `APIs\MOCK\Resources`, click on `GET`
* Click `Method Request` and set `Authorization` to `AWS_IAM`
* Click on `Actions` > `Deploy API`
* Set `Deployment stage` to `dev` and click `Deploy`
* You shoul receive error response after invoking the same API again

```json
{
  "message": "Missing Authentication Token"
}
```

# References

* [AWS Serverless Multi-Tier Architectures](https://d0.awsstatic.com/whitepapers/AWS_Serverless_Multi-Tier_Architectures.pdf)
* [Identify Pool](http://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html)
* 

