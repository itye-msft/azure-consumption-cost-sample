---
services: app-service, functions
platforms: nodejs
author: ityer
---
[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fitye-msft%2Fazure-consumption-cost-sample%2Fmaster%2Fazuredeploy.json)


# On-demand calculation of Azure consumption cost

This code sample demonstrates how to use Azure billing commerce APIs to find consumption cost per subscription and resource-group.
We will use an Azure Function to execute the code.

## The challenge

The [Billing API](https://docs.microsoft.com/en-us/javascript/api/overview/azure/billing?view=azure-node-2.2.0) and the [Consumption API](https://docs.microsoft.com/en-us/javascript/api/overview/azure/consumption?view=azure-node-2.2.0) don't expose the cost of the  consumption. Instead they only expose the ability to interact with past invoices (which is not service an on-demand need), or receive consumption quantities without cost.

For example, you will be able to tell that some subscription had 5 compute hours, but the cost is not present, nor the price per compute unit.

In addition, some functions are also not available for sponsored accounts.


## Explaining the solution

[Azure Commerce APIs](https://docs.microsoft.com/en-us/azure/billing/billing-usage-rate-card-overview)  provides 2 functions:
1.	Resource usage. This will give you consumption data for an Azure subscription.
2.	Resource RateCard. This will give price and metadata information for resources used in an Azure subscription. It contains mainly a price for each meter. 

This solution simply ties all the edges to calculate the actual cost under a single wrapper. 

There are 4 steps to extract the cost:
1. Authenticate to Azure and obtain the `Credentials` object.
2. Use the `Credentials` object to pull `rateCards` which are the rates set up for the given account. Rate are also called `Meters`.
3. Use the `Credentials` object to pull `Consumption` data for each resource in the given subscription. Each consumption listing contains the `Meter ID` from the rate cards. Consumption provides you with a quantity per meterID. 
4. The last step is to multiply the rate value with the consumption quantity. This is the `cost`.

Results can be filtered by resource group or resource tags. The Node.js code makes use of the [Azure node.js SDKs](https://github.com/Azure/azure-sdk-for-node/tree/master/lib/services/commerce).

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

### Calling the function
Example of calling the function:
```sh
curl -H "Content-Type: application/json" -X POST -d '{"filter":"resource-group-name","detailed":"true"}' https://<function-name>.azurewebsites.net/api/get-consumption-cost-node?code=<code>
```

> If the function will be called from a mobile client or a JavaScript web app, we recommend that you add authentication to your Function using [App Service Authentication/Authorization](https://azure.microsoft.com/en-us/documentation/articles/app-service-authentication-overview/). The API key is usually insufficent for security purposes since it can be discovered by sniffing traffic or decompiling the client app.

> The function will take a considerate amount of seconds to execute, because it performs 3 API calls: Login, Get Rates, Get Consumption. A real-world scenario will use caching/sessions to avoid multiple logins and Get rates.

***The response***

Is an object in the form of: 

`{ total: (Number, the total cost), details: (Key-Value pairs of resource and cost)}`

For example:
```javascript
{ total: 10.3, details: { res1:5, res2: 5.3}}
```

## Learn more

- [Authentication and authorization in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/app-service-authentication-overview/)

