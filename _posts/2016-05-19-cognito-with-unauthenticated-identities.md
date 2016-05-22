---
layout: post
title: "Cognito with Unauthenticated Identities in Javascript"
image: '/assets/img/'
description: "In this post, I will demo you how to use Cognito Identity Pool to authorize unauthenticated clients to invoke API Gateway in Javascript"
main-class: aws
tags:
- "identify pool"
- "api gateway"
- "cognito"
- "javascript"
categories: aws
twitter_text: "Cognito with Unauthenticated Identities in Javascript"
introduction: "Use Cognito Identity Pool to authorize unauthenticated clients to invoke API Gateway in Javascript"
---

# Pain Point

I intent to create a REST API to handle request from unauthenticated mobile app(s), but the API should not be invoked by other unrecognized end points.

For access control, I could not rely on providing client applications with static API key strings; these can be extracted from clients and used elsewhere.

The Amazon API Gateway allows to generate client SDKs to integrate with your APIs. That SDK also manages the signing of requests when APIs require authentication.

Each resource/method combination that you create as part of your API is granted its own specific Amazon Resource Name (ARN) that can be referenced in AWS Identity and Access Management (IAM) policies.

API access is enforced by the IAM policies that you create outside the context of your application code. This means that you do not have to write any code to be aware of or enforce those access levels.

Authorizing clients using [AWS Signature version 4 (SigV4)](http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) authorization and IAM policies for API access allows those same credentials to restrict or permit access to other AWS services and resources as needed.

However there is no comprehensive sample code to walk through the idea as I document the post, I hope this could be of some help to whom would like to have similar intention as I do.

# Overview

In this sample code, I'd like to create an **MOCK** service which would only response to client (mobile app) with signing requests. 

* Create MOCK API Gateway and Enable CORS
* Change Authorization Settings to AWS_IAM
* Create Cognito Identity Pool
* Grant Cognito_StoreUnauth_Role to invoke MOCK API Gateway
* Invoke MOCK API Gateway with Cognito SDK in JS

# Steps by Steps

### Create Mock API Gateway and Enable CORS

* Login `AWS Management Console`
* Open `Amazon API Gateway` and click on `Create API`
* Name API with `MOCK`
* Click `Actions` > `Create Resource`
* Name resource with `demo`
* Under `\demo`, click on `Actions` > `Create Method`
* Select `GET` with `Mock Integration` type and click `Save`
* Under `\demo`, click on `Actions` > `Enable CORS`
* Click `Enable CORS and replace existing CORS headers`
* Click `Yes, replace existing values`
* Click `Actions` > `Deploy API` and `[New Stage]` if this is firsty time deployment, otherwise select `dev`
* Name stage with `dev` and click `Deploy`
* You shoul be able to invoke the API successfully by browser

    GET https://[api-id].execute-api.[region].amazonaws.com/dev/demo
    
### Change Authorization Settings to AWS_IAM

* Login `AWS Management Console` and open `Amazon API Gateway`
* Under `APIs\MOCK\Resources`, click on `GET`
* Click `Method Request` and set `Authorization` to `AWS_IAM`
* Click `Actions` > `Deploy API`
* Set `Deployment stage` to `dev` and click `Deploy`
* You shoul receive error response after invoking the same API again

```json
{
  "message": "Missing Authentication Token"
}
```

### Create Cognito Identity Pool

* Login `AWS Management Console`
* Open `Cognito` and click on `Manage Federated Identities`
* Click `Create new identity pool`
* Name identity pool with `MOCK`
* Check `Enable access to unauthenticated identities`
* Click `Create pool` and `Allow`
* Open `Federated Identities` and click on `MOCK` which is just created
* Click `Edit identity pool`
* Keep `Identity pool ID` in mind as this value will be used in later code snippet

### Grant Cognito_StoreUnauth_Role to invoke MOCK API Gateway

* Login `AWS Management Console`
* Open `IAM` and click on `Roles`
* Select `Cognito_MOCKUnauth_Role` which is created automatically as you created identity pool in previous step
* Click `Edit Policy` and paste with the following content

