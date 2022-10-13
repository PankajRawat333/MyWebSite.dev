Title: Maintain state in Azure Stream Analytics
Lead: Maintain Custom State in Azure Stream Analytics using Blob
Description: Maintain Custom State in Azure Stream Analytics using Blob
Published: 11/03/2019
Image: /posts/images/azure-stream-analytics.jpg
Tags:
  - azure
  - stream-analytics
  - realtime
  - azure-blob
---

Azure Stream Analytics is easy to get started. It only takes a few clicks to connect to multiple sources, sinks and to create an end-to-end pipeline, and is based on temporal window. Stream analytics maintain state based on window you are using in your ASA query. If you want to maintain some custom state based on your window result, I don't see any out of box solution.

### Problem

I have a scenario where I'm getting temperature data from various devices and I need to generate an alert if temperature breaches the threshold limit.

Alert should be generated in the following pattern

- Device First alert should generate if temperature continuously breach in 5 minute of window.
- Device second, third and further alert should generate after 10-10 minutes of interval.

**Example** - Device (ABC00099201) start temperature breach at 12:15 PM till 1 PM. alerts should be generated in the following order

<img src="/posts/images/azure-stream-analytics2.jpg">

### Solution

There might be many solutions to this problem, here I'm describing my current approach to solve this problem (as beginner :) of Azure Stream analytics). If you have any other solution please add in the comment section.

### Architecture

Below is my architecture diagram to solve this problem

<img src="/posts/images/azure-stream-analytics1.jpg">

Azure Stream Analytics **Input**

- Add temperature telematic data from Azure Event hub (JSON)
- Add list of devices with temperature threshold (dynamic reference data with date and time e.g.{date}/{time}/DeviceThersholdLimit.csv).
- Add last 10 minutes alerts as dynamic reference data (dynamic reference data with date and time e.g.{date}/{time}/temperature-alerts.csv).

Azure Stream Analytics **Output**

- Add Azure service bus queue as Output data source

Azure Stream Analytics [Query](https://github.com/PankajRawat333/TemperatureAlert/blob/master/TemperatureAlertQuery.txt)

Azure Stream analytics default **compatibility level** is 1.0 having some [limitation](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-compatibility-level), so use level 1.1

**Azure Function**

- Azure function listen to service bus queue for temperature breach alert, It will read message from queue and insert into Cosmos DB.
- Function gets last 10 minutes alerts from DB, after grouping data by deviceId, convert into csv file and upload into blob (with incremented date time format), so ASA query can use this file as reference data.
- Azure stream analytics keep reference data in-memory and update after one minute of interval.

**Limitation**

I tried this POC only for one device, for multiple devices required some logic changes in blob creation (dynamic reference data). After this POC, I got some more feedback and minor changes in requirement, soon I will post a new article with updated requirement.

### References

[Github repository for code and sample data files used in POC](https://github.com/PankajRawat333/TemperatureAlert)

[What is Azure stream analytics and why?](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-introduction)

[Enable real-time hot path analytics and machine learning models in the cloud and on the intelligent edge with Azure Stream Analytics](https://myignite.techcommunity.microsoft.com/sessions/65358?source=sessions)

[Run stream analytics on-premise or your own cluster using Microsoft Trill](https://www.microsoft.com/en-us/research/project/trill/)

Happy cloud computing.