# Troubleshooting

When troubleshooting issues in Azure Databricks, at one point or another you will end up debugging your Spark code, delving into your cluster details, looking through the Spark UI, and ultimately reading through logs. The purpose of this topic is to cover some of the ways you can troubleshoot your Spark jobs and [debug your clusters](#cluster-debugging).

## Troubleshooting Spark code

<!-- Adapted from https://databricks.com/blog/2016/10/18/7-tips-to-debug-apache-spark-code-faster-with-databricks.html -->

This section, which details troubleshooting steps for Spark, is adapted from a blog post by Databricks team member Vida Ha. Some information has been expanded and added. You can view the original blog post [here](https://databricks.com/blog/2016/10/18/7-tips-to-debug-apache-spark-code-faster-with-databricks.html).

### Use count() to call actions on intermediary RDDs/DataFrames

While it’s great that Spark follows a lazy computation model so it doesn’t compute anything until necessary, the downside is that when you do get an error, it may not be clear exactly where the error in your code appeared. Therefore, you’ll want to factor your code such that you can store intermediary RDDs / DataFrames as a variable. When debugging, you should call `count()` on your RDDs / DataFrames to see what stage your error occurred. This is a useful tip not just for errors, but even for optimizing the performance of your Spark jobs. It will allow you to measure the running time of each individual stage and optimize them.

In the screenshot below, we have a DataFrame named `dfLogs` that has undergone some transformation steps. We want to be sure that the transformations aren't causing any issues before we get too far into our notebook. We have set a variable named `debug`, which can be manually set within the notebook, or even passed in as a value within a notebook widget.

![Use count to call actions on interim DataFrames](media/use-count.png 'Use count')

### Working around bad input

When working with large datasets, you will have bad input that is malformed or not as you would expect it. The general recommendation is to be proactive about deciding for your use case, whether you can drop any bad input, or you want to try fixing and recovering, or otherwise investigating why your input data is bad.

DataFrames has some useful methods to clean up missing or malformed data, such as `fillna` and `replace`. You can see these in action within the [Developing Spark apps](../spark-apps/developing-spark-apps.md) topic. Here is some sample code:

```python
# Replace any missing HourlyPrecip and WindSpeed values with 0.0
df = df.fillna('0.0', subset=['HourlyPrecip', 'WindSpeed'])

# Replace any WindSpeed values of "M" with 0.005
df = df.replace('M', '0.005', 'WindSpeed')

# Replace any SeaLevelPressure values of "M" with 29.92 (the average pressure)
df = df.replace('M', '29.92', 'SeaLevelPressure')

# Replace any HourlyPrecip values of "T" (trace) with 0.005
df = df.replace('T', '0.005', 'HourlyPrecip')
```

### Use the debugging tools in Databricks notebooks

The Databricks notebook is the most effective tool in Spark code development and debugging. When you compile code into a JAR and then submit it to a Spark cluster, your whole data pipeline becomes a bit of a black box that is slow to iterate on. Notebooks allow you to isolate and find the parts of your data pipeline that are buggy or slow, and it also allows you to quickly try different fixes. In Azure Databricks, we have a number of additional built-in features to make debugging very easy for you:

#### Commenting

Other users in your organization can comment on your code and suggest improvements. You could even do code reviews directly within notebooks or just share comments on the features.

![Databricks offers real-time collaborative features in Notebooks](media/commenting.png 'Commenting')

#### Version Control

Databricks notebooks have two different types of version control. The first is the traditional method of syncing your notebooks directly into GitHub or BitBucket Cloud. More info can be found at [GitHub Version Control](https://docs.azuredatabricks.net/user-guide/notebooks/github-version-control.html) and [Bitbucket Cloud Version Control](https://docs.azuredatabricks.net/user-guide/notebooks/bitbucket-cloud-version-control.html).

![Git Integration](media/git-integration.png 'Git Integration')

The other is a history of what your notebook looked like at previous points in time, and allows you to revert to an older version with the click of a button.

![Viewing commit history in a Databricks Notebook](media/commit-history.png 'Notebook commit history')

#### Condensed view of the number of partitions

When are you running a Spark Job, you can drill down and see the jobs and stages needed to run your job and how far along they are. In the workload below, for Stage 11, there are 200 partitions, 42 have completed, and 8 are currently running. If this stage were really slow, a larger Spark cluster would allow you to run more of the partitions at once and make the overall job finish faster.

![Condensed view of Spark Job partitions](media/job-partition-condensed-view.png 'Condensed view')

#### Spark UI Pop-out

When you click the "View" link next to a job, the Spark UI will pop up for you to debug with. This is covered in detail within the [Performance Tuning](../performance-tuning/performance-tuning.md#track-an-application-in-the-spark-ui) topic. More details on using the Spark UI can be found there, and within the [debug your clusters](#cluster-debugging) section below.

![Spark UI pop-out is displayed](media/spark-ui-popout.png 'Spark UI pop-out')

A good method to follow in the Spark UI is to look at all the tasks associated with a stage if a failure occurs. You can do this by drilling down into the task page, sorting your page by the status, and examining the Errors column for the tasks that have failed. It is here that you will find a detailed error message.

### You can view ephemeral notebook details

When you run a notebook from another notebook, the notebook that you are calling will have its own set of run details involving jobs, stages, and tasks. These are called "ephemeral notebook jobs".

In the screenshot below, you see an error in the _calling_ notebook's cell where it runs another notebook. The error is simply a `NotebookExecutionException`. When we expand the exception details in this case, it does not offer very useful information.

![Calling notebook exception](media/caller-failure.png 'Calling notebook exception')

You can click the link for the Notebook job. In this case, it's called "Notebook job #3", as seen above. When you click on this, it takes you to the notebook you called so you can view its Ephemeral Notebook Job run, including all of the outputs and errors you would see if you ran the notebook cells from within the notebook.

In this case, we were able to pinpoint the exact cause of the error, which was an exception thrown because a table already existed. Now we can investigate why the `overwrite` mode failed to prevent this exception.

![Ephemeral notebook exception](media/ephemeral-notebook-exception.png 'Ephemeral notebook exception')

### Scale up Spark jobs slowly for very large datasets

If you have a very large dataset to analyze, and run into errors, you may want to try debugging/testing on a portion of your dataset first. And then when you get that running smoothly, go back to the full dataset. Having a smaller dataset makes it quick to reproduce any errors, understand the characteristics of your dataset during various stages of your data pipeline, and more. Note – you can definitely run into more problems when you run the larger dataset – the hope is just that if you can reproduce the error at a smaller scale, it’s easier for you to fix than if you needed the full dataset.

### Reproduce errors or slow Spark jobs using Databricks Jobs

As with any bug, having a reliable reproduction of the bug is half the effort of solving the bug. For that, we recommending reproducing errors and slow Spark jobs using the Databricks Jobs feature. This will help you capture the conditions for the bug/slowness and understand how flakey the bug is. You'll also capture the output permanently to look at – including the running times of each individual cell, the output of each cell, and any error message. And since our jobs feature contains a history UI, you can view any log files and the Spark UI even after your cluster has been shut down. You can also experiment with different Spark versions, instances types, cluster sizes, alternate ways to write your code, etc. in a very controlled environment to figure out how they affect your Spark job.

### Examine the partitioning for your dataset

While Spark does a good job at selecting defaults for your data, if your Spark job runs out of memory or runs slowly, bad partitioning could be at fault. If this happens, a good recommendation is to start with trying different partitioning sizes to see how they affect your job.

If your dataset is large, you can try repartitioning to a larger number to allow more parallelism on your job. A good indication of this is if in the Spark UI – you don’t have a lot of tasks, but each task is very slow to complete.

![Examining Dataset partitions in a Spark job](media/repartition.png 'Repartition')

On the other hand, if you don't have that much data and you have a ton of partitions, the overhead of the having too many partitions can also cause your job to be slow. You can repartition to a smaller number, but `coalesce` may be faster since that will try to combine partitions on the same machines rather than shuffle your data around again.

![Using coalesce() can be faster than repartition()](media/coalesce.png 'Coalesce')

If you are using Spark SQL, you can set the partition for shuffle steps by setting `spark.sql.shuffle.partitions`.

```sql
%sql set spark.sql.shuffle.partitions=1000
```

This and other configuration options can be found in the [Spark docs](http://spark.apache.org/docs/latest/sql-programming-guide.html#other-configuration-options).

## Cluster debugging

This section walks you through the process of troubleshooting an Azure Databricks cluster that is either in the failed state, or running slowly. A 'Failed Cluster' is defined as one that has terminated with an error code. If your jobs are taking longer to run than expected, or you are seeing slow response times in general, you may be experiencing failures upstream from your cluster, such as the services on which the cluster runs. However, the most common cause of these slowdowns have to do with scale. When you create a new cluster, you have many options for selecting [virtual machine sizes](https://docs.microsoft.com/azure/virtual-machines/linux/sizes) and number of worker nodes. The simplest path to fixing slowdowns may be terminating and re-configuring the cluster with larger VMs.

There are a set of general steps to take when diagnosing a failed or slow cluster. They involve getting information about all aspects of the environment, including, but not limited to, all associated Azure Services, cluster configuration, job execution information, and reproducibility of error state. The most common steps taken in this process are listed below.

### General troubleshooting steps to diagnose an Azure Databricks cluster

- Step 1: Gather data about the issue
- Step 2: Examine the cluster log files
- Step 3: Check configuration settings
- Step 4: Check for scheduled Databricks Jobs running on your cluster
- Step 5: Reproduce the failure on a different Cluster

### Step 1: Gather data about the issue

HDInsight provides many tools that you can use to identify and troubleshoot issues like cluster failures and slow response times. The key is to know what these tools are and how to use them. The following steps guide you through these tools and some options for pinpointing the issue for resolution.

#### Identify the problem

Take note of the problem to assist yourself and others during the troubleshooting process. It's easy to miss key details, so be as clear and concise as possible. Here are a few questions that can help you with this process:

- What did I expect to happen? What happened instead?
- How long did the process take to run? How long should it have run?
- Have my tasks always run slowly on this cluster? Did they run faster on a different cluster?
- When did this problem first occur? How often has it happened since?
- Has anything changed in my cluster configuration?

#### Cluster details

Gather key information about your cluster, such as:

- Name of the cluster.
- Cluster ID.
- The cluster's region (you can check for [region outages](https://azure.microsoft.com/status/)).
- The cluster's version.
- The number of worker nodes and whether that number is static or variable through autoscaling.
- Whether the cluster is a serverless or standard type.
- The type of VM assigned to the driver and worker nodes.

You can quickly get much of this top level information via the Azure Databricks workspace cluster details page. A sample screen is shown below:

![Cluster details page](media/cluster-details.png 'Cluster details')

Alternatively, you can use the [Databricks CLI](../automation-orchestration/databricks-cli.md) to get information about a cluster by running the following commands:

```shell
databricks clusters list
databricks clusters get <Cluster ID>
```

Or, you can use the Databricks REST API to view this type of information and more. See [REST API](../automation-orchestration/rest-api.md) for details and samples.

### Step 2: Examine the log files

#### Driver Logs

The direct print and log statements from your notebooks and libraries go to the driver logs. These logs have 3 main outputs:

- Standard output (stdout)
- Standard error (stderr)
- Log4j logs (log4j-active.log)

You can view the output of these logs within the **Driver Logs** tab when viewing your cluster details.

The log files get rotated periodically and the top of the page lists the older log files with timestamp information. You can download any of the logs for troubleshooting.

![Driver Logs tab in cluster details page](media/driver-logs.png 'Driver Logs')

#### Executor Logs

Executor logs are sometimes helpful if you see certain tasks are misbehaving and would like to see the logs for specific tasks.

The [Performance tuning](../performance-tuning/performance-tuning.md) topic contains a section on how to [track an application in the Spark UI](../performance-tuning/performance-tuning.md#track-an-application-in-the-spark-ui). When viewing the stage details, you can get the Executor ID where the failed task was run. Make note of that ID for the next step. As a shortcut, you can download the **stdout** and **stderr** logs from this view under the Host header.

![Failing tasks listed under step details](media/step-details-failing-tasks.png 'Step details: failing tasks')

When you navigate to your cluster details page, you can jump to the Spark Master view within Spark UI by selecting it in the dropdown list to the right of Driver Logs. This dropdown list gives you a convenient way to see the list of Workers by ID to go directly to that worker. When you select the Master cluster, you will also see the listed workers within that page.

![Spark Master cluster details page](media/spark-master-details.png 'Spark Master cluster details page')

Select the Worker whose ID corresponds with the Executor ID you noted earlier. You can view the worker details and download its logs from here.

![Spark Worker details page](media/spark-worker-details.png 'Spark Worker details page')

### Step 3: Check configuration settings

Azure Databricks clusters come preconfigured with default optimizations and settings that are suitable for most workloads. Depending on your cluster's hardware configuration, number of nodes, types of jobs you are running, the data you are working with (and how that data is being processed), as well as the type of cluster, you may need to optimize your configuration.

The [Performance Tuning](../performance-tuning/performance-tuning.md) topic covers methods for improving performance, as well as how to [view the Spark configuration settings](../performance-tuning/performance-tuning.md#viewing-the-spark-configuration-settings) within the Spark Config box in cluster settings, and the Spark UI environment tab.

Another way to configure your clusters is through [Cluster Node Initialization Scripts](https://docs.azuredatabricks.net/user-guide/clusters/init-scripts.html#init-scripts). The [Advanced Cluster Settings and Configuration](../advanced-settings-config/advanced-cluster-settings-configuration.md) topic shows how to [use an init script](../advanced-settings-config/advanced-cluster-settings-configuration.md#connecting-to-an-external-metastore-using-an-init-script), followed by how to collect the current contents of an init script and remove it while you work on fixing the issue.

### Step 4: Check for scheduled Databricks Jobs running on your cluster

When you [create a Job](../jobs/jobs-overview.md) in your Azure Databricks workspace, you have the option to select either a _new_ cluster type, which creates and manages the lifecycle of a Job cluster, or to select an _existing_ cluster type. This second option allows you to select an interactive cluster, such as the one you may be troubleshooting. Jobs can either run notebooks or JAR files on-demand or on a recurring basis.

Check for any Jobs that are configured to use your cluster and view the run history. It may be possible that one or more Jobs are configured to use your cluster, causing unexpected load while other operations are being run on the same cluster. Also, double-check the Job configuration to make sure excessive concurrent runs and long timeouts aren't set that could be causing a performance or other impact.

### Step 5: Reproduce the Failure on a different cluster

A useful technique when you are trying to track down the source of an error is to restart a new cluster with the same configuration and then to submit the job steps (one-by-one) that caused the original cluster to fail. In this way, you can check the results of each step before processing the next one. This method gives you the opportunity to correct and re-run a single step that has failed. This also has the advantage that you only load your input data once which can save time in the troubleshooting process.

To test a cluster step-by-step:

1.  Launch a new cluster, with the same configuration as the failed cluster.
2.  Submit the first job step to the cluster.
3.  When the step completes processing, check for errors in the step log files. The fastest way to locate these log files is by connecting to the master node and viewing the log files there. The step log files do not appear until the step runs for some time, finishes, or fails.
4.  If the step succeeded without error, run the next step. If there were errors, investigate the error in the log files. If it was an error in your code, make the correction and re-run the step. Continue until all steps run without error.
5.  When you are done debugging the test cluster, delete it.

## Next steps

Read about cluster [performance tuning](../performance-tuning/performance-tuning.md) and [advanced cluster settings and configuration](../advanced-settings-config/advanced-cluster-settings-configuration.md).
