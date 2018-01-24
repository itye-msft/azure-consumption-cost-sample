---
services: app-service, functions
platforms: nodejs
author: ityer
---
[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fitye-msft%2Fazure-consumption-cost-sample%2Fmaster%2Fazuredeploy.json)


# Node.js Azure Function for calculating your subscription cost

This code sample demonstrates how to use Azure billing commerce APIs to find consumption cost per subscription and resource-group.
We will use HTTP trigger Azure Function to execute the code.


This sample uses 2 [Azure Commerce APIs](https://docs.microsoft.com/en-us/azure/billing/billing-usage-rate-card-overview) to calculate the consumption cost for an Azure subscription. Results can be filtered by resource group or resource tags. The Node.js code makes use of the [Azure node.js SDKs](https://github.com/Azure/azure-sdk-for-node/tree/master/lib/services/commerce).

## Deploy the sample to Azure

The automated deployment provisions an Azure Storage account and an Azure Function in a Dynamic compute plan and sets up deployment from source control. 

When you setup the Azure Function, you will be requested to supply information and credentials to initialize the function. This is a one time step. Make sure you have in hand:

| Name | Type |  Description |
| --- | ---- | --- |
| clientId | string | Your service principal ID |
| clientSecret | string | Your service principal secret |
| tenantId | string | Your subscription tenant ID |
| subscriptionId | string | Subscription ID to pull consumption info for |
| offerId | string | Must be specific code taken from [Microsoft Azure Offer Details](https://azure.microsoft.com/en-us/support/legal/offer-details/)

The deployment template has a parameter `manualIntegration` which controls whether or not a deployment trigger is registered with GitHub. Use `true` if you are deploying from the main Azure-Samples repo (does not register hook), `false` otherwise (registers hook). Since a value of `false` registers the deployment hook with GitHub, deployment will fail if you don't have write permissions to the repo.
Iy you want to deploy it from your own repo, make sure your account is [authorized GitHub.com](https://github.com/blog/2056-automating-code-deployment-with-github-and-azure), otherwise, it will fail with authentication error.  

## Calling the function
Once the template is deployed, 2 functions will be created:
1. `get-consumption-cost-node`: The function will use the provided credentials and parameters to login into Azure, pull both Rate and Consumption data and calculate the actual cost, based on the rates and consumption.
2. `download-consumtion-cost`: The function will use the provided credentials and parameters to login into Azure, pull both Rate and Consumption data and calculate a detailed report of the actual cost of each resource, based on the rates and consumption. The result is a `csv` files which can be downloaded from the function.

***Optional parameters:***

| Name | Type |  Description |
| --- | ---- | --- |
| detailed | bool | true if the consumption info should contain details. Optional. relevant only for `get-consumption-cost-node` |
| filter | string | A way to limit results only to a specific resource instance info. For example: resource group, tags etc. Optional. |
| startDate | Date | Start time for the report. The date format is [ECMAScript](http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.15) YYYY-MM-DDTHH:mm:ss.sssZ .  Optional. Defaults to 24 hours ago|
| endDate | Date | End time for the report. The date format is [ECMAScript](http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.15) YYYY-MM-DDTHH:mm:ss.sssZ.  Optional. Defaults to now.|
| granularity | string | Can be: "Daily" or "Hourly". Optional. Default is Daily. |

The paramaters are sent as `json` in the body of the POST request.
For example:
```sh
curl -H "Content-Type: application/json" -X POST -d '{"filter":"resource-group-name","detailed":"true"}' https://<function-name>.azurewebsites.net/api/get-consumption-cost-node?code=<code>
```

> If the function will be called from a mobile client or a JavaScript web app, we recommend that you add authentication to your Function using [App Service Authentication/Authorization](https://azure.microsoft.com/en-us/documentation/articles/app-service-authentication-overview/). The API key is usually insufficent for security purposes since it can be discovered by sniffing traffic or decompiling the client app.

> The function will take a considerate amount of seconds to execute, because it performs 3 API calls: Login, Get Rates, Get Consumption. A real-world scenario will use caching/sessions to avoid multiple logins and Get rates.

***The response***

Is an object in the form of:
```javascript
 { total: (Number, the total cost), details: (Key-Value pairs of resource and cost)}. For example: { total: 10.3, details: { res1:5, res2: 5.3}}
```

## Learn more

- [Authentication and authorization in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/app-service-authentication-overview/)

