---
title: REST is not a service... it's an architectural style
author: Chaves
date: 2019-03-18 22:00:00 +0000
categories: [Blogging, Tools, Architect]
tags: [REST, http, web, api, architecture]
---

In this few years, I've been given technical interviews and also performing interviews my self with candidates. Each time that we talk about services or more specific, web services and APIs it's unavoidable the REST concept that comes into play. Recently at another interview that I was having the recruiter also questioned about **"REST Services"** ... well is a fairly misconception that technical people always refer to REST as being a type of web-based services, or a type of API. Well, REST means **"Representational state transfer"** an at Wikipedia definitions states "is a software architectural style that defines a set of constraints to be used for creating web services." - a set of constraints, format, and rules not a "service" per se.

![concorde_engine](/assets/img/posts/concorde_engine.jpg)

Saying that REST is service(s) is the same as SOAP is also a service(s).... which isn't. Right ? ;) Even for SOAP (Simple Object Access Protocol) ends with the word protocol so definitely different than a service who can be exposed on endpoints to be consumed by other entities (services, applications, etc.)

I mean, it's a hard relationship between the format itself and the protocol that we're choosing for our web services or APIs, like XML format is used on SOAP and JSON on REST messages. Then we can make use of words and extend it for the expression "RESTfull services" - in this case it's a more accurated because, it's states that our services are developed and available for use with a REST format/protocol on the messages that are exchanged. 

So if REST isn't a service, what's then? A format that allows us to extract the most of the HTTP protocol by seeking different actions on the same endpoint by changing the HTTP verb used on the message. A lot of RESTfull APIs are available on the internet, and many platforms provide them for small or deep integrations and interoperability between applications. A simple example is the mailChimp RESTfull API, their implementation is simple and a truly RESTfull one. You can check their documentation for further details and samples - it's awesome.

So when my teams developed APIs or services with a REST format, I always try to push them into the most common and standard-ish format, by keeping the same endpoints URLs with the five verbs (GET, POST, PUT, PATCH, DELETE) to interact with the resources. Although not every endpoint should have all options available, by requirements design or feature availability, the most common mistakes that I find with RESTfull implementations are with the URLs of the resources and their actions. let me give you an example: You have to implement a RESTfull API for your customer service platform account, so the famous CRUD operations for a user account registration. The most "restfull" way to implement should be:


`GET /api/user` - Retrieves all the public data of the user accounts 

`GET /api/user/{userid}` - Retrieves the public account data of a specific user if exists.

`POST /api/user` - Should perform an update for each account that is inside of the HTTP message. For instance, you want update the user status for a bunch of accounts.

`POST /api/user/{userid}` - Should perform account data update for the specific user account if exists

`PUT /api/user` - Should create new accounts has many that are on the HTTP message.

`DELETE` - for the same urls but to perform delete operations, you get the point ;)


Also, another issue that I commonly found is with the mismatch between the HTTP verb and the action that the service is performing. For instance, a POST to perform a "new" or create operation on an entity. Again we can say that we're developing a RESTfull API but not following the standard, or the specification itself.

That being said, REST is a style specification, that is very commonly associated with web services or APIs and with JSON format for message exchange. Although it might be very glowing for you to say that you develop REST services, it's not accurate... you develop services with the REST specification or at least RESTfull services. But be aware, it's a specification not services itself. Can I use REST specification off HTTP scenario? Well, why not? But the definition states for web-based, but it's possible to use this specification with other scenarios.

I find myself thinking about this and other misconceptions because I do believe that candidates are able to develop and create powerful services or APIs, but I'm sure that they aren't fully aware of what is behind the hood or what is the real conception of the terms and formats. Also because the recent tools and frameworks can and provide accelerators for these new architectural styles and guides, but they also have their limitations and hide most of the heavy-work, blinding the real concepts underneath.