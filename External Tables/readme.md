# How To Setup External Tables in ADX

After data is in ADLS, external tables can be created to point to that data. 
Microsoft documentation:
* [What is ADX?](https://learn.microsoft.com/en-us/azure/data-explorer/data-explorer-overview)
* [External Table Overview](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/schema-entities/externaltables)
* [Setup External Tables](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake)

Pre-requisites:
* Logs in ADLS Gen 2 storage account in one of the ADX [supported formats](https://learn.microsoft.com/en-us/azure/data-explorer/ingestion-supported-formats) (preferrably parquet)
* Storage account name
* Storage account key (used once for inferring schema)
* Container path format (ie: 2023-02-12/01/24 = {yyyy}-{MM}-{dd}/{HH}/{mm})
* Storage account owner (to grant ADX access to the Storage account)
* An ADX Cluster and Database [stood up](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-and-database?tabs=free)
* Database Admin Permissions on the ADX database

## What is ADX?
Azure Data Explorer (ADX) is a "fully managed, high-performance, big data analytics platform that makes it easy to analyze high volumes of data in near real time" [source](https://learn.microsoft.com/en-us/azure/data-explorer/data-explorer-overview). ADX is used to access data in the data lake.

## What is an External Table?
External tables in ADX are pointers to data stored in Azure Storage or SQL Server tables. This is in contrast to regular tables which are hosted in ADX itself. 
External tables are defined by a table name, schema, partitions, path format, data format, and connection string.

## Why Use External Tables
Tables in ADX are ingested and accessed through a compute frontend. This ties any interaction with that data to ADX compute. By using external tables we decouple storage and compute. This allows access to your data with your selection of compute resource, and only paying for compute when needed, dramatically lowering ingestion cost.

## Granting ADX Access to ADLS
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
3. Grant ADX permissions to ADLS (storage blob data reader)

   
## Inferring Log Schema
External tables in ADX require a schema. The schema defines the column names and their data types. Instead of manually extracting this information, we use the [infer_storage_schema plugin](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/inferstorageschemaplugin) in ADX to infer the schema. After doing so, review the results to ensure the schema is accurate.

1. Open the ADX database
2. Run the following command 
```kql
let options = dynamic({
  'StorageContainers': [
  h@'https://<Storage Account Name>.blob.core.windows.net/<Container>;<Storage Account Key>'
  ],
  'FileExtension': '<File Extension>',
  'Kind': 'blob',
  'partition': '(MinuteBin:datetime = bin(<Time Field Name in Logs>, 1m))',
  'pathformat': '(datetime_pattern("<Container Path Format>",MinuteBin))',
  'DataFormat': '<Log Format>'
});
evaluate infer_storage_schema(options)
```


