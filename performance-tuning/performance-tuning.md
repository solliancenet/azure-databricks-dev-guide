# Performance tuning

The heart and soul of Azure Databricks is the Apache Spark platform. One of the benefits Databricks brings with it is that less configuration and performance tuning is required compared to vanilla installations of Apache Spark, or hosted offerings like [Spark on HDInsight](../overview/compare-to-hdinsight-spark.md). In fact, the optimized Databricks Runtime can improve the performance of Spark jobs in the cloud by 10-100x.

Azure Databricks has gone one step beyond the base Databricks platform by integrating closely with Azure services through collaboration between Databricks and Microsoft. Azure Databricks features [optimized connectors to Azure storage platforms](../data-sources/data-sources-overview.md) (e.g. Data Lake and Blob Storage) for the fastest possible data access. Accelerated Networking provides the fastest virtualized network infrastructure in the cloud. Azure Databricks utilizes this to further improve Spark performance. Finally, Azure Databricks offers a wide array of VMs that are tuned for general workloads, compute, memory, and GPU-acceleration.

## Clusters

An [Azure Databricks cluster](../configuration/clusters.md) runs Spark workloads enhanced by the Databricks Runtime. Each cluster includes default configuration parameters for your Spark cluster at the top level and also at the level of Spark services and service instances in your Spark cluster. A Spark Job is a set of multiple tasks executed via parallel computation. Spark Jobs are generated in response to a Spark actions (such as 'collect' or 'save'). Spark uses a threadpool of tasks for parallel execution rather than a pool of JVM resources (used by MapReduce).

A key aspect of managing a cluster is monitoring all jobs on the cluster to make sure they are running in a predictable manner. This application monitoring includes Apache Spark job monitoring and optimization. You can [read more about the Azure Databricks architecture](../architecture/azure-databricks-architecture.md) to gain a better understanding of how it works at a high level.

The diagram below shows the core Spark Job workflow stages. As above, it's important to review Job workflow objects when optimizing Spark Jobs. In the diagram, the data is represented by the low-level RDD Objects. The next step is the DAG Scheduler. The DAG Scheduler interacts with the Task Scheduler to schedule, submit, launch and retry tasks. These two schedulers interact with the worker instances. Worker instances host threads and also make use of a Block Manager. The Block Manager stores and serves blocks of data to the workflow.

![RDD Stages](media/rdd-stages.png 'RDD Stages')

## Track an application in the Spark UI

When working on Spark Job performance, it's important to start by understanding how to get visibility into Job performance via the Spark monitoring tools within the Azure Databricks workspace.

After you execute a cell within a [Databricks notebook](../notebooks/notebooks-overview.md), you will see an output showing **Spark Jobs**. When you expand this tree, you will see a Spark Job number (not to be confused with [Databricks Jobs](../jobs/jobs-overview.md)) with a **View** link next to it. Select **View** to view the job execution details within Spark UI.

![Expanded Spark Jobs tree with a View link next to the job number](media/spark-jobs-view.png 'Spark Jobs view link')

Clicking the View link will open the Spark Jobs UI in a new pane that overlays the notebook. After the Spark Jobs UI renders, you can drill down to view specific implementation details for the Spark jobs that have been spawned by the application workload(s) that you started. You can review detailed information about jobs, stages, storage, environment, executors and Spark SQL via this UI.

The default view is open to the **Jobs** tab. The Jobs tab lists recently run Spark jobs, ordered by Job Id. It provides a high-level view of the status of Job workflow execution outcome by displaying the Job Id, job description, data and time that the job was submitted, job execution duration, job stage, DAG Visualization, and task status. An example of the Spark Jobs UI displayed within an overlay pane is shown below.

![Job details displayed within Spark UI overlay pane](media/spark-job-ui.png 'Job details')

As mentioned, there are a number of views available in the Spark Jobs UI. They allow you to review detailed execution information about the Spark Jobs that have been run on your cluster. This information is key when monitoring and optimizing Spark Job executions. It is especially important that you review the job stages and tasks using the DAG view and understand the overhead of each job stage so that you can verify that your Spark Job is performing as expected. The DAG (Directed Acyclic Graph) represents the different stages in the application. Each blue box in the graph represents a Spark operation invoked from the application

Select the Stage Description details for a stage to view its detailed information for the selected job:

![The Stage Description details link is shown on the bottom of the job details page](media/stage-description-details.png 'Stage Description details link')

The stage details displays a more detailed view of the DAG Visualization for the selected stage. You can also view additional metrics, an event timeline, summary metrics for all tasks, aggregated metrics by executor (along with direct links to stdout and stderr logs), and a list of each task including each task's status, duration, number of attempts, garbage collection (GC) time, and any errors:

![Stage details](media/stage-details.png 'Stage details')

