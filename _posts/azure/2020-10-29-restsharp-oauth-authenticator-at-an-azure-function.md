---
title: OAuth1 authentication at your Azure Functions
author: Chaves
date: 2020-10-29 00:00:00 +0000
categories: [Blogging, Azure]
tags: [.net, azure, git, powershell]
---

## Handle with OAuth1 Spec

We've been developing some [Azure functions](https://azure.microsoft.com/en-us/services/functions/) to create integrations services that have some specific requirements. The good is that Azure function allows us to work with .net framework potential and we can put our mainstream *"jedi magic"* development knowledge into action.

In this particular case, we're creating some specific HTTP connectors that must handle OAuth1 authentication on each request that we need to perform, no big deal right?

One HTTP request with [OAuth1](https://tools.ietf.org/html/rfc5849) authentication header should look like this:

```yaml
curl --location --request POST 'https://yourProviderWithOauth1Url/?someParam=true' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: OAuth oauth_consumer_key="Your_Consumer_Key",oauth_signature_method="HMAC-SHA1",oauth_timestamp="1603988723",oauth_nonce="_calculated_nonce_",oauth_version="1.0",oauth_signature="_calculated_signature_"' \
--header 'Cookie: ONESESSIONID=data' \
--data-raw '{JSON Body}'
```

We could go rogue and perform the necessary development to create the Authorization header param with the requested OAuth1 fields:

- oauth_consumer_key="...."
- oauth_nonce="..."
- oauth_signature="..."
- oauth_signature_method="HMAC-SHA1"
- oauth_timestamp="..."
- oauth_version="1.0"

And for each this property we can compute the necessary steps to generate the right values on runtime... but nowadays shouldn't be necessary - let get some fuel into it.

## RestSharp into the stage

But putting the [RestSharp](https://restsharp.dev/) lib into action we can reduce this into a few lines of code such as:

{% gist ca30f709a61cc82c9075d7bd651b2ed2 %}

So as you can see everything starts with the client declaration, where you put your specific configuration in place (authentication, properties, host, etc..), and then we just configured the OAuth authentication properties to request a token that uses as default the `OAuthSignatureMethod.HmacSha1` which was we're looking for. The hard part is done, after is just a matter of headers configuration (in case you need to add your specific headers on the HTTP request) and the body params as well.

Execute the client's request and look for what the result has on it. - voilá 😎

RestSharp as they say *"probably, the most popular REST API client library for .NET"* and for my personal experience if it is not it probably is 😁. I never liked having too much backend code performing HTTP calls to other services, especially if we're at a `"RESTfull"` scenario where the consumer apps should orchestrate the call their own, but of course, there are many exceptions and at what we have here is one of them. We need to perform multiple calls from a backend function (server to server) and this library can help us in the effort to write every aspect and HTTP's specification necessary instead of going down the road with the traditional `System.Net.Http.HttpClient` class one.

[RestSharp](https://restsharp.dev/) offers capabilities of serialization, sync and async, authentication (basic, OAuth1, OAuth2, JWT, NTLM, and custom), parameters, forms, files, and extensive configuration that can help you to build your own specific C# HTTP client that is built around your custom integration scenario. 

I've been using this particular library for almost four years and is proven to be strong and reliable. So turn your C# code more readable and maintainable when dealing with REST APIs with this little sharp on it.