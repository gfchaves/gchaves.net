---
title: Azure API Management - How to mock ?
author: Chaves
date: 2020-08-29 15:00:00 +0000
categories: [Blogging, Azure, .NET]
tags: [.net, azure, apim, mock, stub]
---

We had the challenge to configure some mock services at on a APIm instance, and we found out that a simple mock policy wasn't enough, so here's how we solve it.

## Intro
So we had a request to develop and configure a mock service on Azure APIm for a given set of APIs that are currently under dev/test. Such a strong platform as [Azure API Management](https://azure.microsoft.com/en-us/services/api-management/) that has a lot of features and capabilities out of the box should have a handy solution for our problem and it has! Although we needed to go further and we encounter a missing/incomplete feature that we have to take an alternative and even share it within Microsoft's Azure support team.

## The mock-response policy
So given our challenge, we went for the mock policy feature and even follow the [tutorial](https://docs.microsoft.com/en-us/azure/api-management/mock-api-responses) that explains ou the mock feature works. Everything on under the hood of APIm's is based on a set of [policies](https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies) that configures and setups the behavior and the interaction between frontend, inbound, backend, and outbound interfaces/communication flow. 
Primarily we started with a single HTTP 200 response with a JSON payload that represents precisely what we needed to offer for the consumer - which is great because it set you free of the consumer when you still need to perform development.
Great! So we started to define what we needed to response on our API already exposed at APIm catalog, by the setup of the response on the frontend endpoint editor - and this is the editor for the openAPI specification file (see below) and after we have our mock policy that configures the inbound traffic to be responded by a mock with a 200 response and a content type of JSON:

```xml
<policies>
    <inbound>
        <base />
        <mock-response status-code="200" content-type="application/json" />       
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

the openApi spec definition:

```yaml
{
    "openapi": "3.0.1",
    "info": {
        "title": "http-bin-sample",
        "description": "",
        "version": "1.0"
    },
    "servers": [{
        "url": "https://apim-codit-nexus.azure-api.net/httpbin"
    }],
    "paths": {
        "/racer": {
            "get": {
                "summary": "Get TopRacer",
                "description": "get top racer",
                "operationId": "get-topracer",
                "responses": {
                    "200": {
                        "description": "Another test with mock service",
                        "headers": {
                            "MyCustomHeader": {
                                "description": "for testing headers",
                                "required": true,
                                "schema": {
                                    "enum": ["\"teste\""],
                                    "type": "string"
                                }
                            },
                            "AnotherCustomHeader": {
                                "description": "another test with int",
                                "required": true,
                                "schema": {
                                    "enum": ["2"],
                                    "type": "int"
                                }
                            }
                        },
                        "content": {
                            "application/json": {
                                "schema": {
                                    "$ref": "#/components/schemas/Racer"
                                },
                                "example": {
                                    "Name": "Miguel Oliveira",
                                    "Number": "88",
                                    "Class": "motogp"
                                }
                            }
                        }
                    }
                }
            }
        },

        ... other endpoints ...
```

The HTTP request:

```curl
curl --location --request GET 'https://apim-codit-nexus.azure-api.net/httpbin/racer' -v 
```

The HTTP response:

> HTTP/1.1 200 OK
> Content-Length: 58
> Content-Type: application/json
> Date: Sat, 29 Aug 2020 15:21:34 GMT
>
> Connection #0 to host apim-codit-nexus.azure-api.net left intact

```json
{
    "Name":"Miguel Oliveira", 
    "Number":"88",
    "Class":"motogp"
}
```

## The limitation

Look on the HTTP headers.... yes we're missing the previously additional configured HTTP headers:

1. `MyCustomHeader with value of: "teste" (string)`
2. `AnotherCustomHeader with value of "2" (int)`

Apparently and after much digging we found out that this a current limitation of the APIm GUI that don't allow custom headers with the mock policy - so what if the GUI Headers is used for? It's for the definition of the openApi spec file.

![az-apim-mockheaders](/assets/img/posts/az-apim-mock-headers.png)

As you see from the above figure, the GUI let you define the response body and the response headers, but only the body is used at the specification and mock level, headers are only on the spec level... 😒

We're able to confirm with the Azure's support staff here's their statement:

>“I understand that you want to configure your custom Headers in the Mock Response Policy. I would see how best I could be of support to you. 
From my investigation, I understand that the Mock Response Policy doesn’t make use of the response headers specified in the API form-based editor. The form-based editor allows you to design the schema of an expected response, although, by design, the Mock Response Policy makes use of the body representation only and not the response headers. The Return Response policy gives you full control over the content of the response, that is both header and body, and would suit your desired use case scenario.
 
>Thank you also for sharing your idea on the azure forum, this is very important to us. Our product team reviews this forum regularly to get guidance for future enhancement and could consider your idea.”

Ok, thanks for the guidance! Let's roll with the [ReturnResponse Policy](https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies#ReturnResponse) 😊👍

## Our approach

The ReturnResponse Policy lands the response controller to your hand 😎 - This means that with this policy we can have total control over the HTTP response, hence we went with:

```xml
    <return-response>
        <set-status code="200" />
        <set-header name="content-type" exists-action="override">
            <value>application/json</value>
        </set-header>
        <set-header name="MyCustomHeader" exists-action="override">
            <value>teste</value>
        </set-header>
        <set-header name="AnotherCustomHeader" exists-action="override">
            <value>2</value>
        </set-header>
        <set-body>{"Name":"Miguel Oliveira", "Number":"88", "Class":"motogp"}</set-body>
    <return-response>
```

Has you can see with the `<return-response>` policy we can specify our custom HTTP headers and naturally the body message.

Also, you can go further by using the `response-variable-name` field to output what you've stored on a previous request for example (see documentation for further info). The return policy gives you way more control and that said and considering the feedback that we had from the support team I would say that the mocking policy is only for a very tiny POC or for most simple cases that you just want to have something that gives an HTTP status code.

## Conclusion
Just to point it out that we weren't the first to hit with our heads on this, we also found a post on Azure forum to ask that product team can perform the necessary updates on the APIm to use the GUI custom headers on the mock response. You can always vote at [https://feedback.azure.com/forums/248703-api-management/suggestions/38199637-allow-mock-response-headers](https://feedback.azure.com/forums/248703-api-management/suggestions/38199637-allow-mock-response-headers) to call for develop team attention of this missing feature.

