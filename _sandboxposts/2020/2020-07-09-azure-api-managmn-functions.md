---
title: Azure API Management => Functions and logic apps auth with managed identities
author: Chaves
date: 2020-05-29 22:00:00 +0000
categories: [Blogging, Azure, .NET]
tags: [.net, charp, msbuild, sonar, better code, mvc, code coverage, code quality, best pratices, visual studio, bat file, tfs]
---
![az-apim](/assets/img/posts/az-apim1.png)

Here some links of support that gave me real help on the way:

https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview => What is managed identities for Azure resources, and this is important to understand what is happening under the hood with this authentication between your different resources; "Managed identities for Azure resources is the new name for the service formerly known as Managed Service Identity (MSI)." => 😁 small name detail;
https://kalcik.net/2020/04/15/use-managed-identity-in-azure-api-management-to-authenticate-with-an-azure-function/ => first step, where you need to power up the managed identity on APIm himself, otherwise at your backend API you won't receive the correct type of authentication;
https://adatum.no/azure/azure-ad-authentication-in-azure-functions => at your backend APIs you need to power up the right managed identity authentication. In this case, it's a function and the post explains how we've done it;
https://docs.microsoft.com/en-us/azure/api-management/api-management-authentication-policies => The policies that are supported by Azure APIm: basic, certificate, and managed identity. How it should be configured;


This allows that after the authentication between APIm and backend API occurs using the managed authentication, add the result JWT token into the HTTP header that will be sent to your backend API endpoint. => which allows us to transport the authentication from MSI to other Azure resources.

https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT => in this article it needs to point out the following:
"The validate-jwt policy requires that the exp registered claim is included in the JWT token, unless require-expiration-time attribute is specified and set to false. The validate-jwt policy supports HS256 and RS256 signing algorithms. For HS256 the key must be provided inline within the policy in the base64 encoded form. For RS256 the key has to be provide via an Open ID configuration endpoint. The validate-jwt policy supports tokens encrypted with symmetric keys using the following encryption algorithms A128CBC-HS256, A192CBC-HS384, A256CBC-HS512." - otherwise, your JWT token will be always marked as invalid.
https://jwt.io/ => a great online tool to help you out decode/encode JWT tokens only for debug proposes;
 

Using the same flow with logic app
 

Links of support:

https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-securing-a-logic-app 
https://www.c-sharpcorner.com/article/demystifying-sas-token-basics/ - SAS scheme and fields lists, a handout to demystify the token composition