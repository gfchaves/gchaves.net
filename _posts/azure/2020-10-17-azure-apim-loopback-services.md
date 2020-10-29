---
title: Azure APIm - How to configure a loopback services
author: Gonçalo Chaves
date: 2020-10-17 14:10:00 +0800
categories: [Blogging, Tutorial]
tags: [azure, apim, loopback, webapi, webservices]
---

## Intro

We were challenged to build a framework that lies on a specific platform architecture that we hosted by azure resources. Although the complexity of the given design principles for our customer, together both teams came up up a design and process that allow resources be setup to work together on a set of features and services to provide a full enterprise service bus on the cloud - in which it's impressive that his way.

Nowadays talking about services is the same of talking about of the most busiest airport in the word - before this pandemic time (smile) - where only a set of procedures and strucuterd frameworks at in place to be possible operate in real safe procedures that could be safe even when everything looks messy like ordinary traffic jam. 

So services (like airplanes) have common concepts (like wings and engines) but hey differ in their specific ways regarding technology, business requirements, developers capacities or what was the in place procedure at that time... like the two major aircraft manufactures produce different sizes, styles and types but all together arround same principles - like our entrerprise services. 😊 (meme image of "services every where"). At his stage and to keep it more simple our services includes every service type/architecture (micro-services, soap, rest, soap-ish, rest-ish - you got the idea 😁). This means that "the airport" is our hub like the Azure API Managmenent (aka APIm) is our services manager that allows us to have such great capabilities to control and manage our services behavirous, uniformization, rate limits, and access control. 


But like any other airport/service manager we faced some challenges because of the way we want to manage our architecture inside of it. At this post we'll present parts of the current architecture that leads us to enconter the challenge of having multiple services hosted with in it but with the necessity of having caals between then that came up in loopback calls for the same APIm instance.

## The challenge

We started to have a single APIm instance that could host our services layers: 

- **Channels APIs** - public and external facade of internal services to have a single uniformization and a set of security rules, authentication procedures and rate limits to be in place when exposed wide open to the internet;
- **Services APIs** - first internal layer that represents the service implementation as a entity itself but using another services layers to abstract the backend implementation-specific logics;
- **Adapters APIs** - the nearest service layer implementation that hold the specific logic of the backend services. This acts as the bridge between the provider service implementation and the service common architecture layer - by holding the spefific implementation. We can have multiple adapters that services the same service and act as data connectors.

These layers were disposed with a set of names and configuration rules (APIm products and policies) on a single Azure API Management. 

(diagram like massimo suggest)

Calls orchestration are maily guided by: 

Public => ChannelAPI => Service => Adapter => backend provider

So at least we have three calls that can be treated strait foward or have tigh manipulation between them to fullfill specific requirements or have necessary ajustments between each layer untill reach backend requirement. These types of adapters allows the main service layer to gather data for multiple sources and provide single responses with data unified on it.


## Configuration barier

We started to deploy the layers on a out-of-the-box Azure APIm single instance (with developer tier) and we used the APIm hostname to perform the calls between them.

[APIm advanced policies] allows to configure and perform some usefull operations like we needed it, given the following:

- Url service A: https://apim-demo-codit.developer.azure-api.net/services/service-a/operation-flights
- to call adapter B: https://apim-demo-codit.developer.azure-api.net/adapters/adapter-b/operation-flights

For the sake of simplicity every mechanism of authentication, such as API keys, were disabled just for demo propuses. So at the configuration policy of the service A, we were able to use:

- **rewrite-uri** policy, at first sight makes sense to just change the URL and perform the request into another service/endpoint.

```xml
<policies>
    <inbound>
        <base />    
        <rewrite-uri template="adapters/adapter-b/operation-flights" copy-unmatched-params="true" />
    </inbound>
    ...
</policies>
```

and

- **send request** policy, in our opinion, a much more elegant way and understandable regarding the concept of having service A that's sending a new https request to the service B (our adapter) endpoint.

```xml
<policies>
    <inbound>
        <base />    
            <send-request mode="new" response-variable-name="result" timeout="300" ignore-error="false">
                <set-url>https://apim-demo-codit.developer.azure-api.net/adapters/adapter-b/operation-flights</set-url>
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
At our first sight everything was working great and as expected that a major platform as Azure API Management could handle a full catalog set of services and adapters (in our architecture design) and interactions between them.

So when we moved our solution into the client's Azure APIm instance, we were unable to have HTTP/HTTPs calls between hosted services... we aren't able to have loopback calls... instead we got 500 - Internal Server error.

Our client's APIm instance, is configured inside of a private virtual network exposed to the internet using the Application Gateway (WAF) and with the FQDN service usage weren't able to get 200 Ok responses on our "working" solution. By performing APIm traces (link to this concept), we saw that APIm was resolving the domain into his private IP, fowarding the request (As expected) but refusing a connection to himself.

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
        "Error occured while calling backend service.",
        "A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond 172.xx.xx.x:443"
        ]
    }
}
...

```

We got stuck! We started a trial and error debugging to figured out what was the impediment! Why the APIm wasn't taking our loopback calls as we had at our own subscription (the works on my machine effect 🤔) 