Expand the **Event Timeline** to view a visualization of the job event execution details, which gives you a quick idea of how much time was spent on each step of a task:

![Expanded Event Timeline](media/event-timeline.png 'Event Timeline')

- Click the **Executors** tab to see processing and storage information for each executor. In this tab, you can also retrieve the call stack by clicking on the Thread Dump link.

If you would like to go directly to the Spark History Server to view all jobs, stages, and executors without having to execute a cell, select the cluster, then select the Spark UI tab.

## Spark Job Optimization Techniques

Listed below are a set of common Spark Job optimization challenges, considerations and recommended actions to improve results.

[See the overhead considerations section](../spark-apps/developing-spark-apps.md#overhead-considerations) of the [Developing Spark apps](/spark-apps/developing-spark-apps.md) topic for information on selecting the **appropriate data abstraction**, using the **optimal data format**, and **caching**.

### Use Memory Efficiently

Because Spark operates by placing data in memory, appropriately managing memory resources is a key aspect of optimizing the execution of Spark Jobs. There are several techniques that you can follow to use your cluster's memory efficiently. These include the following:

- Prefer smaller data partitions, account for data size, types and distribution in your partitioning strategy
- Monitor and tune Spark configuration settings

For reference, the Spark memory structure and some key executor memory parameters are shown below.

#### Spark Memory

Here are set of common practices you can try if you are addressing 'out of memory' messages:

- Review DAG Management Shuffles -> reduce by map-side reducting, pre-partition (or bucketize) source data, maximize single shuffle, reduce the amount of data sent
- Prefer 'ReduceByKey' (has fixed memory limit) to 'GroupByKey' (more powerful, i.e. aggregations, windowing, etc.. but, has unbounded memory limit)
- Prefer 'TreeReduce' (does more work on the executors or partitions) to 'Reduce' (does all work on the driver)
- Leverage DataFrame rather than the lower-level RDD object
- Create ComplexTypes which encapsulate actions, such as 'Top N', various aggregations or windowing ops

### Use bucketing

Bucketing is similar to data partitioning, but each bucket can hold a set of column values (bucket), instead of just one. This is great for partitioning on large (in the millions +) number of values, like product Ids. A bucket is determined by hashing the bucket key of the row. Bucketed tables offer unique optimizations because they store metadata about how they were bucketed and sorted.

A big benefit of bucketing is join query optimization, by avoiding shuffles (or exchanges) of tables participating in the join. Bucketing results in fewer exchanges, which means fewer stages. Shuffling and sorting are costly due to disk and network IO. Expect at least 2-5x performance gains after bucketing input tables for joins. The key to realizing these benefits is to only bucketize tables that will be used more than once, as the bucketing process takes time.

Some advanced bucketing features are:

- Query optimization based on bucketing meta-information
- Optimized aggregations
- Optimized joins

Example code for bucketing a DataFrame by person name with 42 buckets:

```python
people.write
  .bucketBy(42, "name")
  .sortBy("age")
  .saveAsTable("people_bucketed")
```

You can use partitioning and bucketing at the same time:

```python
people.write
  .partitionBy("favorite_color")
  .bucketBy(42, "name")
  .saveAsTable("people_partitioned_bucketed"))
```

### Fix slow Joins/Shuffles

If you have slow jobs on Join/Shuffle, for example it may take 20 seconds to run a map job, but 4 hours when running a job where the data is joined or shuffled, the cause is probably data skew. Data skew is defined as asymmetry in your job data. To fix data skew, you should salt the entire key, or perform an isolated salt (meaning apply the salt to only some subset of keys). If you are using the 'isolated salt' technique, you should further filter to isolate your subset of salted keys in map joins. Another option is to introduce a bucket column and pre-aggregate in buckets first.

Another factor causing slow joins could be the join type. By default, Spark uses the `SortMerge` join type. This type of join is best suited for large data sets, but is otherwise computationally expensive (slow) because it must first sort the left and right sides of data before merging them. A `Broadcast` join, on the other hand, is best suited for smaller data sets, or where one side of the join is significantly smaller than the other side. It broadcasts one side to all executors, so requires more memory for broadcasts in general.

You can change the join type in your configuration by setting `spark.sql.autoBroadcastJoinThreshold`, or you can change the type with a join hint using the DataFrame APIs (`dataframe.join(broadcast(df2))`).

Example:

```scala
// Option 1
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 1*1024*1024*1024)

// Option 2
val df1 = spark.table("FactTableA")
val df2 = spark.table("dimMP")
df1.join(broadcast(df2), Seq("PK")).
    createOrReplaceTempView("V_JOIN")
sql("SELECT col1, col2 FROM V_JOIN")
```

If you are using bucketed tables, you have a third join type: `Merge` join. A correctly pre-partitioned and pre-sorted dataset will skip the expensive sort phase in a `SortMerge` join.

The order of joins matters, particularly in more complex queries. Make sure you start with the most selective joins. Also, move joins that increase the number of rows after aggregations when possible.

Additionally, to manage parallelism, specifically to fight Cartesian Joins, you can adding nested structures, windowing and/or skip step(s) in your Spark Job.

## Indexes

The Databricks Runtime includes the `Data Skipping Index` feature, which helps avoid scanning irrelevant data. The idea behind it is that we can use file-level statistics in order to perform additional skipping at file granularity. In other words, it uses the file statistics to determine whether to scan the data within the file. This allows you to scan only those files that are important for queries you submit. This works with, but does not depend on, Hive-style partitioning.

The effectiveness of Data Skipping depends on the characteristics of your data and its physical layout. As skipping is done at file granularity, it is important that your data is horizontally partitioned across multiple files. This will typically happen as a consequence of having multiple append jobs, (shuffle) partitioning, bucketing, and/or the use of `spark.sql.files.maxRecordsPerFile`. It works best on tables with sorted buckets (`df.write.bucketBy(...).sortBy(...).saveAsTable(...)` / `CREATE TABLE ... CLUSTERED BY ... SORTED BY ...`), or with columns that are correlated with partition keys (e.g. _brandName - modelName_, _companyID - stockPrice_), but also when your data just happens to exhibit some sortedness / clusteredness (e.g. _orderID, bitcoinValue_).

**Note:** This feature is currently in _beta_, which means the syntax is likely to change. Also, this beta version has a number of important limitations to be aware of:

- It's Opt-In: needs to be enabled manually, on a per-table basis.
- It's SQL only: there is no DataFrame API for it.
- Beware: Once a table is indexed, the effects of subsequent INSERT or ADD PARTITION operations are not guaranteed to be visible until the index is explicitly `REFRESH`ed.

### SQL syntax

Use the following syntax to manage the Data Skipping Index feature on your tables.

#### Create Index

```sql
CREATE DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
```

Enables Data Skipping on the given table for the first (i.e. left-most) N supported columns, where N is controlled by `spark.databricks.io.skipping.defaultNumIndexedCols` (default: 32)

Note that partitionBy columns are always indexed and do not count towards this N.

#### Create Index For Columns

```sql
CREATE DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
    FOR COLUMNS (col1, ...)
```

Enables Data Skipping on the given table for the specified list of columns. Same as above, all partitionBy columns will always be indexed in addition to the ones specified.

#### Describe Index

```sql
DESCRIBE DATASKIPPING INDEX [EXTENDED] ON [TABLE] [db_name.]table_name
```

Displays which columns of the given table are indexed, along with the corresponding types of file-level statistic that are collected.

If `EXTENDED` is specified, a third column called "effectiveness_score" is displayed that gives an approximate measure of how beneficial we expect DataSkipping to be for filters on the corresponding columns.

#### Refresh Full Index

```sql
REFRESH DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
```

Rebuilds the whole index. This means all the table's partitions will be re-indexed.

#### Refresh Partitions

```sql
REFRESH DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
    PARTITION (part_col_name1[=val1], part_col_name2[=val2], ...)
```

Re-indexes the specified partitions only. This operation should generally be faster than full index refresh.

#### Drop Index

```sql
DROP DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
```

Disables Data Skipping on the given table and deletes all index data.

## Manual settings

To reiterate what was said at the beginning of this topic, the thing that sets Azure Databricks apart from vanilla Apache Spark installs is that Azure Databricks has **already tuned Spark for the most common workloads** running on the specific Azure VMs configured for your cluster. In most cases, you should not have to make any configuration changes. However, if you do need to manually set Spark settings, there are a couple of options.

If you want to [configure Spark properties](http://spark.apache.org/docs/latest/configuration.html) at the cluster level, affecting the settings in all notebooks attached to the cluster, update the Spark Config in your cluster configuration. These settings are applied every time the cluster is started. Details on how to do this are covered in the [advanced settings & configuration topic](../advanced-settings-config/advanced-cluster-settings-configuration.md#spark-config).

Otherwise, you can make configuration changes on your Spark context directly within your notebook, affecting just the current session.

### Viewing the Spark configuration settings

You can view your cluster's Spark configuration settings by selecting the cluster in the Azure Databricks workspace, selecting the **Spark UI** tab, then selecting the **Environment** sub-tab.

For example, we have added the following Databricks IO Cache settings to the Spark Config box under the cluster configuration:

```
spark.databricks.io.cache.enabled true
spark.databricks.io.cache.maxDiskUsage 14g
spark.databricks.io.cache.maxMetaDataCache 300m
```

![Databricks IO Cache settings in Spark Config](media/spark-config-example.png 'Spark Config')

With the cluster details open, select the **Spark UI** tab at the top of the page. Then select the **Environment** tab underneath.

![Select Spark UI, then the Environment tab](media/spark-ui-environment-tab.png 'Spark UI Environment')

This page contains all of the Spark configuration settings for your cluster. In our case, because we configured the Databricks IO Cache settings within the cluster's Spark Config, we can see those settings applied when we scroll down the list of Spark Properties.

![Databricks IO Cache settings shown within Spark Properties](media/databricks-io-cache-properties.png 'Databricks IO Cache properties')

### Demonstration of Databricks optimizations

When you create a cluster in your Azure Databricks workspace, it comes with several optimizations out of the box. This section talks about a few of those optimizations and the impact of disabling them by changing the Spark configuration settings within a notebook.

#### Data Skipping Index

In our example, we're using a Databricks table that contains 10 billion rows of stock market data. There is another large table containing stock volume data we will be joining with as well. A Data Skipping Index was created on the `stockprice` table, using the following query:

```sql
CREATE DATASKIPPING INDEX ON stockprice
```

To start, we turned off the Databricks Runtime optimizations for this demonstration:

```sql
set spark.databricks.io.skipping.enabled=false;
set spark.databricks.io.cache.enabled=false;
set spark.databricks.optimizer.dynamicPartitionPruning=false;
set spark.databricks.optimizer.aggregatePushdown.enabled=false;
```

Next, we executed a query against the data set, ensuring the `Data Skipping Index` feature was turned off:

```sql
set spark.databricks.io.skipping.enabled=false;
select count(distinct cast(time as date)) from stockprice where price > 1000;
```

This resulted in an execution time of almost **45 seconds**.

Next, we ran the same query with the Data Skipping Index on:

```sql
set spark.databricks.io.skipping.enabled=true;
select count(distinct cast(time as date)) from stockprice where price > 1000;
```

The execution result was just over **5 seconds**, a substantial improvement. This is because it skipped any Parquet files that had a price value under 1000.

#### Transparent Caching

As mentioned above, transparent caching is provided by Databricks IO Cache, which is enabled by default. To summarize, transparent caching creates a local copy of remote data, automatically. This uses the cluster's SSD rather than storing everything in-memory. This frees up memory for other operations.

To start, we once again turned off the `Data Skipping Index` feature, as well as the Databricks IO Cache:

```sql
set spark.databricks.io.skipping.enabled=false;
set spark.databricks.io.cache.enabled=false;
select date, avg(price) from stockprice group by date having avg(price) > 11
```

The amount of time it took to complete this query was over **59 seconds**.

Next, we ran the same query, enabling data skipping and data caching:

```sql
set spark.databricks.io.skipping.enabled=true;
set spark.databricks.io.cache.enabled=true;
select date, avg(price) from stockprice group by date having avg(price) > 11
```

This time it took just under **35 seconds**, a marked improvement, but it did not pull from cache this first execution. It was automatically added to cache because `spark.databricks.io.cache.enabled` was set to `true`.

When we executed the cell block a second time, it took just over **9 seconds** thanks to caching.

### Push down aggregates and Dynamic Partition Pruning

The `spark.databricks.optimizer.aggregatePushdown` setting pushes down all aggregates below joins.

Dynamic Partition Pruning, set by enabling `spark.databricks.optimizer.dynamicPartitionPruning`, will automatically run sub-selects and re-plan. After enabling this feature, when our partitioned tables are scanned, Spark pushes down the filter predicates involving the `partitionBy` keys. In that case, Spark avoids reading data that doesn’t satisfy those predicates. For example, suppose you have a table `<example-data>` that is partitioned by `<date>`. A query such as `SELECT max(id) FROM <example-data> WHERE date = '2018-06-10'` reads only the data files containing tuples whose date value matches the one specified in the query.

First we executed a more advanced query with optimizations turned **off**:

```sql
set spark.databricks.io.skipping.enabled=false;
set spark.databricks.io.cache.enabled=false;
set spark.databricks.optimizer.dynamicPartitionPruning=false;
set spark.databricks.optimizer.aggregatePushdown.enabled=false;

select avg(p.price)
from stockprice p, stockvolumes v
where p.date = v.date
and v.volume > 998
group by p.date;
```

The amount of time it took to run the above query with all the optimizations turned off was **57 seconds**.

Finally, we re-ran the query after turning on all the optimizations:

```sql
set spark.databricks.io.skipping.enabled=true;
set spark.databricks.io.cache.enabled=true;
set spark.databricks.optimizer.dynamicPartitionPruning=ture;
set spark.databricks.optimizer.aggregatePushdown.enabled=true;

select avg(p.price)
from stockprice p, stockvolumes v
where p.date = v.date
and v.volume > 998
group by p.date;
```

The command took just over **5 seconds** to complete.

## Next steps

Learn about [troubleshooting](../troubleshooting/troubleshooting-and-debugging.md) and debugging your clusters.
