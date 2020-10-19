---
title: Azure APIm - How to configure a loopback services
author: Gonçalo Chaves
date: 2020-10-17 14:10:00 +0800
categories: [Blogging, Tutorial]
tags: [azure, apim, loopback, webapi, webservices]
---

## Intro

We were challenged to build a framework that lies on a specific platform architecture that we hosted by azure resources. Althought the complexitity of the given design principles for our customer, together both teams came up up a design and process that allow resources be setup to work together on a set of features and services to provide a full enterprise service bus on the cloud - in which it's impressive that his way.

Nowdays talking about services is the same of talking about of the most busiest airport in the word - before this pandemic time (smile) - where only a set of procedures and strucuterd frameworks at in place to be possible operate in real safe procedures that could be safe even when everything looks messy like ordinary traffic jam. 

So services (like airplanes) have common concepts (like wings and engines) but hey differ in their specific ways regarading technology, business requirements, developers capacities or what was the in place procedure at that time... like the two major aircraft manufactures produce different sizes, styles and types but all together arround same principles - like our entrerprise services. 😊 (meme image of "services every where"). At his stage and to keep it more simple our services includes every service type/architecture (micro-services, soap, rest, soap-ish, rest-ish - you got the idea 😁). This means that "the airport" is our hub like the Azure API Managmenent (aka APIm) is our services manager that allows us to have such great capabilities to control and manage our services behavirous, uniformization, rate limits, and access control. 


But like any other airport/service manager we faced some challenges because of the way we want to manage our architecture inside of it. At this post we'll preent parts of the current architecture that leads us to enconter the challenge of having multiple services hosted with in it but with the necessity of having caals between then that came up in loopback calls for the same APIm instance.

## The challenge

We started to have a single APIm instance that could host our services layers: 

- **Channels APIs** - public and external facade of internal services to have a single uniformization and a set of security rules, authentication procedures and rate limits to be in place when exposed wide open to the internet;
- **Services APIs** - first internal layer that represents the service implementation as a entity itself but usign another services layers to abstract the backend implementation-specific logics;
- **Adapters APIs** - the nearest service layer implementation that hold the specific logic of the backkend services. This acts as the bridge between the provider service implementation and the service common architecture layer - by holding the spefific implementation. We can have multiple adapters that services the same service and act as data connectors.

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

So when we moved our solution into the client's Azure APIm instance, we were unable to have HTTP/HTTPs calls between hosted services... we aren't able to have loopback calls...

Our client's APIm instance, is configured inside of a private virtual network exposed to the internet using the Application Gateway (WAF) and with the FQDN service usage we're getting 400 bad request as resposes into our "working" solution. By performing APIm traces (link to this concept), we saw that APIm was resolving the domain into his private IP, fowarding the request (As expected) but refusing a connection to himself.

[put the apim error ]

We got stuck! We started a trial and error debugging to figured out what was the impediment! Why the APIm wasn't aking our loopback calls as we helded at our own subscription (the works on my machine effect 🤔) 

We tried to change the address for:

- APIm's private ip address - 400 Bad request | Probably to many network wiring complications that don't allow internal referenced requests;
- APIm's Application gateway public ip address - 400 Bad request | Odd because it's the nears solution regarding our own environment and the WAF should let the request goes out and came back in... no good.
- https://localhost/... - 400 Bad request ? (ja n me lembro a confirmar) | Doesn't makes sense... every server know the localhost special term.. right? No good..
- https://127.0.0.1/... - 400 Bad request | like the localhost... the loopback address should work... this is getting impossible to figured out what is happening..

[colocar os exemplos de resposta para cada caso acima...]


### How we solved it

We got really stuck without many ideas to try/debug and move foward.. it was time to envolve Azure's support. Once again the support proved to been effective and reliable, after two hours of interaction (for them to relly understand whats going on under the hood) they came up with the solution.

To allow loopback calls support requested us to add a new header on the HTTP request that includes the Host name of the APIm domain and use the 127.0.0.1 at the url address:

```xml
    <set-header name="Host" exists-action="override">
        <value>apim-demo-codit.developer.azure-api.net</value>
    </set-header>
```

Eureka 😎 So we've changed our policies to include an additional header with host domain, and it started to accept loopback calls:

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

Azure's support team explained to us that a recent change on APIm core (from January release) was affecting these loopback calls, and at the current configuration (hosted as a private resource inside the subcription) it requires a Host header to allow the internal load balance redirect for the right APIm instance that helds and knows the service that we wanted to call. 

## Conclusions

TODO
