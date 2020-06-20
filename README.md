The objective of this solution is to showcase the integration between Azure Databricks and Azure SQL Managed Instance to deliver insights and data visualizations using a publicly available COVID-19 dataset. In doing so, the solution uses the COVID-19 dataset and runs a machine learning model in Databricks to predict the fatalities which are then written into a datamart in Azure SQL Managed Instance for visualization and reporting.

> Note: Gradient Boosting Trees (GBT) Regression model has been used in
> the solution purely for demonstration purposes and by no means should
> this be considered as the only machine learning algorithm to run
> predictions on the dataset used.


**Azure SQL Spark Connector**

The [Spark connector](https://docs.microsoft.com/en-us/azure/azure-sql/database/spark-connector) enables databases in Azure SQL Database, Azure SQL Managed Instance, and SQL Server to act as the input data source or output data sink for Spark jobs. The Spark connector supports Azure Active Directory (Azure AD) authentication to connect to Azure SQL Database and Azure SQL Managed Instance and it utilizes the Microsoft JDBC Driver for SQL Server to move data between Spark worker nodes and databases.

The connector can either be downloaded from the [azure-sqldb-spark GitHub repo](https://github.com/Azure/azure-sqldb-spark/tree/master/releases/azure-sqldb-spark-1.0.0) or imported into Azure Databricks using the Maven coordinate: _com.microsoft.azure:azure-sqldb-spark:1.0.2_

**Solution Architecture**

The solution architecture shown below depicts the various phases of data flow along with a variety of data sources and data consumers involved.

![Solution Architecture Image](https://github.com/mokabiru/databrickssqlmi/raw/master/media/Solution%20Architecture%20Numbered%20.jpg)

_Fig.1_

The solution comprises of the following parts as described in the data flow below (the sequence numbers are highlighted in the architecture diagram Fig. 1 above)

 1. The solution extracts the COVID-19 public dataset available in a
    data lake (Azure Storage – Blob / ADLS Gen2) into Azure Databricks
    as a dataframe.
  2. The extracted COVID-19 dataset is cleaned, pre-processed, trained
    and scored using a Gradient Boosted Trees (GBT) Machine Learning
    model.

> *GBT is chosen to predict the deaths on a given day in a given country
> purely for   demonstration purposes only and should not
>     be considered as the only model for such prediction.*

3. The resulting dataset with the predicted scores is stored into a
staging table in Azure SQL Managed Instance for further downstream
transformation.
4. Common data dimension tables and the staging table (in Step 3) from
Azure SQL Managed Instance are read into dataframes in Azure
Databricks.

> The two Managed Instances shown in the “Store” and the “Serve” layer
> are essentially the same instance just depicted in different phases of
> the data flow. In a real-world, Azure SQL Managed Instance or Azure
> SQL Databases can play the role of both a data storage service and
> data serving service for consuming applications / data visualization
> tools.

5. The dataframes containing the necessary dimension and staging data
are further refined, joined and transformed to produce a
denormalised fact table for reporting.
6. The resulting denormalised data table is written to Azure SQL
Managed Instance ready to serve the data consumers.

**Azure Services:**
The solution requires the following Azure Services and their relevant SKUs.
1. **Azure Databricks Standard / Premium Workspace**
*Cluster Options: Databricks Runtime Version 6.5; Standard Mode; 2 Worker nodes and 1 Driver Node;
Node specification: Standard_DS3_v2*
2. **Azure SQL Managed Instance Business Critical 4vCores** (General Purpose can also be used). Business Critical is used for demonstration as it comes with a built-in Read Only replica that can be used for data reads / reporting.
3. **Azure Key Vault (Standard)** – to save secrets such as SQL Managed Instance login password

The solution is divided into two parts in two Azure Databricks notebooks.
Notebook 1 ([Covid19-DatabricksML-SQLMI](https://github.com/mokabiru/databrickssqlmi/blob/master/DatabricksNotebooks/Covid19-DatabricksML-SQLMI.dbc)) emphasizes on the machine learning model training and prediction that executes Steps 1-3 in the data flow described above.
Notebook 2 ([COVID19-Load-SQLManagedInstance](https://github.com/mokabiru/databrickssqlmi/blob/master/DatabricksNotebooks/COVID19-Load-SQLManagedInstance.dbc)) emphasizes on reading and writing data to Azure SQL Managed Instance using the Spark AzureSQL Connector that executes steps 4-6 in the data flow above.
