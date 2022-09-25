Title: Update Azure Cosmos-DB RU using Powershell Azure Function

Published: 24/08/2019
Image: /posts/images/update-cosmosdb-ru.jpg
Tags:
  - azure
  - powershell
  - cosmosdb
  - cost
  - automation
  - azure-function
---

This article is one of the implementations of my previous article [Optimize the cost of Azure Cosmos DB](https://www.linkedin.com/pulse/optimize-cost-azure-cosmos-db-pankaj-rawat/), I recommended, scale-up/ scale-down Cosmos DB RU according to your business hours, help you to save money. In my current application, we have 6 Environments (Development, Integration, Performance, QA, Stage and Production), except the production environment, I want to scale-down CosmosDB RU in non-business hours (7 PM to 10 AM and weekends) and scale-up RU on business hours (10 AM to 7 PM). Application won't stop by scale-down RU, but won't able to handle high workload.

I have a single database with 10 collections for each environment. Total 8800 RU is reserved and estimated cost $507 per month for per environment.

**RU** 8800 Per day cost - **$16.90**

**RU** 8800 Per month cost - (**$16.90\*30) = $507**

With my scale up-down logic, we have to pay only **$270** per month for per environment which is 46% less compared to the previous bill. 8800 RU Per month cost -**$126** (for Business hours) + 4000 RU Per month cost -**$144.64** (Non-business hours and weekends)

<img src="/posts/images/update-cosmosdb-ru-1.jpg">

Per collection minimum reserved RU limit is 400, therefore we can't go below that. You can scale up-down CosmosDB RU using SDK, API, CLI and PowerShell. Azure CLI and Powershell was my first choice, but both types of command need to execute ether using WebJob or Azure Automation. I found the cheapest way to execute Powershell command is "Azure Function", Though [Powershell Azure Function](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-function-powershell) still in preview and I have no plan to execute these scripts in Production.

While creating a Powershell Azure Function, I faced [issue](https://stackoverflow.com/a/39985646/4140278) to import Powershell modules, You can import Powershell module in local machine and then upload in function app inside "Modules" folder using KUDO or you can keep your Powershell modules file inside your function app and check-in with your code. below is my sample code

<img src="/posts/images/update-cosmosdb-ru-2.jpg">

Under one function app instance, I created two separate **RU Scale-Up** and **RU Scale-Down** Timer Trigger Function.

Scale Up Function Trigger weekdays at 10 AM IST using below CRON expression.
<img src="/posts/images/update-cosmosdb-ru-3.jpg">

Scale Down Function Trigger weekday at 7 PM IST using below CRON expression.
<img src="/posts/images/update-cosmosdb-ru-4.jpg">



Happy cloud computing.