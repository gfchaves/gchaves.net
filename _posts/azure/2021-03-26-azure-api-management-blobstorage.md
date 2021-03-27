---
title: How to Use Azure API Management Service as Frontstage of a Blob Storage Account
author: Chaves
date: 2021-03-26 00:00:00 +0000
categories: [Blogging, Azure]
tags: [.net, azure, apim, blobstorage,api, facade]
---

# Intro

APIs are becoming the "lingua franca" between all type of entities, companies, services, applications, and many others, but this is not without challenges. In this blog post we outline how you can use Azure API Management service as a Frontstage of a Blob Storage Account.

# Set the stage

In times where AP’s are becoming the “lingua franca” between all type of entities, companies, services, applications, and many others, where the expression of “just give me your API specs and we’ll take it from there,” it’s safe to say APIs are the foundation of any digital project or integration.

This means that APIs are here to stay, and we need to find more efficient ways of managing the many APIs we are interacting with. Services like Azure API Management (APIm) can provide us a unified way to have all our services portfolio under a single and centralized tool and help streamline our interactions between multiple APIs.

The first step in integrating an existing service from your catalog is by virtualizing it in your APIm.

![apim schema](https://www.codit.eu/wp-content/uploads/2020/11/apim-blobstorage-blog-sample.png)

# Challenge: Azure BlobStorage API

We were set a challenge to create a set of services to integrate a customer document management system into the APIm instance. The client’s DMS is also developed with an Azure service so we were looking for an easy approach to integrate, and help our customer on his service unification mission, using the Azure API Management service.

A great example of this service unification is the services contract, meaning that all deployed services at APIm have defined standard input and output formats across the collection, recurring on the JSON format and (when is possible) a REST approach paths.

Azure BlobStorage API doesn’t seem to yet have JSON format at API response messages, instead it relies on the traditional XML format.

Here’s an example blob storage API response for a file not found at our container, a typical 404 (not found response):

{% gist 1b5296bbff76c95e96b20a96d4342c55 %}

At our customer API “standardized version” HTTP responses, we were looking to have something in JSON format like this:

{% gist 2df22086d90adcc623d93e802203ed32 %}

# Solution: Azure API Management Service

Azure API Management service, offers easy ways to accomplish a response transformation from the XML format into the more desirable JSON format, but we took it a step further.

We used the APIm capabilities and created a virtualized API (RESTfull paths approach with a custom body entity model) with a set of operations that act as a façade for our Azure blob storage destination container for client’s files.

This new API also comes with the standardized authorization mechanism across all other available services, and defined specific policy transformations to map the input requests to blob storage and vice-versa, with some extra logic to have other requirements involved, such as logging and interoperability with other systems.

# Wrap-Up

This means that Azure API Management service is a perfect tool to be a front stage for you Azure Storage account, where you can develop your APIs to be native, standard, and unified to handle your needs with storage, whenever are files, tables, blobs and everything else that might be available on it. And then, we’ve created a singular response type message format across all APIs regardless of their final destination or service provider that is behind it.

To help you understand how we can create Azure API Management service policies to handle the blob storage account API interaction, we’ve added some examples at the GitHub repo of Azure API management policy snippets (PR is on the go, but you can see the juice 😎). Here you can see examples of how to create a new file at a blob container, get that file (aka download it) and the delete operation. Also, we added metadata properties to enrich the information that we’re storing there to have more context and future features that can depend on this additional and valuable information.


Got any comments or questions? Feel free to reach out. Thanks for reading.

*this post was originally published on [Codit's blog](https://www.codit.eu/blog/azure-api-management-service-frontstage-blob-storage-account/)*