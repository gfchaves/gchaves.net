---
title: Azure DataFactory - Interact with rest API using a managed identity
author: Chaves
date: 2020-07-16 22:00:00 +0000
categories: [Blogging, Azure, Cloud]
tags: [.net, azure, datafactory, function, serverless, managedidentity]
---
Yes! It's possible! We were trying hard to call Azure Data Factory REST API from one Azure function (serverless) and use the configured user-managed identity (of that function, the account that will be authenticated) to interact with other resources. This means that you can have a specific managed identity that you could use to define permissions and explorer security for single or multiple accounts that you'll be configuring on your setup.

A little sketch to show how we pretended to perform this interaction:

![az-datafactory](/assets/img/posts/az-datafact-1.png)

Steps that represent the above diagram, although we'll focus on function and data factory interaction:

* receive an HTTP Request with subscription key to APIm (Azure API management);
* apply policies and set backend API our function (outbound);
* apply user-managed identity to retrieve a token that function will validate from authorization settings;
* function requests an authorization token with the current user-managed identity that you configured at Identity blade on function
![az-datafactory-2](/assets/img/posts/az-datafact-2.png)

Using the .NET API as follows:

```csharp
var azureServiceTokenProvider = new AzureServiceTokenProvider();
accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://management.azure.com/").ConfigureAwait(false);
ServiceClientCredentials cred = new TokenCredentials(accessToken);
```
The AzureServiceTokenProvider (from microsoft.azure.services.appauthentication) class constructor can receive a connection string with the definition of what identity you want to use to request the authentication token. So you can override with your application id, or configure on function settings/configuration blade (much similar to a web.config appSettings dictionary - for those who have more experience with asp.net apps) with the default name of "**AzureServicesAuthConnectionString**". Here some variants:

```csharp
//Specifing the application
var azureServiceTokenProvider = new AzureServiceTokenProvider($"RunAs=App;AppId=your_app_id");

//to run locally on your Vs instance
var azureServiceTokenProvider = new AzureServiceTokenProvider($"RunAs=Developer; DeveloperTool=VisualStudio");
```

Now that our function has a valid auth token (jwt) we can use the .NET API (package microsoft.azure.management.datafactory) of data factory to instantiate and start our interactions, as follows:

```csharp
var myDataFactoryClient = new DataFactoryManagementClient(cred)
{
        SubscriptionId = subscriptionId
};

var dataFactoryRequestResult = myDataFactoryClient.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName).Result;
```
Our managed-code data-factory client has the method to create a run on a pipeline with HTTP request acting behind the scenes :-) That's why the authorization token saved on our credentials variable is required on the DataFactoryManagementClient class constructor. So it's an HTTP wrapper to perform the interaction with data factory REST API.

Now you might have an error because it's necessary to configure the necessary permissions on data factory to our function. Remember that we're using a managed identity that has been impersonated in our token, so the data factory REST API would validate the token claim (again behind the scenes) and it's obvious that our managed identity has the requested authorization. To configure, go to data factory resource dashboard -> Access Control -> Add role assignment and select your function and the pretended role (DataFactory Contributor), hit save.

You can after check if permission is enabled on Role Assignments as shown below
![az-datafactory-3](/assets/img/posts/az-datafact-3.png)

It took us a while to figure out how to give the "claims" on the auth request because our managed identity when interacts with data factory REST API needs to be authorized has expected to do so.

## To Remark
Data Factory REST API can be used natively (with regular HTTP requests) or with a managed API (.net, PowerShell) and this gives us the freedom to automate processes and interact with resources (pipelines) inside our data factory instances.

Data Factory has triggers, but there'isn (yet of the time of this writing) an HTTP based trigger. So we went for the REST API. 

Also at this time of writing, data factory only supports System Managed identity from it-self to interact with other resources, and this identity is created on data factory deployment process - you can override to specify your system managed identity by specifying it on your ARM template or deployment script.

Some links that provided us guidance on the way:

* [https://docs.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory-rest-api](https://docs.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory-rest-api)
* [https://docs.microsoft.com/en-us/azure/data-factory/data-factory-service-identity](https://docs.microsoft.com/en-us/azure/data-factory/data-factory-service-identity)
* [https://docs.microsoft.com/en-us/azure/data-factory/concepts-pipeline-execution-triggers](https://docs.microsoft.com/en-us/azure/data-factory/concepts-pipeline-execution-triggers)
* [https://docs.microsoft.com/en-us/azure/data-factory/](https://docs.microsoft.com/en-us/azure/data-factory/ quickstart-create-data-factory-rest-api#create-a-data-factory)
* [http://eatcodelive.com/2016/02/24/starting-an-azure-data-factory-pipeline-from-c-net/](http://eatcodelive.com/2016/02/24/starting-an-azure-data-factory-pipeline-from-c-net/)
 

Hope that this post can help you in some way 😎👌