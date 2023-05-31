# Azure Security Data Lake
Today's security analysts require comprehensive visibility and an analytics framework to enhance their ability to identify and defend against threat actors in the ever-expanding security landscape. To be effective, the SIEM requires a collaborative partner to enhance its correlation, detection, and automation capabilities. This solution we will demonstrate how to implement an enterprise cloud-based scalable data lake and Azure analytics solution which can provide better visibility into various log data sources (such as SaaS, PaaS, and IaaS) without incurring excessive costs, while also meeting compliance and long-term retention requirements.

This solution utilizes cloud data storage, automation, and analytics to provide the following improvements:

- It addresses the shared data needs of compliance, infrastructure, and security domains.
- It proactively prepares data for advanced analytics, such as artificial intelligence engines and models.
- It expands the analytics capabilities, allowing for the analysis of Indicators of Attack (IOA) and Indicators of Compromise (IOC) to enhance security detection and correlation.
- The solution focuses on collecting and analyzing network, endpoint, communication, and identity data in a central data lake for effective analytics and investigations.

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