> Replace [region], [account-id] and [api-id] first

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "execute-api:Invoke"
            ],
            "Resource": [
                "arn:aws:execute-api:[region]:[account-id]:[api-id]/dev/*/*"
            ]
        }
    ]
}
```

### Invoke MOCK API Gateway with Cognito SDK in JS

* Login `AWS Management Console` and open `Amazon API Gateway`
* Under `APIs\MOCK\Stages`, click on `dev`
* Select tab `SDK Generation`
* Set `Platform` as `Javascript`
* Click `Generate SDK` and download an archived file
* Decompress the download file to a folder, say `~\foo` 
* Download [aws-sdk](https://github.com/aws/aws-sdk-js/releases) from Github
* Decompress the download file to the previous folder
* Create a static index.html with the following content in the previous folder

```html
<!DOCTYPE html>
<html>
<title>Web Page Design</title>
<head>
    <script type="text/javascript" src="lib/axios/dist/axios.standalone.js"></script>
    <script type="text/javascript" src="lib/CryptoJS/rollups/hmac-sha256.js"></script>
    <script type="text/javascript" src="lib/CryptoJS/rollups/sha256.js"></script>
    <script type="text/javascript" src="lib/CryptoJS/components/hmac.js"></script>
    <script type="text/javascript" src="lib/CryptoJS/components/enc-base64.js"></script>
    <script type="text/javascript" src="lib/url-template/url-template.js"></script>
    <script type="text/javascript" src="lib/apiGatewayCore/sigV4Client.js"></script>
    <script type="text/javascript" src="lib/apiGatewayCore/apiGatewayClient.js"></script>
    <script type="text/javascript" src="lib/apiGatewayCore/simpleHttpClient.js"></script>
    <script type="text/javascript" src="lib/apiGatewayCore/utils.js"></script>
    <script type="text/javascript" src="dist/aws-sdk.min.js"></script>
    <script type="text/javascript" src="apigClient.js"></script>
    <script src="script.js"></script>
</head>
<body>
</body>
</html>
```

* Create a static script.js with the following content in the previous folder

> Replace [region] and [identity-pool-id] first 
    
```javascript
function foo() {

  AWS.config.region = 'region'; // Replace with the region you deploy

  var cognitoidentity = new AWS.CognitoIdentity();
  var params = {
    IdentityPoolId: 'identity-pool-id'
  };

  // Generate a Cognito ID for the 1st time, so IdentityId could be kept for future use
  cognitoidentity.getId(params, function(err, data) {
    if (err) console.log(err, err.stack); // an error occurred
    else console.log(data); // successful response

    var params = {
      IdentityId: data.IdentityId
    };

    // Retrieve temp credentials with IdentityId
    cognitoidentity.getCredentialsForIdentity(params, function(err, data) {
      if (err) console.log(err, err.stack); // an error occurred
      else console.log(data); // successful response

      var apigClient = apigClientFactory.newClient({
        accessKey: data.Credentials.AccessKeyId,
        secretKey: data.Credentials.SecretKey,
        sessionToken: data.Credentials.SessionToken,
        region: 'region' // Replace with the region you deploy
      });

      var params = {
        //This is where any header, path, or querystring request params go. The key is the parameter named as defined in the API
        foo: "foo"
      };

      apigClient.demoGet(params).then(function(result) {
        //This is where you would put a success callback
        console.log(result);
        alert("Hello foo!");
      }).catch(function(result) {
        //This is where you would put an error callback
        console.log(result);
        alert("Oops foo!");
      });
    })

  });
}

foo();
```

> You ONLY need to invoke `AWS.CognitoIdentity.getId()` first time or as you receive `ResourceNotFoundException: Identity 'identity-id' not found.`
> 

* Open index.html in browser, and you should be able to invoke the API successfully and read a prompt message `Hello foo!`

> To use an API key with the API Gateway-generated SDK, you can pass the API key as a parameter to the Factory object by using code similar to the following. If you use an API key, it is specified as part of the x-api-key header and all requests to the API will be signed.
>
> var apigClient = apigClientFactory.newClient({
  apiKey: 'api-key'
});


# References

* [AWS Serverless Multi-Tier Architectures](https://d0.awsstatic.com/whitepapers/AWS_Serverless_Multi-Tier_Architectures.pdf)
* [Enable CORS for a Resource through a Method in API Gateway](http://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)
* [Identify Pool](http://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html)
* [Generate an SDK for an API in API Gateway](http://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-generate-sdk.html)

