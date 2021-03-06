# Comparing Azure Databricks to HDInsight Spark

Microsoft first offered a managed version of Apache Spark starting in 2015 through its suite of Hadoop-based managed services, named [HDInsight](https://docs.microsoft.com/en-us/azure/hdinsight/spark/apache-spark-overview). This offering provided an unmodified version of the open-source Apache Spark project, but abstracted away the server management overhead that comes with building out your own Spark clusters. Apache Spark on Azure HDInsight is very much a PaaS (platform-as-a-service) offering. You have a bit of control over the infrastructure, to the degree of specifying the number of worker nodes and their sizes, the size of the head nodes, and exhaustive Spark configuration options using the Ambari portal. By comparison, Azure Databricks is more of a SaaS (software-as-a-service) offering, making it easier to use with less configuration and a faster deployment cycle.

Let's break down the differences into categories.

## Setup

The fundamental difference between provisioning a new HDInsight Spark instance and Azure Databricks is that with Azure Databricks, you first provision a Workspace, then you can create one or more clusters within that workspace. It only takes minutes to spin up a new Workspace, requiring only a handful of options during the process. With Apache Spark on HDInsight, however, you must undergo a series of steps in which you provision a single cluster. Any subsequent Spark clusters you provision are treated as separate entities. It takes approximately 20 minutes to provision a new Spark cluster on HDInsight, once you have supplied the required parameters.

### Steps to provision an Apache Spark on HDInsight cluster

- **Step 1: Basic settings**

  ![Screenshot showing step 1 form for provisioning Apache Spark on HDInsight](media/provision-apache-spark-on-hdinsight-step1.png 'Provision Apache Spark on HDInsight step 1')

  When you provision a Spark on HDInsight cluster, you must provide a unique name, select the Spark cluster type and version, provide admin and SSH credentials (admin is used to log into Ambari and Jupyter notebooks), and specify the resource group, Azure subscription, and location.

- **Step 2: Storage**

  You must select an existing or create either an Azure Storage or Azure Data Lake Store account. This is where your Spark cluster will persist its working and configuration files. Temporary working files from job runs and the like, are stored within one of the cluster VMs, as are OSS and marketplace applications. More than one cluster can use the same storage account, but not the same container. You also have the option to select external storage and either a Hive or Oozie metastore.

- **Step 3: Applications**

  These are optional applications from the Azure Marketplace.

- **Step 4: Cluster Size**

  ![Screenshot showing step 4 form for provisioning Apache Spark on HDInsight](media/provision-apache-spark-on-hdinsight-step4.png 'Provision Apache Spark on HDInsight step 1')

  The fourth step is to select the number of Worker nodes (which can be changed later), and the VM size of the Worker nodes and Head nodes. Unlike similar managed Spark offerings, HDInsight provides two head nodes instead of just one, improving availability and reliability.

- **Step 5: Advanced settings**

  It is here you can use script actions to run custom PowerShell or Bash scripts on cluster nodes during cluster provisioning. Also, this is where you input virtual network settings, which is a step you need to perform if you want to configure this as a domain-joined secure cluster or if you want it to be in the same VNet as an HDInsight Kafka cluster. Azure Databricks automatically provisions a VNet when you create a Workspace.

The final step is a summary where you confirm your settings.

### Steps to provision an Azure Databricks Workspace and cluster

- **Step 1: Create Azure Databricks workspace**

  ![Screenshot showing the form to create a new Azure Databricks Service](media/provision-databricks-service.png 'Provision Azure Databricks service')

  The first step is to create the Azure Databricks workspace. This creates the resources required to host the Azure Databricks Workspace, such as a VNet, a security group, and a storage account. There is no need to specify these resources as you do with Spark on HDInsight.

  Here we selected the Premium pricing tier so we can use role-based access controls (RBAC), and a few other features like a JDBC connection string that can be used to connect to one of the clusters within the Workspace from external tools, like Power BI. Notice that both pricing tiers are already secured with Azure Active Directory, with no additional configuration required. This is much simpler than creating a secure domain-joined (enterprise) HDInsight cluster, which you then manage using Ranger and Kerberos.

- **Step 2: Create a cluster**

  ![Screenshot showing the form used to create a new cluster](media/create-databricks-cluster.png 'Create a new cluster')

  After the Azure Databricks service is created (2-4 minutes), open the Azure Databricks Workspace. This automatically signs you in using Azure Active Directory. By default, anyone who has an account in your tenant has access, though you can configure user access to resources like notebooks at varying levels of granularity. The premium tier has the added benefit of assigning user roles and setting access based on those roles.

  You use this single form to create a new cluster. A few things of note are, the ability to select either a standard or serverless pool, enable autoscaling, and setting automatic termination. These are all features unique to Azure Databricks. More on these features in the next section.

  Depending on the size of the cluster, it will take between 2 and 5 minutes to create and start it.

## Clusters

|                                           | Spark on HDInsight | Azure Databricks |
| ----------------------------------------- | ------------------ | ---------------- |
| Provisioning time                         | ~20 minutes        | 2-5 minutes      |
| Scale # worker nodes                      | Yes                | Yes              |
| Auto-scale # worker nodes [1]             | No                 | Yes              |
| Serverless pool [2]                       | No                 | Yes              |
| Pause cluster [3]                         | No                 | Yes              |
| Preconfigured with optimal Spark settings | No                 | Yes              |
| Built-in workflow scheduler [4]           | No                 | Yes              |

[1] A [serverless pool](https://docs.azuredatabricks.net/user-guide/clusters/serverless-pools.html#id1) is a self-managed pool of cloud resources that is auto-configured for interactive Spark workloads. You provide the minimum and maximum number of workers and the worker type, and Azure Databricks provisions the compute and local storage based on your usage.

[2] In Azure Databricks, [autoscaling](https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#id1) is a simpler form of elasticity that allows you to select the minimum and maximum number of workers, and it scales up or down based on load requirements. This can provide cost-savings by preventing over-provisioning your cluster for occasionally large workloads.

[3] The ability to [automatically terminate](https://docs.azuredatabricks.net/user-guide/clusters/terminate.html#automatic-termination) the cluster after a period of inactivity is also a cost-savings benefit, because you don't pay for a cluster while it's terminated. Terminating a cluster can be thought of as pausing. Its configuration is stored so that it can be reused at a later time. With HDInsight, there is no ability to pause a cluster. You must delete it if you are not going to use it for a while, otherwise you will pay for every minute it exists. You can persist data between deleting and recreating HDInsight clusters by defining an [external Hive or Oozie metastore](https://docs.microsoft.com/azure/hdinsight/hdinsight-use-external-metadata-stores).

[4] Azure Databricks [jobs](https://docs.azuredatabricks.net/user-guide/jobs.html#id1) provide a way to run a notebook or JAR either on-demand or on a schedule. There are several parameters you can set, such as the number of retries after failures, how many concurrent jobs can run, alerts, setting dependencies, and a run schedule.

![Screenshot of the Jobs page showing a grid with a list of jobs](media/databricks-jobs-list.png 'Jobs page')

## Notebooks

The core work area within a Spark cluster takes place within a notebook. It is here that you collect, explore, wrangle, and visualize your data. The versatility of Spark means that this is also where you build and train machine learning and deep learning models, process data in batch, or process streaming data in real-time. Notebooks are first attached to a Spark cluster before they can be used.

Spark on HDInsight uses both Jupyter and Zeppelin notebooks. Given these options, Jupyter is the standard and the type of notebook you will most likely use. Zeppelin notebooks are useful when you have a domain-joined HDInsight cluster, as it allows for domain-based authentication and the ability to switch users.

Databricks notebooks are very similar to Jupyter. In fact, you can import Jupyter notebooks and run them as-is. That being the case, the controls to execute cells, mix languages (sql, markdown, etc.) and navigate through the notebook will be familiar to anyone used to working with Jupyter notebooks. There are some slight differences, however. For instance, [magic commands](https://docs.azuredatabricks.net/user-guide/notebooks/index.html#mix-languages) in Databricks notebooks have a single percent symbol (%) instead of the double percent symbol (%%) used in Jupyter. For example: `%sql select * from orders`.

### Comments

Databricks notebooks also have the ability to add [comments](https://docs.azuredatabricks.net/user-guide/notebooks/index.html#command-comments) alongside cells. You can highlight command text, then click the comment bubble to add a new comment to that command. It will associate the comment with your user, which is a great feature when you are collaborating with several users.

![Screenshot showing adding a new comment to a command](media/databricks-notebook-comments.png 'Adding a comment to a command')

### Notebooks executing other notebooks

You can run a notebook from another notebook using the `%run` magic. You can even access variables from the other notebook from the calling notebook, even without explicitly creating it. When you execute a notebook from inside another one, you can view all of the Spark jobs associated with the run, and you can view inside the notebook as it is running or after it runs, which is useful for viewing errors or other logging information.

### Widgets

Input widgets make it incredibly simple to add parameters to your notebooks and dashboards. There is a visual component that displays a miniature form that accepts user inputs. This makes it easy to rerun a notebook with different parameters, or to have users supply parameters without having to modify the code. Widgets can also be used to accept parameters from jobs, other notebooks (`%run /path/to/notebook $X="10" $Y="1"`, where `$X` and `$Y` are widget parameters), or from external processes like an [Azure Data Factory Databricks Notebook pipeline activity](https://docs.microsoft.com/azure/data-factory/transform-data-using-databricks-notebook).

### Notebook revisions

Databricks notebooks store a history of changes to your notebook. You can select an item to view the notebook at that point in time. By default, this history is kept in local storage, but when you link a Git provider like GitHub or BitBucket to a notebook, the revision history will display the commit history within the linked repository. This history list is synchronized each time you re-open the History panel. You can choose to restore a given revision directly from the panel.

![Screenshot of the History panel with GitHub commits and an option to restore a specific revision](media/revision-history-panel.png 'History panel')

### Visualizations

Databricks supports a number of visualizations out of the box. All notebooks, regardless of their language, support Databricks visualization using the `display` function. The display function includes support for visualizing multiple data types. As opposed to the handful of basic visualizations that other notebook engines provide, Azure Databricks includes several out of the box that you traditionally would need to rely on an external library such as `matplotlib` to obtain. However, if you wish to use external libraries to augment the default ones, you are free to do so.

![](media/azure-databricks-visualizations.png)

## File system

Azure Databricks uses the Databricks File System (DBFS) to accelerate access to files stored in Azure Blob storage. This distributed files system is automatically installed on Databricks Runtime clusters, and provides caching and optimized analysis over existing data. Any time you store data to DBFS, it is saved to the Azure Blob storage account that was automatically provisioned when you created the service in Azure.

Use the `dbutils` utility to work with files within DBFS as if you would from a local filesystem. You can either execute commands against `dbutils` directly:

`dbutils.fs.mkdirs("/newdir/")`

or by using the `%fs` magic for shorthand:

`%fs mkdirs "/newdir2/"`

If you would like to provide all users within your Azure Databricks workspace access to an Azure Blob storage container, you can mount it using the `dbutils.fs.mount` [command](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-storage.html#mount-an-azure-blob-storage-container).

Files are accessed by using either the `/mnt/` prefix or `dbfs:/`, both followed by the mount name specified in the `dbutils.fs.mount` command:

```python
df = spark.read.text("/mnt/%s/...." % <mount-name>)
df = spark.read.text("dbfs:/<mount-name>/...")
```

Spark on HDInsight uses the Hadoop Distributed File System (HDFS) to access files in Azure Blob storage. You can do the same within Azure Databricks by using the HDFS API. If you are accessing a public storage account, you do not need to configure anything. However, if you want to access a private storage account, you must configure access with the `spark.conf.set` [command](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-storage.html#access-azure-blob-storage-using-the-hdfs-api). Once access has been configured (in the case of private storage accounts), you can use standard Spark and Databricks APIs to read from the storage account. Note that the Azure Blob storage access path begins with `wasbs://`:

```python
df = spark.read.parquet("wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net/<your-directory-name>")
```

## Integrations

Microsoft has worked closely with Databricks to integrate many features of the Azure platform, such as Azure Storage, Azure Data Lake Store, Power BI, Azure Active Directory, Azure Data Factory, Azure SQL Data Warehouse, Azure SQL Database, and Azure Cosmos DB. These integrations make it simple for you, as a developer, to build end-to-end data architectures on Azure.

Aside from Azure integrations, Azure Databricks provides a number of other supported [data sources](https://docs.azuredatabricks.net/spark/latest/data-sources/index.html):

- Couchbase
- ElasticSearch
- Importing Hive Tables
- Neo4j
- Avro Files
- CSV Files
- JSON Files
- LZO Compressed Files
- Parquet Files
- Redis
- Riak Time Series
- Zip Files

## Workspace

One of the core strengths of Azure Databricks is its approach in creating a collaborative space for teams of people, from data scientists and engineers to data architects. While HDInsight Spark is naturally configured for a single user account, with all work being performed from the context of a single cluster, though domain-joined clusters do allow for more users, Azure Databricks comes with a [central workspace](../workspace/workspace-overview.md). This is the landing place for team members to create and share notebooks, libraries, and dashboards, create folders, clone files, view documentation and release notes, and view other's work. Each user has their own private home folder that isn't shared. Clusters are created and managed within the workspace, and executed against a common set of data and notebooks. The focus is taken off individual clusters and instead drawn into a collaborative environment that spans cluster lifecycles and emphasizes accessibility of data.

![Screenshot showing the Workspace and user folders](media/azure-databricks-workspace.png 'Azure Databricks Workspace')

## Databases and Tables

With HDInsight, if you want to persist your structured data across cluster lifecycles, or allow other clusters to gain access to this data, you need to configure an external Hive or Oozie metastore. The metastore holds metadata information for files externally stored. So when you create a DataFrame in Spark from flat files stored in Azure Blob storage, for instance, you can persist the structure within a metastore. When you delete your cluster and recreate it later on, you can access those Hive or Oozie tables from the new cluster by configuring it to attach to the metastore. Or, if you have a Spark HDInsight cluster and a Hive LLAP HDInsight cluster, they can both access the metastore.

Every Azure Databricks deployment has a central Hive metastore accessible by all clusters to persist table metadata. There is no additional configuration needed. However, you do have the option to use your existing external Hive metastore instance. In fact, you can even [use the Hive metastore of an HDInsight cluster](https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-use-external-metadata-stores) as an external metastore in Azure Databricks. This allows you to share metastores with HDInsight clusters, such as Hive LLAP, to use the same metadata for different workloads.

The Azure Databricks Hive metastore is simply called a _table_. Tables are contained within a database, which is associated with a cluster.

![Screenshot showing the Data section of the workspace, displaying a database and tables within](media/azure-databricks-tables.png 'Azure Databricks Tables')

You can view the databases and tables by selecting the **Data** menu option. You must have at least one cluster to view databases. The screenshot above shows a number of tables that have either been created through the UI, using the Create New Table option you see to the right, or have been programmatically created from a DataFrame.

There are two different types of tables in Azure Databricks: **local** and **global**. A global table is available across all clusters, and are displayed within a database as shown here. A local table, on the other hand, is not accessible from other clusters and is not registered in the Hive metastore. This is also known as a _temporary table_ or a _view_.

Here is an example of creating a new table with the UI:

![Creating a new table with the UI](media/azure-databricks-create-new-table-ui.png 'Create new table - UI')

This example demonstrates using the Upload File option.

When you select the _Create Table with UI_ button, you select which cluster to use for the preview, then you can specify parameters for creating the table, including the name, file type, and data types:

![Previewing the table with UI and specifying parameters](media/preview_new_table_ui.png 'Preview new table - UI')

You can programmatically create global tables from a DataFrame in Scala or Python:

```python
dataFrame.write.saveAsTable("<table-name>")
```

or with SQL syntax:

```sql
CREATE TABLE tableName ...
```

Local tables are created with the `createOrReplaceTempView` command: `dataFrame.createOrReplaceTempView("<table-name>")`.

These are examples of creating _managed_ tables. This means that the Spark SQL table has metadata information that stores the schema and the data itself. The word managed is used because Spark SQL manages both the data and the metadata. If you drop a managed table, the data is deleted along with its metadata.

This differs from how Spark on HDInsight works with external metastores (Hive tables). Only the metadata is stored, and the metadata includes a pointer to the data's location. In Azure Databricks, this is referred to as an _unmanaged table_. Spark SQL manages the metadata, but you specify the data location.

Here's how you can programmatically create unmanaged tables from a DataFrame in Scala or Python:

```python
dataframe.write.option('path', '<your-storage-path>').saveAsTable('<example-table>')
```

or with SQL syntax:

```sql
CREATE TABLE <example-table>(id STRING, value STRING) USING org.apache.spark.sql.parquet OPTIONS (path '<your-storage-path>')
```

## Next Steps

Read next: [Core scenarios introduction](../core-scenarios/README.md)