# Network Summarization Logic Apps

This sample logic app summarizes data from the data lake and sends it to custom tables in Sentinel. The logic app requires data in the lake along with external tables in an ADX cluster.  
Below we break down the compontents of the logic app. It is important to note that the queries may be different from your environment due to variations in logging sources. Make sure the query logic aligns with the naming in your environment (field names, table names).

### Contents
* [Trigger and Variables](#Trigger-and-Variables)
* [Size Estimate Query](#Size-Estimate-Query)
* [Summarization Query](#Summarization-Query)
<br><br>

## Trigger and Variables
<img width="800" alt="image" src="https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/75258742/54e67794-1bf4-4f84-8cfc-eb4c79a45b4f">
This logic app runs hourly by default. The recurrence can be changed to fit your SOC needs, more frequent runs reduce the time it takes to see data in Sentinel at the expense of ingesting more data.

The logic app initializes two buckets of variables. The first variable is current time, which is used in the queries run on top of the data lake. This is initialized at the beginning and not at each search to ensure every action queries the same time window. The second set of variables are used for row tracking. ADX limits query results to 50k rows, and we iterate through 30k rows at a time to avoid this limit (more info below).


## Size Estimate Query
<img width="400" alt="image" src="https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/75258742/786b3774-cbfc-4899-a4e2-e93adfc0476d">
<img width="400" alt="image" src="https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/75258742/ee755751-9e3e-4da8-9ddb-02e1e20c91f3">

This query estimates the size of the original, unsummarized data using the estimate_data_size kusto function and sends it to a custom Sentinel table. This is used to demonstrate size saving value of summarization.

## Summarization Query
<img width="300" alt="image" src="https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/75258742/c8898389-0b19-4de8-9bfc-e5ff035f83d8">

These for-loops summarize network traffic from the data lake, sending 30k rows at a time.

<img width="300" alt="image" src="https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/75258742/4dc3bccb-548b-43da-9e46-21237c4f0e9e">

The result is the total number of connections, remote ports, remote IPs, for a given IP/direction/firewall action per hour. This hour bucket can be adjusted based on the logic app trigger.
With the result of this summarization, we aim to provide enough information for Network IOC matching and high level network detections in Sentinel. After an incident is triggered the analyst can then pivot to the raw logs for triage.

