# Azure Security Data Lake
Your platform for extracting and shipping security value from your data lake to Sentinel. 

Security thrives when you don't have to choose between dropping your logs and dropping your budget. Large models make your data more valuable, and your insights more reachable than ever before. Ingest and store your data into a data lake at a fraction of the cost of the SIEM, while unlocking security insights that were complete gaps prior.

This solution is designed to sift through your lake and find security gold. We do this by summarizing the data, and only returning the summarized data to Sentinel. Your SIEM now has highly concentrated information which it can use for OoB Rules (functions), and workbooks for quick visualization and hunting (Workbooks). 

### Components
* [Summarization](#Summarization)
* [Enrichment](#Enrichment)
* [Functions](#Functions)
* [Workbooks](#Workbooks)
* [Deployment](#Deployment)


### Summarization
Many detection use cases don't require each individual log record. This summarization runs an hourly playbook against ADX that aggregates entity data from an external table in ADLS. Doing so, only the summarized data is ingested into compute ready (Sentinel) storage while the raw data is kept in ADLS for ad hoc searches or as other analytics use cases are developed. 

### Enrichment
After an incident triggers on summarized data, the next step might be to query the raw logs. We have automated this through enrichment playbooks. These playbooks trigger on an incident and pull contextual logs from ADLS back into the incident to help direct the SOC.

### Functions
Summarized entities are ingested into custom tables, but can utilize the power of Out of the Box content using ASIM functions. This solution provides ASIM functions which integrate the network summary tables to the ASIM and IM Network session functions. 

### Workbooks
The workbook is intended to be the one stop for monitoring trends, anomalies, and the activity of your summarized data. The workbook provides top entities and allows you to pivot and correlate from firewall to endpoint log sources, putting together a full chain of events.

### Deployment
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fseyed-nouraie%2FAzure-Security-Data-Lake%2Fmain%2FDeploy%2Fazuredeploy.json)
