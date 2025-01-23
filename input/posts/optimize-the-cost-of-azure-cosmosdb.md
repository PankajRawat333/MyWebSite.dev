Title: Optimize the cost of Azure Cosmos DB
Lead: How to reduce Azure CosmosDB cost
Description: In this post, I'll share some of the best practices to reduce Azure CosmosDB cost.
Published: 26/01/2019
Image: /posts/images/optimize-cosmosdb-cost.jpg
PrimaryTag: azure
Tags:
  - azure
  - cosmosdb
  - database
  - cost
---
I used Azure Cosmos DB in many projects and heard common concern from application owner and stakeholder is **Cosmos DB is best NOSQL database but it's very costly!**

**I don't agree** with this line that is why I'm writing this article. If you need premium product with premium service, you need to pay extra cost. Cosmos DB is Azure manage PaaS service, PaaS service comes with higher cost ([What is PaaS](https://azure.microsoft.com/en-in/resources/cloud-computing-dictionary/what-is-paas/)). Cosmos DB come with many unique feature like offers multiple well-defined consistency models, multiple API, guarantees single-digit-millisecond read and write latencies at the 99th percentile, and guarantees 99.999 high availability in the world.

In this post, I'm sharing my experience, which might help you to reduce the cost of Azure Cosmos DB.

### Azure Cosmos DB emulator for Dev/Test

Azure Cosmos DB [Emulator](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator) is local downloadable version of Cosmos DB cloud. you can use emulator for local development and testing, you don't need to rely on Internet and Azure subscription. I see many developer uses Azure Cosmos DB for Dev/Test, Instead of Cosmos DB emulator on local system.

### Estimate RU/s and Cost of reads and writes

Azure Cosmos DB guarantees predictable performance in terms of throughput and latency by using a provisioned throughput model. [Request unit in Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/request-units) is combination or measurement of CPU, Memory and IOPS. If you reserved high RU, mean you reserved more power, and you need to pay more money. It is very important to understand your database workload and [estimate throughput needs](https://docs.microsoft.com/en-us/azure/cosmos-db/request-units#estimating-throughput-needs). you can [calculate/estimate](https://www.documentdb.com/capacityplanner) RU with your sample data. The following table shows the cost of reads and writes in terms of RU/s for items that are 1 KB and 100 KB in size.
<img src="/posts/images/optimize-cosmosdb-cost1.jpg" class="img-fluid centered-img">
The read and write costs are applicable when using the default session consistency level. The considerations around RUs include: item size, property count, data consistency, indexed properties, indexing, and query patterns. Provisioning 1,000 RU/s translates to 3.6 million RU/hour and will cost $0.08 for the hour (in the US and Europe).

### Number of collections
Azure Cosmos DB bill may impact based on number of collections. If you've setup RU per collection wise, you need to pay money for each collection whether you're using that collection or not. Azure Cosmos DB Collection minimum limit is 400 RU. Before adding new collection, analysis your collection requirement and read/write frequency.

### RU sharing
Azure Cosmos DB offer reserve throughput per collection, where you need to understand your per collection workload and estimate RU usage. (Recently release) Now you can provision throughput collectively, for a set of collections as well, which allows those collections to share the provisioned throughput. This dramatically simplifies the capacity planning, since you can now configure provisioned throughput at different granularities based on your data model or application needs. Now you've the ability to control cost at different granularities in your application�s data model (e.g., database, collection, table, etc.). [Click here](https://azure.microsoft.com/en-us/blog/sharing-provisioned-throughput-across-multiple-containers-in-azure-cosmosdb/) for more detail.

### Scale Up/Down Reserve RU
Request Unit is collection setting which you can scale up and down as you needed. If your application do't have same workload by 24*7 a day or weekends, you can schedule scale up/down RU by programmatically or [Powershell](https://www.linkedin.com/pulse/update-cosmos-db-ru-using-powershell-azure-function-pankaj-rawat/?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3BFhrdt7HAR7iN7%2BJ7qjETgA%3D%3D) command. I see many applications having workload in standard business hours, scale down RU in night and weekends will help you to save money.

### Billing Model
Azure Cosmos DB billed based on reserved RU per hours. If you're scaling RU dynamically you need to understand Cosmos DB pricing calculation. Take a example, you reserved 1000 RU in collection. @10:45 AM you scale up RU from 1000 to 2000 and @11:15 AM scale down from 2000 to 1000. you will be charged for 2000 RU between 10 AM to 12 AM.

### Time to live (TTL)
With "time to live" or TTL, Azure Cosmos DB provides the ability to have documents automatically purged from the database after a period of time. I used this feature in my IoT application. we had hundreds of IoT devices which sends telematics data every minutes, we process that data and generates alerts. telematics data doesn't required for application after 7 days, hence we setup TTL 7 days. when data would purge using TTL setting, Azure Cosmos DB won't use any RU, mean you don't need to reserve any RU for data purging. Data archiving is best place where we can set TTL after data move in cold storage. This is solution is mostly used in IoT Lambda architecture).

### Don't scale RU programmatically (Automatic)
You can scale up/down RU programmatically in your code. I don't recommend this solution because it may increase your unexpected bill. In case of malicious activity (DDOS), having throttling is better at some point.

### Optimize Query
It is standard practices for all database. optimize query takes less CPU and memory, Same applicable with Azure cosmos DB, optimize query use less RU and you can reduce reserved RU, which help you to reduce Cosmos DB bill. You should consider following points to optimize query.
- Use partition if you've large amount of data.
- Choose right partition key so your data evenly distributed across all they partition.
- Use single partition key in get operation and avoid cross partition query.
- Create user define function (SQL API) to increase your SQL query vocabulary.
- Check query RU usage using Rest API, SDK, or metrics, this will help you to optimize query

### Cosmos DB Change Feed
I see many application run frequent query on database to get updated data. Cosmos DB change feed can help on this case. Change feed listen to an Azure Cosmos DB container for any changes (Add/Update). It then outputs the sorted list of documents that were changed in the order in which they were modified. The changes are persisted, can be processed asynchronously and incrementally, and the output can be distributed across one or more consumers for parallel processing. Note change feed consume RU on every trigger.
<img src="/posts/images/optimize-cosmosdb-cost2.jpg" class="img-fluid centered-img">

### Storage Cost
Azure Cosmos DB is billed with the unit of GBs. The cost for storage in Azure Cosmos DB is $0.25 GB/month, see Pricing. The total storage used is equal to the storage required by the data and indexes used across all the regions where you are using Azure Cosmos DB. If you globally replicate an Azure Cosmos account across three regions, you will pay for the total storage cost in each of those three regions.
- Move less frequent access data in cold storage (Blob, StorageTable etc). Data archived is widely used solution in relation and non-relational database.
<img src="/posts/images/optimize-cosmosdb-cost3.jpg" class="img-fluid centered-img">
- By default, Cosmos DB data is automatically indexed, which can increase the total storage cost. you can see [right] my current application has 10% storage size occupied only by indexed. However, you can apply custom index policies to reduce this overhead.
<img src="/posts/images/optimize-cosmosdb-cost4.jpg" class="img-fluid centered-img">

### Reserved Capacity
Azure Cosmos DB reserved capacity can significantly reduce your Cosmos DB costs�up to 65% on regular prices with a one-year or three-year upfront commitment. Reserved capacity provides a billing discount and doesn't affect the runtime state of your Azure Cosmos DB resources. [Check](https://docs.microsoft.com/en-us/azure/cosmos-db/cosmos-db-reserved-capacity) complete detail of reserved capacity in Azure Cosmos DB.

### Conclusion
Azure Cosmos DB is premium product with premium price, I recommend you do your due diligence to evaluate Cosmos DB and other available database.


Happy cloud computing.