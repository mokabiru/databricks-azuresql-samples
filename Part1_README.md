## Solution Part 1
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/Solution%20Architecture%20Numbered%20.jpg)

Notebook 1: [Covid19-DatabricksML-SQLMI](https://github.com/mokabiru/databrickssqlmi/blob/master/DatabricksNotebooks/Covid19-DatabricksML-SQLMI.dbc)

This notebook executes steps 1-3 as described in the solution architecture above. It loads the dataset from the publicly available _[pandemicdatalake](https://azure.microsoft.com/en-au/services/open-datasets/catalog/ecdc-covid-19-cases/)_ in Azure Storage.

The dataset is read into a dataframe as shown below:
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/readdf1.png)
In preparing this dataframe as input featureset to the machine learning model, some cleanup is performed to drop columns that are not relevant for the model and to assemble all the relevant columns into a featureset. Also, equally important for the model’s input featureset is to index string and categorical columns using a String and Vector Indexer to flag that these are categorical values as opposed to continuous values.

Note that the *model’s outcome is to predict the number of deaths* (fatalities) based on a partial dataset the model is trained with. Hence, the column ‘deaths’ needs to be dropped from the featureset.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/features1.png)

The dataframe is split into a training (70%) and a test (30%) dataset following which a GBT Regression model is then applied to make the prediction. To achieve a reasonable level of accuracy, the model is also tuned using a cross validator which tests a grid of hyperparameters and chooses the best set from it.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/GBT.png)

The penultimate step in the machine learning workflow is to tie all the steps above in pipeline to process features, apply the evaluation metric, cross validate the hyperparameters and train the model by calling `fit()`.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/train.png)

The final step is to make predictions on the test dataset by calling `transform()`. This demo applies the prediction to the entire dataset to allow us to visualize the predictions against the actuals in downstream applications and reports.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/transform1.png)

The predicted dataset (stored in a dataframe) is then written to a staging table `dbo.StagingPredictedCovid19` in SQL MI.<BR>

To demonstrate the integration using both the Microsoft SQL Server JDBC driver library (`mssql-jdbc`) and the Azure SQL Database Spark connector library (`azure-sqldb-spark`), this notebook uses the `mssql-jdbc` driver while the notebook in Part 2 (COVID19-Load-SQLManagedInstance) uses the `azure-sqldb-spark` connector.

> While the Microsoft SQL Server JDBC driver library (mssql-jdbc) is pre-installed in Databricks Runtime 3.4 and above, the Azure SQL Database Spark connector needs to be exclusively installed as shown in the pre-requisites section of this solution.

*Compared to the built-in JDBC connector, the Azure SQL Database Spark connector provides the ability to bulk insert data into SQL databases. It can outperform row-by-row insertion with 10x to 20x faster performance. The Spark connector for SQL Server and Azure SQL Database also supports Azure Active Directory (Azure AD) authentication, enabling connectivity to Azure SQL databases from Azure Databricks using an Azure AD account.*
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/notebook1-writetoMI.png)

> It is recommended to store SQL MI credentials as secrets either in [Databricks scoped secrets or in Azure Key Vault](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes) to enable sharing the notebooks with multiple users without exposing the credentials displayed. Secrets retrieved in the notebook are always redacted when displayed assuring the SQL MI credentials are
> never exposed to users.

The records can be verified by querying the table `dbo.StagingPredictedCovid19` in SQL MI.
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/SQLMInotebook1screenshot.png)
![enter image description here](https://github.com/mokabiru/databrickssqlmi/raw/master/media/stagingtableresults.png)
In the next part of the solution, data from this staging table is read back into Databricks along with the dimension table `dbo.DimCountry`, joined together and transformed to create a denormalized dataset to be written into the fact table `dbo.FactCovid19`.
