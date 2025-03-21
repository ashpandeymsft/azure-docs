---
title: Log data ingestion time in Azure Monitor | Microsoft Docs
description: Explains the different factors that affect latency in collecting log data in Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.reviewer: eternovsky
ms.date: 03/21/2022

---

# Log data ingestion time in Azure Monitor
Azure Monitor is a high scale data service that serves thousands of customers sending terabytes of data each month at a growing pace. There are often questions about the time it takes for log data to become available after it's collected. This article explains the different factors that affect this latency.

## Typical latency
Latency refers to the time that data is created on the monitored system and the time that it comes available for analysis in Azure Monitor. The typical latency to ingest log data is **between 20 seconds and 3 minutes**. However, the specific latency for any particular data will vary depending on a variety of factors explained below.


## Factors affecting latency
The total ingestion time for a particular set of data can be broken down into the following high-level areas. 

- **Agent time** - The time to discover an event, collect it, and then send it to Azure Monitor Logs ingestion point as a log record. In most cases, this process is handled by an agent. Additional latency may be introduced by the network.
- **Pipeline time** - The time for the ingestion pipeline to process the log record. This includes parsing the properties of the event and potentially adding calculated information.
- **Indexing time** - The time spent to ingest a log record into Azure Monitor big data store.

Details on the different latency introduced in this process are described below.

### Agent collection latency

**Time varies**

Agents and management solutions use different strategies to collect data from a virtual machine, which may affect the latency. Some specific examples include the following:

| Type of data  | Collection frequency  | Notes |
|:--------------|:----------------------|:------|
| Windows events, syslog events, and performance metrics | collected immediately| | 
| Linux performance counters | polled at 30-second intervals| |
| IIS logs and text logs | collected once their timestamp changes | For IIS logs, this is influenced by the [rollover schedule configured on IIS](../agents/data-sources-iis-logs.md). |
| Active Directory Replication solution | Assessment every five days | The agent collects these logs only when assessment is complete.|
| Active Directory Assessment solution | weekly assessment of your Active Directory infrastructure | The agent collects these logs only when assessment is complete.|

### Agent upload frequency

**Under 1 minute**

To ensure the Log Analytics agent is lightweight, the agent buffers logs and periodically uploads them to Azure Monitor. Upload frequency varies between 30 seconds and 2 minutes depending on the type of data. Most data is uploaded in under 1 minute. 

### Network

**Varies**

Network conditions may negatively affect the latency of this data to reach Azure Monitor Logs ingestion point.

### Azure metrics, resource logs, activity log

**30 seconds to 15 minutes**

Azure data adds additional time to become available at Azure Monitor Logs ingestion point for processing:

