# Advanced settings & configuration

When you create a new cluster, there are several general settings you are most likely to configure before moving on to [development tasks](../spark-apps/developing-spark-apps.md). These settings are covered in detail within the [cluster configuration](../configuration/clusters.md) topic. However, there are several advanced settings and configuration options that can enhance your cluster's capabilities, such as tagging, logging, and the Spark Config. This topic discusses each of these settings and more, and when you may need to use them.

**Note:** As compared to configuring an Apache Spark cluster locally or within a [Spark on HDInsight instance](../overview/compare-to-hdinsight-spark.md), Azure Databricks comes preconfigured with optimal Spark settings that you should not need to modify.

## Spark Config

To fine-tune Spark jobs, you can provide custom [Spark configuration properties](http://spark.apache.org/docs/latest/configuration.html) under the **Spark** tab at the bottom of the cluster creation page.

![Spark Config text area](media/spark-config-blank.png 'Spark Config')

Let's walk through an example where you would use this.

In January 2018, Databricks announced general availability of [Databricks Cache](https://docs.azuredatabricks.net/user-guide/databricks-io-cache.html#databricks-io-cache), a Databricks Runtime feature that can improve the scan speed of your Apache Spark workloads by up to **10x** without any application code change. It does this by automatically caching hot input data for a user and load balances across a cluster. It is also capable of caching 30 times more data than Spark's in-memory cache.

However, this feature is not enabled by default when you create a new cluster. You can configure your cluster to enable Databricks Cache every time it runs by entering the following settings in Spark Config:

```
spark.databricks.io.cache.enabled true
spark.databricks.io.cache.maxDiskUsage "{DISK SPACE PER NODE RESERVED FOR CACHED DATA}"
spark.databricks.io.cache.maxMetaDataCache "{DISK SPACE PER NODE RESERVED FOR CACHED METADATA}"
```

**Note:** The Databricks IO cache supports reading Parquet files from DBFS, HDFS, Azure Blob Storage, and Azure Data Lake. It does not support other storage formats such as CSV, JSON, and ORC.

Here is an example of the Databricks Cache configuration being added to the Spark Config settings:

![Spark Config text area with Databricks Cache configuration](media/spark-config-databricks-cache.png 'Spark Config')

You can update Spark Config settings on an existing cluster. When you apply new settings, you must restart the cluster for them to take effect.

## Logging

![Cluster log delivery settings](media/logging-configuration.png 'Logging')

Cluster log delivery can be configured under the **Logging**, you can specify a location to deliver driver and Spark worker logs. Logs are delivered every five minutes to your chosen destination. The final destination depends on the cluster ID. If the specified destination is `dbfs:/cluster-logs`, cluster logs for 0606-183920-walk677 are delivered to `dbfs:/cluster-logs/0606-183920-walk677`.

You can find your cluster ID by going to the Tags tab within your cluster configuration:

![ClusterId under Tags tab](media/cluster-id.png 'Tags')

After saving your Logging configuration, you will see the final path, based on your chosen destination and cluster Id:

![Completed logging path](media/logging-path.png 'Logging')

You will find **driver** and **executor** logs underneath separate subdirectories the associated node types:

![Driver and executor node type subdirectories for the logs](media/logging-subdirectories.png 'Logging')

The **driver** node type subdirectory contains `log4j`, `stderr`, and `stdout` logs:

![Log files contained within the driver node type subdirectory](media/logging-log-files.png 'Log files within driver node subdirectory')

The **executor** node type subdirectory contains additional subdirectory for each associated worker node.

## Cluster tags

Cluster tags allow you to specify [Azure tags](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags) as key-value pairs that will be propagated to all Azure resources (VMs, disks, NICs, etc.) associated with a cluster. This helps you locate related resources within the Azure Databricks workspace's managed resource group. You can also use tags to display billing information for resources with the same tag. This can help your organization keep track of costs in any number of ways, such as by cost center:

![Screenshot showing billing information downloaded from the Azure portal, by tags](media/azure-billing-by-tags.png 'Azure billing by tags')

In addition to user-provided tags, Databricks automatically applies four default tags to each cluster: ClusterName, ClusterId, Vendor, and Creator username.

![Screenshot showing adding a custom tag](media/tags-adding-custom-tag.png 'Tags')

When you view your Azure Datbricks workspace Managed Resource Group in the Azure Portal, you can differentiate resources by any number of tags. In the screenshot below, we configured the resource group to show the ClusterName tag so we can view which resources belong to which cluster:

![Screenshot showing ClusterName tags displayed within managed resource group](media/tags-managed-resource-group.png 'ClusterName tags displayed in managed resource group')

## External Hive Metastore

Azure Databricks comes preconfigured with its own Hive Metastore, which acts as a data catalog that abstracts away the schema and table properties with metadata about your tables. This simplifies access to your data without needing to know the underlying structure. This Metastore is fully managed and shared across multiple clusters.

However, if you already have your own Hive Metastore, you can use it instead. For instance, if you have an [Interactive Query (LLAP) HDInsight cluster](https://docs.microsoft.com/azure/hdinsight/interactive-query/apache-interactive-query-get-started) and want to provide easy access to shared data, you can configure both the LLAP cluster and Azure Databricks cluster with the same external Hive Metastore.

The Hive Metastore has a metastore proxy service that users connect to, and the data is stored in a relational database, such as Azure SQL Database.

### Connecting to an external metastore using an init script

[Cluster Node Initialization Scripts](https://docs.azuredatabricks.net/user-guide/clusters/init-scripts.html#init-scripts) are a convenient way to configure your clusters to connect to an existing Hive Metastore without explicitly setting required configurations when creating the cluster. Below is an example of how to do this. Refer to the [External Hive Metastore documentation](https://docs.azuredatabricks.net/user-guide/advanced/external-hive-metastore.html#id1) for more configuration examples as well as compatibility information.

```java
%scala

dbutils.fs.put(
    "/databricks/init/${cluster-name}/external-metastore.sh",
    """#!/bin/sh
      |# Loads environment variables to determine the correct JDBC driver to use.
      |source /etc/environment
      |# Quoting the label (i.e. EOF) with single quotes to disable variable interpolation.
      |cat << 'EOF' > /databricks/driver/conf/00-custom-spark.conf
      |[driver] {
      |    # Hive specific configuration options.
      |    # spark.hadoop prefix is added to make sure these Hive specific options will propagate to the metastore client.
      |    # JDBC connect string for a JDBC metastore
      |    "spark.hadoop.javax.jdo.option.ConnectionURL" = "${mssql-connection-string}"
      |
      |    # Username to use against metastore database
      |    "spark.hadoop.javax.jdo.option.ConnectionUserName" = "${mssql-username}"
      |
      |    # Password to use against metastore database
      |    "spark.hadoop.javax.jdo.option.ConnectionPassword" = "${mssql-password}"
      |
      |    # Driver class name for a JDBC metastore
      |    "spark.hadoop.javax.jdo.option.ConnectionDriverName" = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
      |
      |    # Spark specific configuration options
      |    "spark.sql.hive.metastore.version" = "${hive-version}"
      |    # Skip this one if ${hive-version} is 0.13.x.
      |    "spark.sql.hive.metastore.jars" = "${hive-jar-source}"
      |}
      |EOF
      |""".stripMargin,
    overwrite = true
)
```

### Potential problems caused by external Hive Metastores

The following issues could potentially lead to an outage if changes are made to the metastore environment:

- Broken network connectivity
- Changes to the underlying Hive version
- Changes to metastore credentials
- Firewall rules added to the metastore

To temporarily resolve the issue, you can use the Databricks Rest API to collect the current contents of the init script (if you followed the example above) and remove it while you work on fixing the issue.

```python
DOMAIN = '<databricks-instance>'
TOKEN = '<your-token>'
BASE_URL = 'https://%s/api/2.0/dbfs/' % (DOMAIN)

import json, pprint, requests

path_to_script = '/databricks/init/{clusterName}/external-metastore.sh'
env_url = 'https://yourenv.cloud.databricks.com'

# Helper to pretty print json
def pprint_j(i):
  print json.dumps(json.loads(i), indent=4, sort_keys=True)

# Read Example
read_payload = {
  'path' : path_to_script
}

resp = requests.get(
  BASE_URL + '/read',
  headers={"Authorization": "Basic " + base64.standard_b64encode("token:" + TOKEN)},
  json = read_payload)
results = resp.content
pprint_j(results)
print resp.status_code
# Decode the base64 binary strings
print json.loads(results)['data'].decode('base64')

# Delete example
delete_payload = {
  'path' : path_to_script,
  'recursive' : 'false'
}

resp = requests.post(BASE_URL + '/delete',
  headers={"Authorization": "Basic " + base64.standard_b64encode("token:" + TOKEN)},
  json = delete_payload)
results = resp.content
pprint_j(results)
print resp.status_code
```

## Next steps

Read more about [cluster configuration](../configuration/clusters.md)
