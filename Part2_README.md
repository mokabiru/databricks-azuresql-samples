## Solution Part 2
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/Solution%20Architecture%20Numbered%20.jpg)

4. Common data dimension tables and the staging table (in Step 3) from Azure SQL Managed Instance are read into dataframes in Azure Databricks.
> The two Managed Instances shown in the “Store” and the “Serve” layer are essentially the same instance just depicted in different phases of the data flow. In a real-world, Azure SQL Managed Instance or Azure SQL Databases can play the role of both a data storage service and data serving service for consuming applications / data visualization tools.
5. The dataframes containing the necessary dimension and staging data are further refined, joined and transformed to produce a denormalized fact table for reporting.
6. The resulting denormalized data table is written to Azure SQL Managed Instance ready to serve the data consumers.

Notebook 2: [COVID19-Load-SQLManagedInstance](https://github.com/mokabiru/databrickssqlmi/blob/master/DatabricksNotebooks/COVID19-Load-SQLManagedInstance.dbc)

The objective of this Scala notebook is to demonstrate the usage of Azure SQL Database Spark connector to read and write data from Azure SQL Managed Instance while leveraging Spark in Azure Databricks to perform data transformation (Steps 4-6 in the architecture diagram above).

In this solution, Azure SQL Managed Instance [Business Critical tier](https://docs.microsoft.com/en-us/azure/azure-sql/managed-instance/sql-managed-instance-paas-overview#business-critical-service-tier) is used which provides a readable secondary by default. The solution works just as good with the [General Purpose tier](https://docs.microsoft.com/en-us/azure/azure-sql/managed-instance/sql-managed-instance-paas-overview#general-purpose-service-tier) except that there is not readable secondary. Public endpoint for the SQL MI is [securely enabled](https://docs.microsoft.com/en-us/azure/azure-sql/managed-instance/public-endpoint-overview) to allow Azure Databricks to connect to the instance. If there are security restrictions in your organization that prevents the usage of public endpoint, the solution will require [Azure Databricks to be injected into a Vet](https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject) and use one of the [connectivity architectures](https://docs.microsoft.com/en-us/azure/azure-sql/managed-instance/connect-application-instance?view=sql-server-2017) either using [VNet Peering](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview) or [VNet-to-VNet VPN Gateway](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal) to establish connectivity between Azure Databricks and SQL MI.

The following modules are required to read and write data from Azure SQL Managed Instance and hence imported at the start of the notebook.

**To read data from SQL MI:**
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/importlibrariesread.png)

**To write data to SQL MI:**
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/importlibrarieswrite.png)

**Retrieve credentials for SQL MI connection:**
Similar to Part 1 of the solution, the connection details and credentials stored as Azure Key Vault scoped secrets are retrieved in the notebook. Note the secrets will always be `[REDACTED]` in the notebook and will never be visible in plain text.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/secretssqlmi2.png)

**Read tables from SQL MI:**
In order to take advantage of the readable secondary available in Business Critical tier, data can be read from the secondary by specifying the `ApplicationIntent` property of the connection as shown below. Note that `Port 3342` is specified to connect using SQL MI’s public endpoint.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/readsqlmi2.png)

**Data Transformation:**
Data from the tables `dbo.DimCountry` and `dbo.StagingPredictCovid19` is read into dataframes and joined together to produce a denormalized dataset that is recommended for data visualization and reporting tools.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/dfjoin.png)

The joined dataframe is then finalized by dropping unimportant columns, renaming columns and concatenating date part columns to produce a `datetime` column before writing the dataset to SQL MI.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/dfcleanup.png)

**Write results to SQL MI:**
The finalized dataset is written to the fact table `dbo.FactCovid19` in SQL MI using the `Append` mode. Note that the mode can be changed to `Overwrite` based on your requirement.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/sqlmiwrite.png)

The records are now available in dbo.FactCovid19 in SQL MI
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/ssmsfactexplorer.png)
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/ssmsfactselect.png)

The fact data can be visualized by reporting or data visualization tools like [Power BI](https://powerbi.microsoft.com/en-us/), consumed by APIs in downstream applications or even visualized in the same Databricks notebook as shown below.

**Data visualization:**
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/map.png)
