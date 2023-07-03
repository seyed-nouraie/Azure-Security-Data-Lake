# Azure Security Data Lake
[Skip to deployment](#Deployment)

Today's security analysts require comprehensive visibility and an analytics framework to enhance their ability to identify and defend against threat actors in the ever-expanding security landscape. To be effective, the SIEM requires a collaborative partner to enhance its correlation, detection, and automation capabilities. This solution we will demonstrate how to implement an enterprise cloud-based scalable data lake and Azure analytics solution which can provide better visibility into [various log data sources](https://github.com/seyed-nouraie/Azure-Security-Data-Lake/tree/main/DataSources) (such as SaaS, PaaS, and IaaS) without incurring excessive costs, while also meeting compliance and long-term retention requirements.

This solution utilizes cloud data storage, automation, and analytics to provide the following improvements:

- It addresses the shared data needs of compliance, infrastructure, and security domains.
- It proactively prepares data for advanced analytics, such as artificial intelligence engines and models.
- It expands the analytics capabilities, allowing for the analysis of Indicators of Attack (IOA) and Indicators of Compromise (IOC) to enhance security detection and correlation.
- The solution focuses on collecting and analyzing network, endpoint, communication, and identity data in a central data lake for effective analytics and investigations.

We will be utilizing Azure Storage Account Gen2, Logic Apps, Azure Data Explorer, and Azure Sentinel SIEM. Additionally, we will demonstrate how to use the data lake for investigating and enriching incidents, allowing security analysts to more rapidly investigate and resolve incidents.

![AzureDataLakeFlow01](https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/12141454/02adf71a-ff7e-4cef-b236-4d84474328c1)

1. [Extract, transform, and load](https://github.com/seyed-nouraie/Azure-Data-Lake-ETL/tree/main/Nifi) logging data into ADLS.  Transforms include structure formatting and parquet output.
2. Each logging entity resides in its own container with standard chronological organization.
3. External tables are created at the container level for each logging entity. The majority of compute-work is done by ADX. Internal tables are created for summary data.
4. Logic app performs entity summarization of hourly data lake data through ADX
     * Source and Destination IP
     * Bytes
     * Device action
5. Logic app sends summarized data to Sentinel table.
6. Sentinel table is normalized through ASIM Network Events Schema.
7. Many analytics rules, IOC lookups, workbooks, and searches can continue to directly refer to the ASIM normalized schema without modification.
8. On Incident trigger logic app runs to enrich incident with relevant data lake event information and OpenAI enrichment.
9. Connect to external or internal tables using direct query.  Dashboards and reports will show near-time updates as logs hit ADLS.

Follow our blog and video series at https://azurecloudai.blog/2023/06/21/azure-security-data-lake/

### Components
* [ADX External Tables](#ADX-External-Tables)
* [Summarization](#Summarization)
* [Enrichment](#Enrichment)
* [Functions](#Functions)
* [Workbooks](#Workbooks)
* [Deployment](#Deployment)


### ADX External Tables
External tables pointing to ADLS are created in Sentinel. Each top level container includes a different [log source](https://github.com/seyed-nouraie/Azure-Security-Data-Lake/tree/main/DataSources). These external tables decouple compute from storage, allowing ADX to only access the data as needed. These external tables are used for summarization and enrichmentn of incidents from Sentinel.

### Summarization
Many detection use cases don't require each individual log record. This summarization runs an hourly playbook against ADX that aggregates entity data from an external table in ADLS. Doing so, only the summarized data is ingested into compute ready (Sentinel) storage while the raw data is kept in ADLS for ad hoc searches or as other analytics use cases are developed. 

### Enrichment
After an incident triggers on summarized data, the next step might be to query the raw logs. We have automated this through enrichment playbooks. These playbooks trigger on an incident and pull contextual logs from ADLS back into the incident to help direct the SOC.

### Functions
Summarized entities are ingested into custom tables, but can utilize the power of Out of the Box content using ASIM functions. This solution provides ASIM functions which integrate the network summary tables to the ASIM and IM Network session functions. 

### Workbooks
The workbook is intended to be the one stop for monitoring trends, anomalies, and the activity of your summarized data. The workbook provides top entities and allows you to pivot and correlate from firewall to endpoint log sources, putting together a full chain of events.

### Deployment
1. [Set up ADLS](https://github.com/seyed-nouraie/Azure-Security-Data-Lake/tree/main/ADLS)
2. [Send Data to ADLS](https://github.com/seyed-nouraie/Azure-Data-Lake-ETL/tree/main/Nifi)
3. [Create External Tables](https://github.com/seyed-nouraie/Azure-Security-Data-Lake/tree/main/External%20Tables)
4. Deploy:
   
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fseyed-nouraie%2FAzure-Security-Data-Lake%2Fmain%2FDeploy%2Fazuredeploy.json)


### Glossary
* [ADX - Azure Data Explorer](https://learn.microsoft.com/en-us/azure/data-explorer/data-explorer-overview)
* [ADLS - Azure Data Lake Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)
* [ASIM - Advanced Security Information Model](https://learn.microsoft.com/en-us/azure/sentinel/normalization)
