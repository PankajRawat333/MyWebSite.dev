Title: What is the reason of your Azure WebJobs stop/restart?
Description: Use Azure activity log to find Start/Stop reason in Azure WebJobs.
Lead: Start/Stop Continuous Azure WebJobs activity log
Published: 30/07/2018
Image: /posts/images/webjobs.jpg
PrimaryTag: azure
Tags:
  - azure
  - webjobs
  - activitylog
---
In this post, I shared how to use Azure activity log to find Azure WebJobs stop/restart problem.
## Context

Recently I was working in project where we were processing thousands of events per second using Azure WebJobs. I received a complaint from customer that data processing was stopped for few seconds but now processing.
What was the problem?
To find that problem, I directly jump into Azure WebJobs logs and found the reason of delay event.
```
[07/30/2018 08:40:45 > b842d7: SYS INFO] WebJob is stopping due to website shutting down
[07/30/2018 08:40:45 > b842d7: SYS INFO] Status changed to Stopping
[07/30/2018 08:40:50 > b842d7: SYS INFO] WebJob process was aborted
[07/30/2018 08:40:50 > b842d7: SYS INFO] Status changed to Stopped
[07/30/2018 08:41:12 > b842d7: SYS INFO] Status changed to Starting
```

After looking application diagnostic logs, I got my answer but I was curious to know that why WebJob has restarted and who did? To find this answer is not an easy task but Azure team did very good work and made it very simple.

Open Tab > Diagnose and solve problems > Web App Restarted

<img src="/posts/images/webjobs-diagnose.jpg">
<img src="/posts/images/webjobs-diagnose-detail.jpg">
Using activity log we can find out user name as well
<img src="/posts/images/webjobs-activitylog.jpg">

Happy cloud computing.