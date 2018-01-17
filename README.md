---
services: app-service, functions
platforms: nodejs
author: ityer
---
[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fitye-msft%2Fazure-consumption-cost-sample%2Fmaster%2Fazuredeploy.json)


# Node.js Azure Function for calculating your subscription cost

This is a sample HTTP trigger Azure Function that returns consumption cost per subscription.

## Deploy to Azure

The automated deployment provisions an Azure Storage account and an Azure Function in a Dynamic compute plan and sets up deployment from source control. 

The deployment template has a parameter `manualIntegration` which controls whether or not a deployment trigger is registered with GitHub. Use `true` if you are deploying from the main Azure-Samples repo (does not register hook), `false` otherwise (registers hook). Since a value of `false` registers the deployment hook with GitHub, deployment will fail if you don't have write permissions to the repo.
Iy you want to deploy it from your own repo, make sure your account is [authorized GitHub.com](https://github.com/blog/2056-automating-code-deployment-with-github-and-azure), otherwise, it will fail with authentication error.  
## How it works



If the function will be called from a mobile client or a JavaScript web app, we recommend that you add authentication to your Function using [App Service Authentication/Authorization](https://azure.microsoft.com/en-us/documentation/articles/app-service-authentication-overview/). The API key is usually insufficent for security purposes since it can be discovered by sniffing traffic or decompiling the client app.

## Calling the function



Response:



## Learn more

- [Authentication and authorization in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/app-service-authentication-overview/)

