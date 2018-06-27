# Data Sources

In this article we examine the various data sources available to you in Azure Databricks, with a focus on the Azure data sources. As we discuss each, sample code is provided, and for some step-by-step examples. Throughout the article we explain how to connect to each data source, and how to read and write data to them.

Azure Databricks currently supports the following Azure data sources:

- [Azure Blob storage](#azure-blob-storage)
- [Azure Data Lake Store](#azure-data-lake-store)
- [Azure Cosmos DB](#azure-cosmos-db)
- [Azure SQL Data Warehouse](#azure-sql-dat-warehouse)

## Azure Blob Storage

[Azure Blob storage](https://azure.microsoft.com/services/storage/blobs/) is a service for storing large amounts of unstructured object data, such as text or binary data, that can be accessed from anywhere in the world via HTTP or HTTPS. Azure Blob storage has a 500 TB max storage capacity.

Azure Databricks provides access to files stored in blob storage through either direct access or by mounting the Blob storage account in Data Bricks File System (DBFS).

### Access Azure Blob storage directly

You can easily access an Azure Blob storage account directly from an Azure Databricks notebook by using the Spark and Databricks APIs. Accessing the storage account in this manner will make the storage account available only to the notebook the code below is executed within.

Every request made against a storage account must be authorized, so there are three ways you can allow access to your blob storage account:

1. **Anonymous read access**: This is enabled by configuring your container for **Public read access for blobs only** or **Full public read access**. While this option makes access the files within the account from external applications easy, it should only be used if the contents of the storage container are freely accessible to anyone, and there is no sensitive or proprietary data. It is also important to note that this option is read only, so cannot be used if you need to write files into your storage account.
2. **Account access key**: Every storage account has two access keys, which can be used to authorize external access. These keys provide full access to your storage account, so they should be protected against external exposure the same as any password or secret. Should the keys become compromised, they can be regenerated in your storage account.
3. **Shared access signature (SAS)**: A shared access signature (SAS) provides a way to grant limited external access to objects in your storage account, without exposing your account key. This is the preferred method for most applications, because it does not risk compromising your account keys, limited rights can be assigned to the SAS, expiration dates can be set to automatically expire the SAS after a specified period of time, and access using the SAS can be restricted to only allowed IP addresses.

    > NOTE: To protect keys and other secrets, it is recommended that they be entered into Azure Databricks secret management, and accessed via `dbutils.secrets.get`. Read [Secret management](https://docs.azuredatabricks.net/user-guide/secrets/index.html) to learn more.

Now, let's take a look at setting up direct access to the files in an Azure Blob storage account.

1. In an Azure Databricks notebook, set up an account access key or SAS token for your account. For anonymous access, this step can be skipped.

    - Set up an account access key:

        ```python
        spark.conf.set(
          "fs.azure.account.key.<your-storage-account-name>.blob.core.windows.net",
          "<your-storage-account-access-key>")
        ```

    - Set up a SAS for a specified container:

        ```python
        spark.conf.set(
          "fs.azure.sas.<your-container-name>.<your-storage-account-name>.blob.core.windows.net",
          "<complete-connection-string-of-your-sas-for-the-container>")
        ```

    > NOTE: This step can also be accomplished for your cluster by adding `fs.azure.account.key.<your-storage-account-name>.blob.core.windows.net <your-storage-account-access-key>` into the Spark Config box when creating your cluster, or by stopping the cluster, and adding it on the Configuration tab. This will make the connection available to all notebooks in your cluster.

2. Once the account key or SAS is added to the configuration for your notebook, you can use standard Spark and Databricks APIs to read from the storage account, using a `wasbs://` path. There are no differences in accessing the container regardless of whether you used anonymous access, an account key, or a SAS.

    ```python
    df = spark.read.parquet("wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net/<your-directory-name>")

    dbutils.fs.ls("wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net/<your-directory-name>")
    ```

### Mount Azure Blob storage container with DBFS

In addition to accessing a storage account directly, you can mount Blob storage containers using Databricks Runtime 4.0 or higher. This is useful if you would like the Blob storage account being mounted to be available to all users and clusters in the Databricks workspace.

> To use the mount point in other clusters, you must run `dbutils.fs.refreshMounts()` on that running cluster to make the mount point available.

To create a mount point for an Azure Blob storage account, do the following:

1. Similar to accessing Blob storage directly, you will need your account key or SAS. In an Azure Databricks notebook, add the following code:

    > Note: The mount directory must begin with `/mnt/`.

    - Set up using an account access key:

    ```python
    dbutils.fs.mount(
      source = "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net/<your-directory-name>",
      mount_point = "/mnt/path/to/mount",
      extra_configs = {"fs.azure.account.key.<your-account-key>.blob.core.windows.net": "<your-account-key>"})
    ```

    - Set up using a SAS for a specified container:

    ```python
    dbutils.fs.mount(
      source = "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net/<your-directory-name>",
      mount_point = "/mnt/path/to/mount",
      extra_configs = {"fs.azure.sas.<your-container-name>.<your-account-name>.blob.core.windows.net": "<your-sas-token>"})
    ```

2. Once that is done, you are ready to start working with your files, just as if they were part of the DBFS. Use the following code to access files at the mount point:

    ```python
    df = spark.read.text("wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net/<your-directory-name>/file-001.txt")

    # List the file at the mount point
    dbutils.fs.ls("/mnt/path/to/mount/")
    ```

Now, let's step through a real example of this. To complete this you will need a Databricks workspace with an active cluster. We will access files from a Blob storage account made accessible with a SAS, or feel free to substitute the connection information for your own Blob storage account.

1. Open a new notebook in your Azure Databricks workspace, and add the following code to declare and assign variables into the first cell:

    ```python
    accountName = "retaildatasamples"
    containerName = "data"
    directory = "retaildata/rawdata/"
    sasToken = "?sv=2017-11-09&ss=b&srt=sco&sp=rl&se=2099-12-31T16:59:59Z&st=2018-06-26T21:42:10Z&spr=https&sig=%2BwsAw0KISuMuqlKpAHcTHHwerXcXhPFpcWPw1rUN9X0%3D"
    ```

2. Insert another cell, and add the following to perform the mounting operation. This will mount the Blob storage account at "/mnt/retaildata/rawdata/".

    ```python
    # Mount the Blob storage account at /mnt/<directory>.
    dbutils.fs.mount(
      source = "wasbs://" + containerName + "@" + accountName + ".blob.core.windows.net/" + directory,
      mount_point = "/mnt/" + directory,
      extra_configs = {"fs.azure.sas." + containerName + "." + accountName + ".blob.core.windows.net": sasToken})
    ```

3. Finally, you can run `dbutils.fs.ls` to list the files in one of the directories included at the mount point.

    ```python
    # Inspect the files in the EventData directory
    dbutils.fs.ls("/mnt/" + directory + "EventData")
    ```

4. To read files from the mounted storage into a DataFrame, use the `dbfs:/` prefix on the file path, indicating it is being accessed via DBFS, and not the `wasbs://` path that was used to access files in Blob storage with the direct access method.

```python
df = spark.read.options(header='true', inferSchema='true').csv("dbfs:/mnt/retaildata/rawdata/EventData/part*.csv", header=True)
```

To unmount a Blob storage account you use `dbutils.fs.unmount`, like the following:

```python
dbutils.fs.unmount("/mnt/retaildata/rawdata")
```

## Azure Data Lake Store

[Azure Data Lake Store](https://azure.microsoft.com/en-us/services/data-lake-store/) is an enterprise-wide hyper-scale repository for big data analytic workloads. It is a step up from Azure Blob storage, providing unlimited storage capacity, and greatly improved performance. Azure Data Lake Store enables you to capture data of any size, type, and ingestion speed in a single place for operational and exploratory analytics. Azure Data Lake Store is specifically designed to enable analytics on the stored data and is tuned for performance for data analytics scenarios.

As with Azure Blob storage, Azure Databricks provides access to files stored in Azure Data Lake Store through either direct access or by mounting the Azure Data Lake Store with DDBFS.

### Access Azure Data Lake Store directly

Setting up access to an Azure Data Lake Store directly from an Azure Databricks notebook is similar to the setup for an Azure Blob storage account. You must provide Spark with the service credentials necessary to authenticate against your Data Lake Store. The Data Lake Store provides directory level access control, so the service principal must have access to the directories that you want to read from as well as the Data Lake Store resource.

    > NOTE: To protect keys and other secrets, it is recommended that they be entered into Azure Databricks secret management, and accessed via `dbutils.secrets.get`. Read [Secret management](https://docs.azuredatabricks.net/user-guide/secrets/index.html) to learn more.

Now, let's take a look at setting up direct access to the files in an Azure Data Lake Store.

1. In an Azure Databricks notebook, you must provide your service credentials so the notebooks can authenticate.

    ```python
    spark.conf.set("dfs.adls.oauth2.access.token.provider.type", "ClientCredential")
    spark.conf.set("dfs.adls.oauth2.client.id", "<your-service-client-id>")
    spark.conf.set("dfs.adls.oauth2.credential", "<your-service-credentials>")
    spark.conf.set("dfs.adls.oauth2.refresh.url", "https://login.microsoftonline.com/<your-directory-id>/oauth2/token")
    ```

2. Once the service client credentials are added to the configuration for your notebook, you can use standard Spark and Databricks APIs to read from the Azure Data Lake Store. Files are accessed using a `adl://` path.

    ```python
    val df = spark.read.parquet("adl://<your-data-lake-store-account-name>.azuredatalakestore.net/<your-directory-name>")

    dbutils.fs.ls("adl://<your-data-lake-store-account-name>.azuredatalakestore.net/<your-directory-name>")
    ```

### Mount Azure Data Lake Store with DBFS

In addition to accessing Azure Data Lake Store directly, you can also mount a Data Lake Store or a folder inside it through DBFS. This gives all users and clusters in the same workspace the ability to access the Data Lake Store or the folder inside it through the mount point. DBFS uses the credentials you provide when you create the mount point to access the mounted Azure Data Lake Store.

> Important: The service client that you use to access the Data Lake Store should be granted access only to that Data Lake Store; it should not be granted access to other resources in Azure.

To create a mount point for an Azure Data Lake Store, complete the following:

1. Similar to accessing the Data Lake Store directly, you will need your service credentials. In your notebook, add the following code:

    > Note: The mount directory must begin with `/mnt/`.

    ```python
    configs = {"dfs.adls.oauth2.access.token.provider.type": "ClientCredential",
               "dfs.adls.oauth2.client.id": "<your-service-client-id>",
               "dfs.adls.oauth2.credential": "<your-service-credentials>",
               "dfs.adls.oauth2.refresh.url": "https://login.microsoftonline.com/<your-directory-id>/oauth2/token"}

    dbutils.fs.mount(
      source = "adl://<your-data-lake-store-account-name>.azuredatalakestore.net/<your-directory-name>",
      mount_point = "/mnt/[mount-name]",
      extra_configs = configs)
    ```

    > [mount-name] is a DBFS path representing where the Data Lake Store or a folder inside it (specified in source) will be mounted in DBFS.

2. To access files in your Azure Data Lake Store, use code similar to the following:

    ```python
    df = spark.read.text("/mnt/%s/...." % [mount-name])
    df = spark.read.text("dbfs:/[mount-name]/....")
    ```

3. To unmount the Data Lake Store, use the following:

    ```python
    dbutils.fs.unmount("/mnt/<mount-name>")
    ```

## Upload files directly into Azure Databricks tables

Before moving on to the next Azure data source, we are going to highlight another way to get data directly into Azure Databricks: upload CSV files directly into Azure Databricks global tables.

> Make sure your cluster is started before beginning the steps below, as it will be used to preview the data uploaded from the CSV file.

1. Three sample files to be used for this can be found at <http://bit.ly/2wGAqrl>. Download the ZIP file, and extract the three files contained within:

    - FlightDelaysWithAirportCodes.csv
    - FlightWeatherWithAirportCodes.csv
    - AirportCodeLocationLookupClean.csv

2. Open your Azure Databricks workspace in the [Azure portal](https://portal.azure.com).

3. In your workspace, select **Data** from the left-hand menu, select **Default** under Databases, and then select the plus (+) icon in the top right corner, next to Tables.

    ![In the Azure Databricks workspace, Data is selected in the left-hand menu, and default is highlighted under Databases.](media/azure-databricks-workspace-databases.png "Azure Databricks databases")

4. Select **Upload File** under Create New Table, and then drag and drop the `FlightDelaysWithAirportCodes.csv` file into the file area. (Note: Clicking Select to browse to the file won't work, because it attempts to find files in the /FileStore/tables/ directory in DBFS.)

    ![In the Create New Table dialog in Azure Databricks, Upload File is selected and the FlightDelaysWithAirportCodes.csv file has been dropped in the file area.](media/azure-databricks-workspace-databases-create-new-table.png "Create new table")

5. Select **Create Table with UI**, and then select your cluster to preview the table and select **Preview Table**.

6. Change the Table Name to "flight_delays_with_airport_codes" (remove "_csv"), check the **First row is header** and **Infer schema** boxes, and then select **Create Table**.

    > For the demo, we will use **Infer schema** to assign data types, but for real-world scenarios, you would want to evaluate each field, and the data type being assigned, and make sure they are correct for your data.

    ![In the Specify Table Attributes section, the Table Name is specified as "flight_delays_with_airport_codes", and the First row is header and Infer schema check boxes are checked.](media/azure-databricks-workspace-create-table-specify-attributes.png "Specify Table Attributes")

7. Select **Data** again from the left-hand menu, and you will notice **flight_delays_with_airport_codes** now listed under tables.

8. Repeat steps 3 - 6 above for the `FlightWeatherWithAirportCodes.csv` and `AirportCodeLocationLookupClean.csv` files, changing the name in step 6 appropriately for each file.

9. Each of the tables is now accessible from your cluster, as will be demonstrated below in the Azure Cosmos DB and SQL Data Warehouse examples below.

## Cosmos DB

[Azure Cosmos DB](https://azure.microsoft.com/en-us/services/cosmos-db/) is Microsoft’s globally distributed, multi-model database. Azure Cosmos DB enables you to elastically and independently scale throughput and storage across any number of Azure’s geographic regions. It offers throughput, latency, availability, and consistency guarantees with comprehensive service level agreements (SLAs). Azure Cosmos DB provides APIs for the following data models, with SDKs available in multiple languages:

- SQL API
- MongoDB API
- Cassandra API
- Graph (Gremlin) API
- Table API

To demonstrate how to connect to Cosmos DB, and read and write data, you can following along in the step-by-step walk-through below.

### Prerequisites

Before continuing with the walk-through, you need the following:

1. An Azure Cosmos DB in your subscription.
2. Within your Cosmos DB instance, create a new database named "demo".
3. Add a collection named "airport_code_location_lookup" to the "demo" database.
4. You uploaded the files as described in [Upload files directly into Azure Databricks tables](#upload-files-directly-into-azure-databricks-tables).

Once your Cosmos DB is provisioned, the following steps need to be performed prior to connecting Azure Databricks with Azure Databricks.

- A Databricks workspace with an active cluster.
- Prior to connecting your Azure Databricks instance with Azure Cosmos DB, you need to include the Azure Cosmos DB Spark JAR, by following these instructions:
  - Download the [azure-cosmosdb-spark_2.2.0_2.11 (1.11) library](http://repo1.maven.org/maven2/com/microsoft/azure/azure-cosmosdb-spark_2.2.0_2.11/1.1.1/azure-cosmosdb-spark_2.2.0_2.11-1.1.1-uber.jar) (The connector requires Databricks Runtime 3.4 or higher.)
  - Upload the downloaded JAR files to Databricks following the instructions in [Upload a Java JAR or Scala JAR](https://docs.azuredatabricks.net/user-guide/libraries.html#uploading-libraries)
  - [Attach the uploaded libraries](https://docs.azuredatabricks.net/user-guide/libraries.html#attach-libraries) to your Databricks cluster
    > Learn more about the [Azure Cosmos DB Spark Connector](https://github.com/Azure/azure-cosmosdb-spark).

### Reading data from Cosmos DB

To demonstrate how to create a connection, and enable querying against Cosmos DB, we've providing the following step-by-step walk-through.

1. In order to query Cosmos DB, you need to first create a configuration object that contains the configuration information. If you are curious, read the [configuration reference](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuration-references) for details on all of the options.

    The core items you need to provide are:

    - **Endpoint**: Your Cosmos DB url (i.e. <https://your-cosmos-db.documents.azure.com:443/>)
    - **Masterkey**: The primary or secondary key string for you Cosmos DB account
    - **Database**: The name of the database
    - **Collection**: The name of the collection that you wish to query

2. In your Databricks workspace, create a new notebook and enter the following code to create the connection to the demo environment, substituting in your values for the tokenized components:

    ```python
    readConfig = {
        "Endpoint" : "https://<your-cosmos-db>.documents.azure.com:443/",
        "Masterkey" : "<your-master-key>",
        "Database" : "demo",
        "Collection" : "airport_code_location_lookup",
        "preferredRegions" : "<your-cosmos-db-region>",
        "SamplingRatio" : "1.0",
        "schema_samplesize" : "1000",
        "query_pagesize" : "2147483647"
    }
    ```

3. After providing the configuration, you can use the Read API to query the collection.

    ```python
    # Connect via Spark connector to create Spark DataFrame
    airport_code_location_lookup = spark.read.format("com.microsoft.azure.cosmosdb.spark").options(**readConfig).load()
    display(airport_code_location_lookup)
    ```

4. At this point, there isn't any data in the `airport_code_location_lookup` collection, so you will get "OK" back as a response. Move on to the next section to write some data to Cosmos DB.

### Write data to Cosmos DB

Expanding on the example above, let's look at what is involved in writing data to Cosmos DB. By creating a DataFrame that contains the desired data, you can use the Write API to save the data to Cosmos DB.

To write to the Cosmos DB, do the following:

1. First, insert a new cell into your notebook, and add the following code to set the `writeConfig` for your Cosmos DB instance. Take note of the `Upsert` parameter used here, which enables Upsert actions on the Cosmos DB side.

    ```python
    writeConfig = {
        "Endpoint" : "https://<your-cosmos-db>.documents.azure.com:443/",
        "Masterkey" : "<your-master-key>",
        "Database" : "demo",
        "Collection" : "airport_code_location_lookup",
        "Upsert" : "true"
    }
    ```

2. In a new cell, create a new DataFrame containing the data from the `airport_code_location_lookup` table you created in the [Upload files directly into Azure Databricks tables](#upload-files-directly-into-azure-databricks-tables) section above.

    ```python
    airportCodes = spark.sql("SELECT * FROM airport_code_location_lookup_clean")
    ```

3. Now, add another new cell, and add code to write the data from the `airportCodes` DatFrame to the `airport_code_location_lookup` collection in Cosmos DB.

    ```python
    airportCodes.write.format("com.microsoft.azure.cosmosdb.spark").options(**writeConfig).save()
    ```

4. You can now rerun the read query and view the data written to the table.

    ```sql
    airport_code_location_lookup = spark.read.format("com.microsoft.azure.cosmosdb.spark").options(**readConfig).load()
    display(airport_code_location_lookup)
    ```

    ![Query results from the airport_code_location_lookup collection in Cosmos DB are displayed.](media/cosmos-db-airport-codes-query-results.png "Query results from Cosmos DB")

5. Now, let's insert a new row into the `airport_code_location_lookup` table.

6. Spark DataFrames are immutable, so if you want to insert new rows into a Cosmos DB collection, you need to create a new DataFrame. If you want to both modify and insert, the connector supports the upsert operation. You can then create a union between the DataFrames, so an upsert can be performed that will result in the one new row being added. In the new row we create below, we only specify values for the Airport, Airport_Id, Display_Airport_Name, Latitude, and Longitude fields. By setting all the other values to `None` we allow Cosmos DB to automatically set those values on the server side.

    ```python
    from pyspark.sql import Row

    newRows = [
      Row("ZZZ","99999","Demo Airport","44.31666667","-75.89972222",None,None,None,None,None,None)
    ]

    parallelizeRows = spark.sparkContext.parallelize(newRows)

    # Create a new DataFrame
    new_airport_codes = spark.createDataFrame(parallelizeRows, airport_code_location_lookup.schema)

    # Union the new DataFrame with the airport_code_location_lookup DataFrame
    updated_airport_codes = airport_code_location_lookup.union(new_airport_codes)
    display(updated_airport_codes)
    ```

7. By default, the Connector will not allow writing to a collection that has documents already. In order to perform an upsert, the `overwrite` mode must specified.

    ```python
    updated_airport_codes.write.format("com.microsoft.azure.cosmosdb.spark").mode("overwrite").options(**writeConfig).save()
    ```

8. Now, run the read command again to retrieve the newly inserted row from Cosmos DB, and write the results into a temporary view.

    ```python
    airport_codes = spark.read.format("com.microsoft.azure.cosmosdb.spark").options(**readConfig).load()
    airport_codes.createOrReplaceTempView("airportCodes")
    ```

9. Finally, use SQL to query for the view for the newly added airport code.

    ```sql
    %sql
    SELECT * FROM airportCodes WHERE airport = 'ZZZ'
    ```

    ![The newly inserted airport code is highlighted in the query results.](media/cosmos-db-upsert-query-results.png "Query results")

### Reading the Azure Cosmos DB change feed

Azure Cosmos DB is well-suited for IoT, gaming, retail, and operational logging applications. A common design pattern in these applications is to use changes to the data to kick off additional actions. These additional actions could be any of the following:

- Triggering a notification or a call to an API when a document is inserted or modified.
- Stream processing for IoT or performing analytics.
- Additional data movement by synchronizing with a cache, search engine, or data warehouse, or archiving data to cold storage.

The **change feed** support in Azure Cosmos DB enables you to build efficient and scalable solutions for each of these patterns.

It is possible to read the change feed from your Azure Databricks notebook, as shown below.

1. First, create a shared folder in your workspace which will act as the change feed checkpoint location. Select **Workspace** from the left-hand menu, select the drop down arrow next to Workspace, and then select **Create > Folder**.

    ![Create a new folder through the Workspace menu.](media/databricks-workspace-create-folder.png "Workspace create folder")

2. Name the folder **CosmosChangeFeed** and select **Create**.

3. Now, return to your notebook, and add the following code into a new cell, updating the tokenized values as appropriate. Note the `ReadChangeFeed` parameter in the config.

    ```python
    # Adding variables
    rollingChangeFeed = False
    startFromTheBeginning = False
    useNextToken = True

    changeFeedConfig = {
      "Endpoint" : "https://<your-cosmos-db>.documents.azure.com:443/",
      "Masterkey" : "<your-master-key>",
      "Database" : "demo",
      "Collection" : "airport_code_location_lookup",
      "ReadChangeFeed" : "true",
      "ChangeFeedQueryName" : str(rollingChangeFeed) + str(startFromTheBeginning) + str(useNextToken),
      "ChangeFeedStartFromTheBeginning" : str(startFromTheBeginning),
      "ChangeFeedUseNextToken" : str(useNextToken),
      "RollingChangeFeed" : str(rollingChangeFeed),
      "ChangeFeedCheckpointLocation" : "/CosmosChangeFeed/changefeedcheckpointlocation",
      "SamplingRatio" : "1.0"
    }

    newAirportCodes = spark.read.format("com.microsoft.azure.cosmosdb.spark").options(**changeFeedConfig).load()
    ```

4. Insert another cell, and add the following code to show the contents of the change feed. 

    ```python
    newAirportCodes.show()
    ```

    > If the change feed is empty, you can return to step 6 under Write data to Cosmos DV above, change the `AIRPORT` value of the new row you inserted and run the query again to force changes into the feed, then rerun this code again.

> If you are following along in the walk-through, we are done with Azure Cosmos DB for this article, so it is recommended that you delete your Azure Cosmos DB instance to save on costs.

## SQL Data Warehouse

[Azure SQL Data Warehouse](https://azure.microsoft.com/en-us/services/sql-data-warehouse/) is a cloud-based enterprise data warehouse that leverages massively parallel processing (MPP) to quickly run complex queries across petabytes of data. Use SQL Data Warehouse as a key component of a big data solution. Import big data into SQL Data Warehouse with simple PolyBase T-SQL queries, and then use the power of MPP to run high-performance analytics. As you integrate and analyze, the data warehouse will become the single version of truth your business can count on for insights.

You can access Azure SQL Data Warehouse (SQL DW) from Azure Databricks using the SQL Data Warehouse connector (referred to as the SQL DW connector), a data source implementation for Apache Spark that uses Azure Blob storage, and PolyBase in SQL DW to transfer large volumes of data efficiently between a Databricks cluster and a SQL DW instance.

Both the Azure Databricks cluster and the SQL DW instance access a common Blob storage container to exchange data between these two systems. In Databricks, Spark jobs are triggered by the SQL DW connector to read data from and write data to the Blob storage container. On the SQL DW side, data loading and unloading operations performed by PolyBase are triggered by the SQL DW connector through JDBC.

The SQL DW connector is more suited to ETL than to interactive queries, because each query execution can extract large amounts of data to Blob storage. If you plan to perform several queries against the same SQL DW table, we recommend that you save the extracted data in a format such as Parquet.

### Prerequisites

The following steps need to be performed prior to connecting Azure Databricks with SQL Data Warehouse.

1. Create an Azure Databricks workspace and cluster
2. Create a new Databricks notebook
3. Create a general purpose Azure Storage account
4. Provision an Azure SQL Data Warehouse
5. You uploaded the files as described in [Upload files directly into Azure Databricks tables](#upload-files-directly-into-azure-databricks-tables).

There are two pre-requisites for connecting Azure Databricks with SQL Data Warehouse that apply to the SQL Data Warehouse:

1. You need to [create a database master key](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/create-a-database-master-key) for the Azure SQL Data Warehouse
2. You need to ensure that the [Firewall](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-firewall-configure#manage-firewall-rules-using-the-azure-portal) on the Azure SQL Server that contains your SQL Data Warehouse is configured to allow Azure services to connect (e.g., Allow access to Azure services is set to On).

Azure Storage Blobs are used as the intermediary for the exchange of data between Azure Databricks and Azure SQL Data Warehouse. As a result of this, you will need to:

1. Acquire the Account Name and Account Key for that Storage Account
2. Create a container that will be used to store data used during the exchange (this must exists before you run any queries against SQL DW)

### Enabling Storage account access

In the steps above learned how to [mount a Blob storage account](#mount-azure-blob-storage-container-with-dbfs). Open your Databricks notebook, and perform those steps here to mount your Blob storage account. This will be used as the intermediary for the exchange of data between Azure Databricks and Azure SQL Data Warehouse.

### Connection to your SQL Data Warehouse

1. In order to connect to your Azure SQL Data Warehouse from the notebook, you will need to provide the JDBC connection string for your Azure SQL Data Warehouse. You can obtain the JDBC connection string by selecting **Connection strings** on the overview blade of your SQL Data Warehouse, and copying the value from the JDBC tab.

    ![The JDBC tab of the SQL Data Warehouse's Connection strings blade is selected.](media/azure-sql-dw-jdbc-connection-string.png "JDBC Connection string")

2. If you haven't done it already, mount your storage account, as mentioned in the [Enabling Storage account access](#enabling-storage-account-access) section above.

3. In your notebook, create a new cell, and add the following code to insert the demo SQL DW JDBC connection string:

    ```python
    sql_dw_connection_string = "jdbc:sqlserver://databrick-demo-dw.database.windows.net:1433;database=databricks-sqldw;user=demouser@databrick-demo-dw;password=Abc!1234567890;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"
    ```

4. You are now going to read data from the `flight_delays_with_airport_codes` Databricks table you created in the [Upload files directly into Azure Databricks tables](#upload-files-directly-into-azure-databricks-tables) section above. Add a new cell to your notebook, and run the following to create a DataFrame from the data in the CSV file:

    ```python
    df = spark.sql("SELECT * FROM flight_delays_with_airport_codes")
    ```

5. Insert another cell, and add the following code to create a new table in your SQL Data Warehouse from the DataFrame you created in the previous step. Be sure to replace `storage_account_name` and `storage_container_name` with appropriate values for your Blob storage account.

    ```python
    new_table_name = "flight_delays_with_airport_codes"
    temp_dir_url = "wasbs://{}@{}.blob.core.windows.net/".format(storage_container_name, storage_account_name)

    df.write \
      .format("com.databricks.spark.sqldw") \
      .option("url", sql_dw_connection_string) \
      .option("forward_spark_azure_storage_credentials", "true") \
      .option("dbtable", new_table_name) \
      .option("tempdir", temp_dir_url) \
      .save()
    ```

6. With a table and data now in the SQL Data Warehouse, you can query directly against it, as follows:

    ```python
    query = "SELECT * FROM flight_delays_with_airport_codes"

    df = spark.read \
      .format("com.databricks.spark.sqldw") \
      .option("url", sql_dw_connection_string) \
      .option("tempdir", temp_dir_url) \
      .option("forward_spark_azure_storage_credentials", "true") \
      .option("query", query) \
      .load()

    display(df)
    ```

> If you are following along in the walk-through, we are done with Azure SQL Data Warehouse for this article, so it is recommended that you pause or delete your Azure SQL Data Warehouse instance to save on costs.

## Next Steps

Read next: [Solution oriented scenarios](../solution-oriented-scenarios/readme.md)