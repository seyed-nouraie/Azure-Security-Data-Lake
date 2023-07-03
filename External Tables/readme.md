# How To Setup External Tables in ADX

After data is in ADLS, external tables can be created to point to that data. For official documentation on external tables refer to 
* [MS setup Docs](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake)
* [External Table Docs](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/schema-entities/externaltables)
  
## What is an External Table?
External tables in ADX are pointers to data stored in Azure Storage or SQL Server tables. This is in contrast to regular tables which are hosted in ADX itself. 
External tables are defined by a table name, schema, partitions, path format, data format, and connection string.

## Why Use External Tables
Tables in ADX are ingested and accessed through a compute frontend. This ties any interaction with that data to ADX compute. By using external tables we decouple storage and compute. This allows access to your data with your selection of compute resource, and only paying for compute when needed, dramatically lowering ingestion cost.

## Inferring Log Schema
External tables in ADX require a schema. The schema defines the column names and their data types. Instead of manually extracting this information, we use the [infer_storage_schema plugin](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/inferstorageschemaplugin) in ADX to infer the schema. After doing so, review the column names and types to ensure the schema is accurate.
