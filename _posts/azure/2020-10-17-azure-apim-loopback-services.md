---
title: How to Configure Loopback Services at Azure API Management
author: Gonçalo Chaves and João Dinis
date: 2020-12-05 14:10:00 +0800
categories: [Blogging, Azure, .NET]
tags: [azure, apim, loopback, webapi, webservices]
---

*This post was originally published at [Codit's blog](https://www.codit.eu/blog/configure-loopback-services-azure-api-management/)*

## Intro

Not so long ago, we were challenged to build a framework that lies on a specific platform architecture that we hosted on Azure resources. Although we had to take into account the particular design wishes of our customer, we worked together with them to come up with a design and process that fit the bill. The end design allows resources to be set up to work together on a set of features and services and provide a full enterprise service bus on the cloud – which is impressive! 💪

Nowadays talking about services is the same as talking about of the busiest airport in the word – before this pandemic time. Wherein only a set of procedures and structured frameworks could be put in place to operate procedures safely, even when everything looks like a huge traffic jam.

So services (like airplanes) have common concepts (like wings and engines) but they differ in their specific ways in respect to technology, business requirements, developers capacities, or what was the in-place procedure at that time. You can liken this to two major aircraft manufacturers producing different sizes, styles, and types of parts, but the same principles apply – like our enterprise services.

At this stage, and to keep it simple, our services include every service type and architecture (micro-services, SOAP, REST, SOAP-ish, REST-ish – you got the idea 😁). This means that like an airport, we need a hub. Our  hub is Azure API Management (aka APIm). APIm is our services manager that allows us to have great capabilities to control and manage our service’s behaviors, uniformization, rate limits, and access control.

But like any other airport/service manager we faced some challenges because of the way we want to manage our architecture inside of it. In this post, we’ll present parts of the current architecture that led us to the challenge of having multiple services hosted within it, but also needed calls between them in the form of loopback calls for the same APIm instance.

## The challenge

We started to have a single APIm instance that could host our services layers: 

- **Channels APIs** - the public and external façade of internal services to have a single uniformization and a set of security rules, authentication procedures, and rate limits to be in place when exposed wide open to the internet;
- **Services APIs** - first internal layer that represents the service implementation as an entity itself, but using other services layers to abstract the backend implementation-specific logic;  
- **Adapters APIs** - the nearest service layer implementation that holds the specific logic of the backend services. This acts as the bridge between the provider service implementation and the service common architecture layer – by holding the specific implementation. We can have multiple adapters that serve the same service and act as data connectors;

These layers were composed by a set of names and configuration rules (APIm products and policies) on a single Azure API Management.  

![diagram](https://www.codit.eu/wp-content/uploads/2020/10/diagram-01-768x441.png)

Calls orchestration are mainly guided by:  

Public => ChannelAPI => Service => Adapter => Backend provider

So at least we have three calls that can be treated in a straight forward manner, or have high manipulation between them to fulfill specific requirements, or have necessary adjustments between each layer until the request reaches the backend. These types of adapters allow the main service layer to gather data for multiple sources and provide single responses with data unified on it.


## Configuration barrier

We started to deploy the layers out-of-the-box on Azure APIm single instance (with developer tier) and we used the APIm hostname to perform the service https calls between them.

[APIm advanced policies](https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies) allow us to configure and perform some useful operations as we needed it, given the following scenario:

- Url **service A**: https://apim-demo-codit.developer.azure-api.net/services/service-a/operation-flights
- To call **adapter B**: https://apim-demo-codit.developer.azure-api.net/adapters/adapter-b/operation-flights

For the sake of simplicity, every mechanism of authentication, such as API subscriptions keys, was disabled just for demo proposes and troubleshooting. Using APIm policies, on service A, we were able to set up:

- **Rewrite-uri** policy, at first sight, makes sense to just change the URL and perform the request into another service/endpoint – our case to adapter-b.

```xml
<policies>
    <inbound>
        <base />    
        <rewrite-uri template="adapters/adapter-b/operation" copy-unmatched-params="true" />
    </inbound>
    ...
</policies>
```

and

- **Send request** policy, in our opinion, a much more elegant and understandable regarding the concept of having service A that’s sending a new https request to the service B (our adapter) endpoint.

```xml
<policies>
    <inbound>
        <base />    
            <send-request mode="new" response-variable-name="result" timeout="300" ignore-error="false">
                <set-url>https://apim-demo-codit.developer.azure-api.net/adapters/adapter-b/operation</set-url>
                <set-method>GET</set-method>
                <set-header name="content-type" exists-action="override">
                    <value>application/json</value>
                </set-header>                                
        </send-request>
        <return-response response-variable-name="result" />
    </inbound>
    ...
</policies>
```
Everything was working great, and as expected a major platform as Azure API Management could handle a full catalog set of services and adapters (in our architecture design) and interactions between them.

So when we moved our solution into the client’s Azure APIm instance, we were surprised to see we were unable to have **HTTP/HTTPs calls between hosted services.** We couldn’t get loopback calls. Instead we got 500 – Internal Server error.

Our client’s APIm instance is configured inside of a private virtual network exposed to the internet using the [Application Gateway (WAF)](https://docs.microsoft.com/en-us/azure/application-gateway/overview) and with the FQDN service usage, we weren’t able to get 200 Ok responses on our “working” solution. By performing [APIm traces](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-api-inspector), we saw that the APIm was resolving the domain into its private IP, forwarding the request (as expected) but refused a connection to itself.

```json
...
{
    "source": "send-request",
    "timestamp": "2020-10-...",
    "elapsed": "00:00:00.0006939",
    "data": {
        "message": "Request is being forwarded to the backend service. Timeout set to 300 seconds",
        "request": {
        "method": "GET",
        "url": "https://172.xx.xx.x/echo/test",
        "headers": [
            {
            "name": "Host",
            "value": "172.xx.xx.x"
            },
            {
            "name": "content-type",
            "value": "application/json"
            },
            {
            "name": "Ocp-Apim-Subscription-Key",
            "value": "e78d2de532c447b1bf0958d6d738bd8a"
            },
            {
            "name": "X-Forwarded-For",
            "value": "10.xx.x.xx"
            }
        ]
        }
    }
    },
    {
    "source": "send-request",
    "timestamp": "2020-10-...",
    "elapsed": "00:00:21.0091023",
    "data": {
        "messages": [
        "Unable to connect to the remote server",
        "Error occurred while calling backend service.",
        "A connection attempt failed because the connected party did not properly respond after a period of time,
            or established connection failed because connected host has failed to respond 172.xx.xx.x:443"
        ]
    }
}

```

We got stuck! We started a trial and error debugging to figure out what was going wrong! Why wasn’t the APIm taking our loopback calls as we had in our subscription?

We brainstormed ideas and started to change the address to troubleshoot the problem , such as:

- **https://localhost/...** - 404 Not found | Doesn't makes sense... every server know the localhost special term.. right? No good... 

- **https://127.0.0.1/...** - 404 Not found | like the localhost... the loopback address should work... this is getting impossible to figured out what is happening - even using the Apim-Trace output.

- **APIm's private ip address** - 500 Internal Server Error | Probably to many network wiring complications that don't allow internal referenced requests. Trace ouput was the same of above, resolved into the private but not allowing connection into it;

- **Application gateway public (that's configured to forward external traffic into APIm) ip address** - 404 Not found | But on the response body we can see that it was the [azure application gateway](https://docs.microsoft.com/en-us/azure/application-gateway/overview) who responded... again odd, because it's the nears solution regarding our own environment and the WAF should let the request goes out and came back in... no good;

- **APIm's public ip address (the one that's given by Azure)** - we've got two different responses: 
    - **Inside the client's network:** 500 Internal Server Error | Trace output was the same of above, resolved into the ip address but not allowing connection into it;
    - **Public network:** 504 Gateway Timeout | This is the Azure Application Gateway response time out guessing that forward the request and never got any response from the internal load balancer;

With no more ideas or suggestions to try, we felt that we might reach a bug or even a physical APIm limitation, with no other way to resolve the problem unless we ask for official help.

### The solution

We got stuck, and with no ideas to tr or debug, we couldn’t move forward. It was time for Azure’s support to come to the rescue. Once again, Azure support proved to be effective and reliable. After two hours of interaction (for them to understand what was going on under the hood) they came up with the “solution” – yes! 

To allow loopback calls, Azure support requested that we add a new header on the HTTP request that includes the name of the **Host** of the APIm domain and use the **127.0.0.1** at the url address such as:

```xml
    <set-header name="Host" exists-action="override">
        <value>apim-demo-codit.developer.azure-api.net</value>
    </set-header>
```

Eureka 😎! So we’ve changed our APIm’s policy to include an additional header with **Host domain**, and APIm started to accept loopback calls:

```xml
<policies>
    <inbound>
        <base />    
            <send-request mode="new" response-variable-name="result" timeout="300" ignore-error="false">
                <set-url>https://127.0.0.1/adapters/adapter-b/operation</set-url>
                <set-method>GET</set-method>
                <set-header name="content-type" exists-action="override">
                    <value>application/json</value>
                </set-header>
                <set-header name="Host" exists-action="override">
                    <value>apim-demo-codit.developer.azure-api.net</value>
                </set-header>                                
            </send-request>
        <return-response response-variable-name="result" />
    </inbound>
    ...
</policies>
```

Azure’s support team explained to us that a *recent change* on APIm’s core (from January release) was affecting these loopback type calls, and at the current configuration (hosted as a private resource inside the subscription) it requires a Host header to allow the internal load balance redirect to the right APIm instance that holds and knows the service that we wanted to call. Also to be careful regarding the casing of the headers’ names – although we tried both ways, this doesn’t seem to be the case. 

Afterwards, we also found out that even with this new **Host header**, we can also use the localhost instead, on the set-url field – meaning that this new Host header is what makes the real deal.

Also, we can use another policy type like the rewrite, but again adding the Host as a header such as:
```xml
<policies>
    <inbound>
        <base />
        <set-header name="content-type" exists-action="override">
            <value>application/json</value>
        </set-header>
        <set-header name="Host" exists-action="override">
            <value>apim-demo-codit.developer.azure-api.net</value>
        </set-header>
        <rewrite-uri template="/echo/test" />
    </inbound>
...
```

We didn’t find any reference in the official documentation regarding this specific header configuration, but we’ll suggest it. 

Keep in mind that if your loopback calls need authentication, you need to forward those specific headers to the service you’re reaching out to. Otherwise, APIm will cross those out and you’ll get a 403 by the authentication fail response. In our scenario, at the policy, we added additional code to grab the key and forward it on the “new” request that we’re performing at this stage. Azure’s support also confirmed to us that we had to do it manually because at the moment there isn’t an automated policy or key work.

So in the end our policy looked like:

```xml
<policies>
    <inbound>
        <base />
        <set-variable name="subscriptionKey" value="@(context.Request.Headers.GetValueOrDefault("Ocp-Apim-Subscription-Key","scheme param"))" />
        <send-request mode="new" response-variable-name="result" timeout="300" ignore-error="false">
            <set-url>https://localhost/echo/test</set-url>
            <set-method>GET</set-method>
            <set-header name="content-type" exists-action="override">
                <value>application/json</value>
            </set-header>
            <set-header name="HOST" exists-action="override">
                <value>apim-demo-codit.developer.azure-api.net</value>
            </set-header>
            <set-header name="Ocp-Apim-Subscription-Key" exists-action="override">
                <value>@($"{(string)context.Variables["subscriptionKey"]}")</value>
            </set-header>
        </send-request>
        <return-response response-variable-name="result" />
    </inbound>
...
```

## Conclusions and acknowledgements

APIm is a remarkable service to handle APIs and to have a uniformization across your service catalog, but like all solutions, we find some challenges when we’re pushing our architecture towards new goals and specific scenarios. Besides this little loopback issue that stumbled on in our client’s scenario, we would like to give some tips that we always keep in mind:

- Don’t only perform your tests using the Azure’s APIm portal interface, sometimes we got different results from the portal when we call outside (example from our postman collections);
- Make your tests inside and outside the APIm’s network, if you have yours deployed within a private network, be sure to test both scenarios: private and public – we also found out unexpected results;
- Keep in mind the internal APIm’s cache, at each configuration change, remember that it takes moments to load up the new configuration (support said it can be up to one minute.)
- Try to keep your policies with low complexity as long as you can;
- Ensure to use the trace [header feature](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-api-inspector) to see what happened with your request;
- Debug Azure API Management policies in Visual Studio Code – [see how](https://docs.microsoft.com/en-us/azure/api-management/api-management-debug-policies)
- Check if the domain is working properly (not expired, has DNS records, etc) before you think that you’re the problem, or if you don’t have a typo on it;
- Avoid the “rewrite” at your policy and use the “send request” at least in this type of scenario;
- Check your network security groups configurations, be aware of traffic rules that can be applied;
- When you receive a 400 bad request (testing from the portal) check for the CORS bypass option to obtain more information;

Last but not least, we would like to send a big thank you to all our Codit colleagues that helped us during this challenge, and of course also to Azure’s support for their contribution. 