We started to change the address into what cames at our minds, such as:

- https://localhost/... - 404 Not found | Doesn't makes sense... every server know the localhost special term.. right? No good... 

- https://127.0.0.1/... - 404 Not found | like the localhost... the loopback address should work... this is getting impossible to figured out what is happening - even using the Apim-Trace output.

- APIm's private ip address - 500 Internal Server Error | Probably to many network wiring complications that don't allow internal referenced requests. Trace ouput was the same of above, resolved into the private but not allowing connection into it;

- Application gateway public (that's configured to foward external traffic into APIm) ip address - 404 Not found | But on the response body we can see that it was the azure aplication gateway who responded... again odd, because it's the nears solution regarding our own environment and the WAF should let the request goes out and came back in... no good;

- APIm's public ip address (the one that's given by Azure) - we've got two different responses: 
    - Inside the client's network: 500 Internal Server Error | Trace ouput was the same of above, resolved into the ip address but not allowing connection into it;
    - Public network: 504 Gateway Timeout | This is the Azure Aplication Gateway response time out guessing that foward the request and never got any response from the internal load balancer;

With no more ideas or suggestions to try, we felt that we might reach a bug or a specific physical APIm limitation, with no toher way unless to ask for offical help.

### How we solved it

We got really stuck without many ideas to try/debug and move forward.. it was time to involve Azure's support for the rescue. And once again, the support proved to been effective and reliable, after two hours of interaction (for them to really understand whats going on under the hood) they came up with the "solution" - yes! We got over it.

To allow loopback calls support requested us to add a new header on the HTTP request that includes the **Host** name of the APIm domain and use the **127.0.0.1** at the url address such as:

```xml
    <set-header name="Host" exists-action="override">
        <value>apim-demo-codit.developer.azure-api.net</value>
    </set-header>
```

Eureka 😎 So we've changed our APIm's policy to include an additional header with host domain, and it started to accept loopback calls:

```xml
<policies>
    <inbound>
        <base />    
            <send-request mode="new" response-variable-name="result" timeout="300" ignore-error="false">
                <set-url>https://**127.0.0.1**/adapters/adapter-b/operation-flights</set-url>
                <set-method>GET</set-method>
                <set-header name="content-type" exists-action="override">
                    <value>application/json</value>
                </set-header>
                <set-header name="Host" exists-action="override">
                    <value>**apim-demo-codit.developer.azure-api.net**</value>
                </set-header>                                
            </send-request>
        <return-response response-variable-name="result" />
    </inbound>
    ...
</policies>
```

Azure's support team explained to us that a *recent change* on APIm's core (from January release) was affecting these loopback type calls, and at the current configuration (hosted as a private resource inside the subscription) it requires a Host header to allow the internal load balance redirect to the right APIm instance that helds and knows the service that we wanted to call. Also to be carefull regarding the casing of the headers names, though we've tried both ways and seems not be case. 

After, we also found out that even with this new Host header, we can also use the **localhost** instead on the set-url field - meaning that the new header is what makes the real-deal.
Also we can use another policy type like the rewrite, but again adding the Host as a header such as:

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

We didn't find any reference at oficial documentation regarding this specific header configuration, thought we'll suggest it. 

Keep in mind that if you loopback calls need authentication you need to foward those specific headers to the service your're reching out. Otherwise, APIm can't stripes those out and you'll get an 403 by the authentication fail response. At our scenario, at the policy, we added additional code to grab the key and forward it on the "new" request that we're performin at this stage. Azure's support also confirmed to us that we had to do-it manualy because at the moment there isn't an automated policy or key work. So we ended up at our policy like:

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


## Conclusions and aknoagle

APIm is a remarkable service to handle API's and to have a uniformization accross your service catalog, but like any other we can find some challenges when we're pushing our arhictecture towards new goals and specific scenarios. Besides this little loopback issue that we cauth at our client's scenario we would like to remark some tips that we always keep with us:

- Don't only perform your tests using the Azure's APIm portal interface, sometimes we got different results from the portal when we call outside (example from our postman collections);
- Test inside and outside the APIm's network, if you have yours deployed within a private network, be sure to test both secnarios: private and public - we also found out unexpected results;
- Keep in mind the internal APIm's cache, at each configuration change, remeber that it takes moments to load up the new configuration (support said it can be up to one minute.)
- Try to keep your policies with low complexity as far as you can;
- Ensure to use the trace header feature (link) to see what happened with your request;
- Debug Azure API Management policies in Visual Studio Code - (see how)[https://docs.microsoft.com/en-us/azure/api-management/api-management-debug-policies]
- Check if the domain is working properly (not expired, has dns records, etc) before you think that you're the problem, or if you don't have a typo on it;
- Avoid the "rewrite" at your policy and use the "send request" at least on this type of scenario;
- When you receive a 400 bad request (testing from the portal) check for the CORS bypass option to otbain more information;

Last but not least, I would like to send a big thank you to all our Codit fellows that help us on the way of this challenge, and of course also to Azure's support for their contribution. 