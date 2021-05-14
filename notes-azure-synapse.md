# Notes: Azure Synapse

### Quickstart tutorial 

{% embed url="https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-create-workspace" %}

{% embed url="https://docs.microsoft.com/en-us/azure/data-factory/data-factory-ux-troubleshoot-guide\#third-party-cookies-blocked" %}

Notes:

What is abfss ? \([https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-abfs-driver](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-abfs-driver)\)

* `https://contosolake.dfs.core.windows.net/users/NYCTripSmall.parquet`
* `abfss://users@contosolake.dfs.core.windows.net/NYCTripSmall.parquet`



{% embed url="https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-sql-on-demand" %}



{% embed url="https://docs.microsoft.com/en-us/azure/data-factory/connector-azure-data-lake-storage" %}

### Concepts

{% embed url="https://docs.microsoft.com/en-us/azure/azure-sql/database/logical-servers" %}



### Username and Password

DATA\_SOURCE = 'TabLake',

data\_explorer WITH PASSWORD = 'My Very Strong Password that I like to use all the time, everywhere';

DataExplorationDB

![](.gitbook/assets/image%20%2894%29.png)

### Tested Connecting with OAuth

Basic Admin setup:

{% embed url="https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-add-admin" %}

Best Practice:

{% embed url="https://docs.microsoft.com/en-us/azure/synapse-analytics/security/how-to-set-up-access-control" %}



{% embed url="https://kb.tableau.com/articles/issue/Error-Authentication-failed-When-Connecting-to-Azure-Data-Lake-Storage-Gen2" %}

![](.gitbook/assets/image%20%2895%29.png)

![](.gitbook/assets/image%20%28100%29.png)

![](.gitbook/assets/image%20%2899%29.png)

{% embed url="https://www.thorogood.com/webcasts/2020-07-30-tableau-and-azure/" %}

{% embed url="https://docs.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure?tabs=azure-powershell" %}

[https://www.sqlshack.com/sql-on-demand-pools-in-azure-synapse-analytics/](https://www.sqlshack.com/sql-on-demand-pools-in-azure-synapse-analytics/)

