# Accessing Data in ADLS
After data is in ADLS, external tables can be created to point to that data. ADX will be used to query the data lake and return extracted security value back to Sentinel.

### Components
* [Introduction](#Introduction)
   * [What is ADX](#What-is-ADX)
   * [What are External Tables?](#What-are-External-Tables)
   * [Why External Tables](#Why-External-Tables)
   * [Pre-requisites](#Pre-requisites)
* [Setup](#Setup)

  
#### Microsoft documentation:
* [What is ADX?](https://learn.microsoft.com/en-us/azure/data-explorer/data-explorer-overview)
* [External Table Overview](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/schema-entities/externaltables)
* [Setup External Tables](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake)

## Introduction

### What is ADX?
Azure Data Explorer (ADX) is a "fully managed, high-performance, big data analytics platform that makes it easy to analyze high volumes of data in near real time" [source](https://learn.microsoft.com/en-us/azure/data-explorer/data-explorer-overview). ADX is used to access data in the data lake.

### What are External Tables?
External tables in ADX are pointers to data stored in Azure Storage or SQL Server tables. This is in contrast to regular tables which are hosted in ADX itself. 
External tables are defined by a table name, schema, partitions, path format, data format, and connection string.

### Why External Tables
Tables in ADX are ingested and accessed through a compute frontend. This ties any interaction with that data to ADX compute. By using external tables we decouple storage and compute. This allows access to your data with your selection of compute resource, and only paying for compute when needed, dramatically lowering ingestion cost.

### Pre-requisites:
* Logs in ADLS Gen 2 storage account in one of the ADX [supported formats](https://learn.microsoft.com/en-us/azure/data-explorer/ingestion-supported-formats) (preferrably parquet)
* An ADX Cluster and Database [stood up](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-and-database?tabs=free)
* Database Admin Permissions on the ADX database

### Information to Gather:
* Storage account name
* Storage account key (used once for inferring schema)
* Container path format (ie: 2023-02-12/01/24: {yyyy}-{MM}-{dd}/{HH}/{mm})
* Storage account owner (to grant ADX access to the Storage account)


## Setup
### Granting ADX Access to ADLS
ADX will be using a [managed identity to access ADLS](https://learn.microsoft.com/en-us/azure/data-explorer/external-tables-managed-identities?tabs=system-assigned%2Cazure-storage#1---configure-a-managed-identity-for-use-with-external-tables). In order to do this:
1. Open the ADX database
2. Run the following command to enable managed identity usage for external tables
```kql
   .alter-merge cluster policy managed_identity ```[
    {
      "ObjectId": "system",
      "AllowedUsages": "ExternalTable"
    }
]```
```
3. Grant the ADX identity permissions to ADLS
   * Storage Blob Data Reader

   
### Inferring Log Schema
External tables in ADX require a schema. The schema defines the column names and their data types. Instead of manually extracting this information, we use the [infer_storage_schema plugin](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/inferstorageschemaplugin) in ADX to infer the schema. After doing so, review the results to ensure the schema is accurate.

1. Open the ADX database
2. Run the following command (if the command takes too long you can replace the connecetion string h@'...' with the path to a subset of the logs).
```kql
let options = dynamic({
  'StorageContainers': [
  h@'https://<Storage Account Name>.blob.core.windows.net/<Container>/<Optional Path to Subset of Logs>;<Storage Account Key>'
  ],
  'FileExtension': '<File Extension>',
  'Kind': 'storage',
  'partition': '(MinuteBin:datetime = bin(<Time Field Name in Logs>, 1m))',
  'pathformat': '(datetime_pattern("<Container Path Format>",MinuteBin))',
  'DataFormat': '<Log Format>'
});
evaluate infer_storage_schema(options)
```
3. Verify the accuracy of the output schema
4. Copy the value of the output

#### Example:
<img width="1000" alt="image" src="https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/75258742/01e5613b-a18e-45d7-a46a-9cd96b952c5d">
<img width="500" alt="Schema Output" src="https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/75258742/c31cebcd-29c2-4164-9a0a-5246c02798de">

### Creating the External Table
Now the external table can be created with the schema and managed identity. We are using a virtual column (IngestTime) to partition the table. This adds a new column to the logs that reads the datetime from the container format and optimizes filtering by that column. To learn more about partitioning read [here](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake#partitions-formatting).
1. Open the ADX database
2. Run the following command
```kql
.create external table <External Table Name>(<Schema Output>)
kind=storage
partition by (IngestTime:datetime)
pathformat=(datetime_pattern('<Path Format>',IngestTime))
dataformat=<Log Format>
(
  h@'https://<Storage Account Name>.blob.core.windows.net/<Container>;managed_identity=system'
)
```
#### Example:
<img width="1000" alt="image" src="https://github.com/seyed-nouraie/Azure-Security-Data-Lake/assets/75258742/9b0d3307-5d25-4244-951d-151171a48b2e">


### Querying the External Table
```kql
external_table("<External Table Name>")
 | where IngestTime between (ago(1h) .. now())
```