- **Azure platform metrics** are available in under a minute in the metrics database, but take additional 3 minutes to be exported to the Azure Monitor Logs ingestion point.
- **Resource logs** typically add 30-90 seconds, depending on the Azure service. Some Azure services (specifically, Azure SQL Database and Azure Virtual Network) currently report their logs at 5 min intervals. Work is in progress to improve this further. See the [query below](#checking-ingestion-time) to examine this latency in your environment
- **Activity log** data is ingested in 30 seconds when you use the recommended subscription-level diagnostic settings to send them into Azure Monitor Logs. However, they may take 10-15 minutes if you instead use the legacy integration.  

### Management solutions collection

**Varies**

Some solutions do not collect their data from an agent and may use a collection method that introduces additional latency. Some solutions collect data at regular intervals without attempting near-real time collection. Specific examples include the following:

- Microsoft 365 solution polls activity logs using the Management Activity API, which currently does not provide any near-real time latency guarantees.
- Windows Analytics solutions (Update Compliance for example) data is collected by the solution at a daily frequency.

Refer to the [documentation for each solution](../insights/solutions.md) to determine its collection frequency.

### Pipeline-process time

**30 to 60 seconds**

Once available at ingestion point, data takes additional 30-60 seconds to be available for querying.

Once log records are ingested into the Azure Monitor pipeline (as identified in the [_TimeReceived](./log-standard-columns.md#_timereceived) property), they're written to temporary storage to ensure tenant isolation and to make sure that data isn't lost. This process typically adds 5-15 seconds. Some management solutions implement heavier algorithms to aggregate data and derive insights as data is streaming in. For example, the Network Performance Monitoring aggregates incoming data over 3-minute intervals, effectively adding 3-minute latency. Another process that adds latency is the process that handles custom logs. In some cases, this process might add few minutes of latency to logs that are collected from files by the agent.

### New custom data types provisioning

When a new type of custom data is created from a [custom log](../agents/data-sources-custom-logs.md) or the [Data Collector API](../logs/data-collector-api.md), the system creates a dedicated storage container. This is a one-time overhead that occurs only on the first appearance of this data type.

### Surge protection

**Typically less than 1 minute but can be more** 

The top priority of Azure Monitor is to ensure that no customer data is lost, so the system has built-in protection for data surges. This includes buffers to ensure that even under immense load, the system will keep functioning. Under normal load, these controls add less than a minute, but in extreme conditions and failures they could add significant time while ensuring data is safe.

### Indexing time

**5 minutes or less**

There is a built-in balance for every big data platform between providing analytics and advanced search capabilities as opposed to providing immediate access to the data. Azure Monitor allows you to run powerful queries on billions of records and get results within a few seconds. This is made possible because the infrastructure transforms the data dramatically during its ingestion and stores it in unique compact structures. The system buffers the data until enough of it is available to create these structures. This must be completed before the log record appears in search results.

This process currently takes about 5 minutes when there is low volume of data but less time at higher data rates. This seems counterintuitive, but this process allows optimization of latency for high-volume production workloads.

## Checking ingestion time
Ingestion time may vary for different resources under different circumstances. You can use log queries to identify specific behavior of your environment. The following table specifies how you can determine the different times for a record as it's created and sent to Azure Monitor.

| Step | Property or Function | Comments |
|:---|:---|:---|
| Record created at data source | [TimeGenerated](./log-standard-columns.md#timegenerated) <br>If the data source doesn't set this value, then it will be set to the same time as _TimeReceived. | If at processing time, the Time Generated value is older than 3 days the row will be dropped. |
| Record received by Azure Monitor ingestion endpoint | [_TimeReceived](./log-standard-columns.md#_timereceived) | This field is not optimized for mass processing and should not be used to filter large datasets. |
| Record stored in workspace and available for queries | [ingestion_time()](/azure/kusto/query/ingestiontimefunction) | It is recommended to use ingestion_time() if there is a need to filter only records that were ingested in a certain time window. In such case, it is recommended to add also TimeGenerated filter with a larger range. |

### Ingestion latency delays
You can measure the latency of a specific record by comparing the result of the [ingestion_time()](/azure/kusto/query/ingestiontimefunction) function to the _TimeGenerated_ property. This data can be used with various aggregations to find how ingestion latency behaves. Examine some percentile of the ingestion time to get insights for large amount of data. 

For example, the following query will show you which computers had the highest ingestion time over the prior 8 hours: 

``` Kusto
Heartbeat
| where TimeGenerated > ago(8h) 
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated 
| extend AgentLatency = _TimeReceived - TimeGenerated 
| summarize percentiles(E2EIngestionLatency,50,95), percentiles(AgentLatency,50,95) by Computer 
| top 20 by percentile_E2EIngestionLatency_95 desc
```

The preceding percentile checks are good for finding general trends in latency. To identify a short-term spike in latency, using the maximum (`max()`) might be more effective.

If you want to drill down on the ingestion time for a specific computer over a period of time, use the following query, which also visualizes the data from the past day in a graph: 


``` Kusto
Heartbeat 
| where TimeGenerated > ago(24h) //and Computer == "ContosoWeb2-Linux"  
| extend E2EIngestionLatencyMin = todouble(datetime_diff("Second",ingestion_time(),TimeGenerated))/60 
| extend AgentLatencyMin = todouble(datetime_diff("Second",_TimeReceived,TimeGenerated))/60 
| summarize percentiles(E2EIngestionLatencyMin,50,95), percentiles(AgentLatencyMin,50,95) by bin(TimeGenerated,30m) 
| render timechart
```
 
Use the following query to show computer ingestion time by the country/region they are located in which is based on their IP address: 

``` Kusto
Heartbeat 
| where TimeGenerated > ago(8h) 
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated 
| extend AgentLatency = _TimeReceived - TimeGenerated 
| summarize percentiles(E2EIngestionLatency,50,95),percentiles(AgentLatency,50,95) by RemoteIPCountry 
```
 
Different data types originating from the agent might have different ingestion latency time, so the previous queries could be used with other types. Use the following query to examine the ingestion time of various Azure services: 

``` Kusto
AzureDiagnostics 
| where TimeGenerated > ago(8h) 
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated 
| extend AgentLatency = _TimeReceived - TimeGenerated 
| summarize percentiles(E2EIngestionLatency,50,95), percentiles(AgentLatency,50,95) by ResourceProvider
```

### Resources that stop responding 
In some cases, a resource could stop sending data. To understand if a resource is sending data or not, look at its most recent record which can be identified by the standard _TimeGenerated_ field.  

Use the _Heartbeat_ table to check the availability of a VM, since a heartbeat is sent once a minute by the agent. Use the following query to list the active computers that haven’t reported heartbeat recently: 

``` Kusto
Heartbeat  
| where TimeGenerated > ago(1d) //show only VMs that were active in the last day 
| summarize NoHeartbeatPeriod = now() - max(TimeGenerated) by Computer  
| top 20 by NoHeartbeatPeriod desc 
```

## Next steps
* Read the [Service Level Agreement (SLA)](https://azure.microsoft.com/support/legal/sla/monitor/v1_3/) for Azure Monitor.
