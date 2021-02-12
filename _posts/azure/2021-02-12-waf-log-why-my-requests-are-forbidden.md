---
title: Why my requests are being refused by Azure WAF?
author: Chaves
date: 2021-02-12 08:00:00 +0000
categories: [Blogging, Azure]
tags: azure, apim, waf, http, requests, log]
---

## Intro

The Azure [Web application firewall aka WAF](https://docs.microsoft.com/en-us/azure/web-application-firewall/ag/ag-overview) it's a great tool when it comes to protect our cloud web applications from the most common and know attacks. So this protection is built on a set of [OWASP](https://owasp.org/) [CRS rules](https://docs.microsoft.com/en-us/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules) and those rules contains different types of request audience, i.e, are looking for different causes or suspecting things in the incoming request and given them a score as those rules are being evaluated. When this score reach a certain number, you request is or can be denied (if your WAF is configured on prevention mode - the recommended, on the detection mode it only make a log of it). Also these rule sets have a version and you can configure the one you feel that fits your needs/limitations regarding the current scenario you're working with.

## Debugging

So after our WAF evaluated our incoming requests this means that some of them are blocked and typically we get a HTTP 403 forbidden as shown:

```html
<html>

<head>
	<title>403 Forbidden</title>
</head>

<body>
	<center>
		<h1>403 Forbidden</h1>
	</center>
	<hr>
	<center>Microsoft-Azure-Application-Gateway/v2</center>
</body>

</html>
```
So what has our request that is affecting or being cough by WAF rules? It's time to debug it to find out what we have on the request, and this could be a little bit hard to get, but with Azure logs we can fetch some interesting data to evaluate.


Our request:

```json
curl --location --request POST 'https://my-azure-apim-domain/my-api-sample/email/send' \
--header 'Ocp-Apim-Subscription-Key: myApimSubcriptionKey' \
--header 'Ocp-Apim-Trace: true' \
--header 'Content-Type: application/json' \
--data-raw '{
    "sender": {
        "email": "no-reply@testdomain.com",
        "name": "Sample Email Test sender"
    },
    "recipients": {
        "to": [
            {
                "email": "goncalo.chaves@codit.eu",
                "name": "Chaves"
            }
        ]
    },
    "subject": "This is the subject of the email test that's is being block",
    "body": "&lt;html&gt;Some sort of html message body&lt;/html&gt;",
    "isBodyHtml": true,
    "isHighImportance": false,
    "isTransactional": false,
    "campaignId": ""
}'
```

Http response is a 403 forbidden... nothing awkward with our request? on the body our json even has encoded html characters to avoid parsing issues... still no good. A good way to debug and seek for what is happening is to check the logs, as an example you can run the following log query (at the WAF Monitoring/Log blade) to find out more about you request that was blocked:

```xml
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| order by TimeGenerated desc
```

After select one of the records from query result sample, here's what we get:

|TenantId|****|
|TimeGenerated [UTC]|2021...|
|ResourceId|***|
|Category|ApplicationGatewayFirewallLog|
|ResourceGroup|*****|
|SubscriptionId|guid|
|ResourceProvider|MICROSOFT.NETWORK|
|Resource| my resource|
|ResourceType|APPLICATIONGATEWAYS|
|OperationName|ApplicationGatewayFirewall|
|requestUri_s| ****/v1/email/send|
|Message|Remote Command Execution: **Unix Command Injection**|
|instanceId_s|appgw_1|
|SourceSystem|Azure|
|clientIp_s| 176.****|
|ruleSetType_s|OWASP_CRS|
|ruleSetVersion_s|3.0.0|
|ruleId_s|**932100**|
|action_s|**Matched**|
|site_s|Global|
|details_message_s|Warning. Pattern match ....  at ARGS:body .... |
|details_data_s|Matched Data: ;gzip found within ARGS:body: &#27;gzip|
|details_file_s|rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf|
|details_line_s|79|
|hostname_s|****|
|transactionId_g|93b8f29d-b3bc-94b2-aa48-dee92e81f7b3|
|policyId_s|default|
|policyScope_s|Global|
|policyScopeName_s|Global|
|timeStamp_t [UTC]|2021-...|
|Type|AzureDiagnostics|
|_ResourceId|...|

As you can see we have some good info regarding what was matched to hit the OWASP rule, at this example the rule 932100 it's a match and even the detail reval the pattern that was evaluated and the matched data from regex expression 😁. So "gzip" word on the request body was a hit.

So this is very usefully for understand how the rules are applied and what they are exactly doing though out our requests. Now the challenge is how we can overcome the rule, whenever is to change our request logic/mapping or have to disable a specific WAF rule to avoid those blocks.

It's recommended to go with the changes on the request/model or service logic to avoid the rule disable, but in some cases it can be impossible to change and WAF gives you this possibility with a customization.

## More about WAF


Some great resources:

* A great intro session at [Azure Friday](https://www.youtube.com/watch?v=Kgn_Y9nkv94)
* [Combine rules into your own policy](https://docs.microsoft.com/en-us/azure/web-application-firewall/ag/create-waf-policy-ag)
* [Azure Resource template gallery](https://azure.microsoft.com/en-us/resources/templates/)
* [Top 10 applications security risks by owasp](https://owasp.org/www-project-top-ten/)
* [Custom rules samples and use cases](https://techcommunity.microsoft.com/t5/azure-network-security/azure-waf-custom-rule-samples-and-use-cases/ba-p/2033020)