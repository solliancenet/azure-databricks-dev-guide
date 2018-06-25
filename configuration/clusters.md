# Configuring the cluster

The clusters supported by Azure Databricks come in multiple flavors. There are standard clusters that are pre-provisioned by data scientists or data engineers to have a certain minimal capacity for both the Spark driver node and the Spark worker nodes. There are also serverless pools, where the resources to support interactive Spark workloads are provided dynamically by Azure Databricks itself, where the Spark tasks are managed to ensure fairness across users and isolation between executing notebooks. Additionally, the clusters provide different Databricks runtime versions. For example, the ML runtime provides clusters that are pre-configured specifically for doing deep-learning, including the necessary scale-out GPU drivers and deep learning libraries.

All options support autoscale, whereby the cluster will automatically scale compute up or down within a specified number of worker nodes according to the load (measured in terms of pending Spark tasks) on the cluster.

## Creating and configuring a new cluster

The **Clusters** area of the Azure Databricks workspace displays all of the clusters that have been created. It is here where you can terminate, start, modify, or create clusters.

You will notice that clusters are categorized underneath either Interactive Clusters or Job Clusters. Interactive clusters are used to analyze data collaboratively with interactive notebooks. Job clusters are used to run fast and robust automated [workloads](https://docs.azuredatabricks.net/user-guide/jobs.html#jobs) using the UI or API.

![Screenshot showing the Clusters page](media/clusters-page.png 'Clusters page')

To create a new cluster, simply select the **+ Create Cluster** button on top of the page. Please note that clusters can also be created and configured using the [Databricks CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#databricks-cli) and [Clusters API](https://docs.azuredatabricks.net/api/latest/clusters.html#cluster-api).

Below is a screenshot of the Create Cluster form. This is the same form you see when you edit a cluster. We will go over what each of the options mean:

![Screenshot showing the form used to create a new cluster](../overview/media/create-databricks-cluster.png 'Create a new cluster')

### Cluster Mode

On top of the form, you have a choice between creating a Standard cluster, or a Serverless Pool. As stated in the introduction above, serverless pools are cloud resources managed by Azure Databricks and are auto-configured for interactive Spark workloads, ensuring fairness across users and isolation between executing notebooks. You do have some level of control where you indicate the minimum and maximum number of workers and worker type. Those parameters are used by Azure Databricks to provision the compute and local storage _based on your usage_.

#### Serverless pools

You will notice that there are fewer configuration options when you select the Serverless Pool cluster mode:

![Screenshot showing the New Cluster form after selecting the Serverless Pool cluster mode](media/create-serverless-cluster.png 'New Cluster - Serverless Pool')

You do not select the driver type, just the worker type. The minimum and maximum number of workers are configured in the same way you configure the range of workers for a standard cluster when you have autoscaling enabled.

**Note:**

- This feature is in **Beta**.
- Serverless pools work only for SQL, Python, and R.
- You do not have the option to control advanced parameters like Spark Config, SSH public key, and Logging options, just Tags.
- You cannot specify the Databricks Runtime version, including the Spark version.

The key benefits of serverless pools are:

- **Auto-Configuration**: Optimizes the Spark configuration to get the best performance for SQL and machine learning workloads in a shared environment. Also chooses the best cluster parameters to save cost on infrastructure.
- **Elasticity**: Automatically scales compute resources and local storage independently based on usage. See Autoscaling and Autoscaling Local Storage.
- **Fine-grained sharing**: Provides Spark-native fine-grained sharing for maximum resource utilization and minimum query latencies.
  - **Preemption**: Proactively preempts Spark tasks from over-committed users to ensure all users get their fair share of cluster time and their jobs complete in a timely manner even when contending with dozens of other users. This uses Spark Task Preemption for High Concurrency.
  - **Fault isolation**: Creates an environment for each notebook, effectively isolating them from one another.

To choose between serverless pools and standard clusters, consider which of the following describes your environment and workload requirements.

- **Serverless pools**
  - Use SQL, Python, or R.
  - Want Azure Databricks to manage worker selection.
- **Standard cluster**
  - Use Scala.
  - Require a specific Spark version or want to configure Spark.
  - Want to control some advanced parameters like SSH public key, logging, and so on.

### Databricks Runtime Version

The Databricks Runtime is the set of core components that run on the clusters managed by Azure Databricks. It includes Apache Spark but also adds a number of components and updates that substantially improve the usability, performance, and security of big data analytics.

You can choose from among many supported Runtime versions when you create a cluster. The screenshot below shows some possible options, and the list is constantly evolving as new versions become available:

![Select list of available Databricks runtime versions](media/databricks-runtime-versions.png 'Databricks runtime version selector')

The Runtime consists of the following components:

- Apache Spark: each runtime version contains a specific Apache Spark version
- Databricks IO: a layer on top of Apache Spark that provides additional reliability and blazing performance
- Databricks Serverless: a layer on top of Apache Spark that provides fine-grained resource sharing to optimize cloud costs
- Ubuntu and its accompanying system libraries
- Pre-installed Java, Scala, Python, and R languages
- Pre-installed Java, Scala, Python, and R libraries
- GPU libraries for GPU-enabled clusters
- Databricks services that integrate with other components of the platform, such as notebooks, jobs, and cluster manager

The [Databricks Runtime Release Notes](https://docs.azuredatabricks.net/release-notes/cluster-images/index.html) list the library versions included in each Runtime version.

**Note:** As a developer, the ability to select a specific Apache Spark version is critical for ensuring compatibility with third-party libraries or that exported notebooks or other components like trained machine learning models are compatible with other environments. Please note that you cannot select the Databricks Runtime version/Spark version when you create a serverless pool-based cluster. This may be a limiting factor when you need to target a specific version.

### Python version

You can specify the Python version using this dropdown. Python 3 is supported on all Databricks Runtime versions. You can create a cluster running a specific version of Python using the API by setting the environment variable PYSPARK_PYTHON to /databricks/python/bin/python or /databricks/python3/bin/python3.

Here is an example of creating a Python 3 cluster using the Databricks REST API and the popular [requests](http://docs.python-requests.org/en/master/) Python HTTP library:

```python
import requests

DOMAIN = '<databricks-instance>'
TOKEN = '<your-token>'

response = requests.post(
  'https://%s/api/2.0/clusters/create' % (DOMAIN),
  headers={'Authorization': "Basic " + base64.standard_b64encode("token:" + TOKEN)},
  json={
  "new_cluster": {
    "spark_version": "4.0.x-scala2.11",
    "node_type_id": "Standard_D3_v2",
    'spark_env_vars': {
      'PYSPARK_PYTHON': '/databricks/python3/bin/python3',
    }
  }
)

if response.status_code == 200:
  print(response.json()['cluster_id'])
else:
  print("Error launching cluster: %s: %s" % (response.json()["error_code"], response.json()["message"]))
```

Visit the [REST API documentation page](https://docs.azuredatabricks.net/api/latest/examples.html#api-examples) for more API examples.

View the [Python Clusters FAQs](https://docs.azuredatabricks.net/user-guide/clusters/python3.html#frequently-asked-questions) for more information about Python versions in Azure Databricks clusters.

### Driver and worker types

A cluster consists of one driver node and worker nodes. When you are creating a Standard cluster, you can pick the Azure instance type separately for the driver and worker nodes. When creating Serverless Pool-based clusters, you can only select the instance type for worker nodes. This is because the service will pick a VM instance for one driver node for the Serverless pool. Different families of instance types fit different use cases, such as memory-intensive or compute-intensive workloads.

Azure Databricks maps cluster node instance types to compute units known as DBUs. A Databricks unit (“DBU”) is a unit of processing capability per hour, billed on per-second usage. See the instance types pricing page for a list of the supported instance types and their corresponding DBUs. See [Azure instance type specifications and pricing](https://azure.microsoft.com/en-us/pricing/details/databricks/).

On the pricing page, you will see workloads classified under Data Engineering or Data Analytics. The Data Engineering workload is defined as a job that both starts and terminates the cluster on which it runs. For example, a workload may be triggered by the Databricks job scheduler, which launches a new Apache Spark cluster solely for the job and automatically terminates the cluster after the job is complete.

The Data Analytics workload is any workload that is not an automated workload, for example, running a command within Databricks notebooks. These commands run on Apache Spark clusters that may persist until manually terminated. Multiple users can share a cluster to perform interactive analysis collaboratively.

Driver and worker nodes have different responsibilities:

- **Driver node**

  - Maintains state information of all notebooks attached to the cluster. Also responsible for maintaining the SparkContext and interpreting all the commands you run from a notebook or a library on the cluster. In addition, the driver node runs the Spark master that coordinates with the Spark executors.
  - The default value of the driver node type is same as the worker node type. You can choose a larger driver node type with more memory if you are planning to collect() a lot of data from Spark workers and analyze them in the notebook.
  - Since the driver node maintains all the state information of the notebooks attached, make sure to detach unused notebooks from the driver.

- **Worker node**
  - Databricks workers run the Spark executors and other services required for the proper functioning of the clusters. When you distribute your workload with Spark, all the distributed processing happens on workers.
  - To run a Spark job, you need at least one worker. If a cluster has zero workers, you can run non-Spark commands on the driver but Spark commands will fail.

**Note:** Creating a cluster without worker nodes is a cost-effective way to run single-machine workflows that do not require the Spark engine. An example of this is [using Keras on a single node](https://docs.azuredatabricks.net/applications/deep-learning/keras.html#use-keras-on-a-single-node).

### Autoscaling

When creating a cluster, you can either provide a fixed number of workers for the cluster or provide a min and max range for the number of workers for the cluster.

When you provide a fixed size cluster, Azure Databricks ensures that your cluster has the specified number of workers. When you provide a range for the number of workers, Databricks chooses the appropriate number of workers required to run your job.

Fixed size:

![Only one textbox for selecting number of workers when fixed size](media/fixed-size-cluster.png 'Fixed size cluster')

Variable size (autoscaling enabled):

![Min and max textboxes displayed when autoscaling is enabled](media/autoscaling.png 'Variable sie cluster - autoscaling enabled')

When you check the **Enable autoscaling** checkbox, you can specify the minimum and maximum number of workers. Azure Databricks automatically adjusts the cluster size between these limits during the cluster's lifetime.

During runtime Databricks will dynamically reallocate workers to account for the characteristics of your job. Certain parts of your pipeline may be more computationally demanding than others, and Databricks automatically adds additional workers during these phases of your job (and removes when they’re no longer needed).

![The minimum and maximum number of nodes displayed within a range of worker nodes, with the current size displaying in the middle](media/autoscaling-diagram.png 'Autoscaling diagram')

Autoscaling makes it easier to achieve high cluster utilization, because you don't need to provision the cluster to match a workload. This applies especially to workloads whose requirements change over time (like exploring a dataset during the course of a day), but it can also apply to a one-time shorter workload whose provisioning requirements are unknown. Autoscaling thus offers two advantages:

- Workloads can run faster compared to a constant-sized under-provisioned cluster.
- Autoscaling clusters can reduce overall costs compared to a statically-sized cluster.

Depending on the constant size of the cluster and on the workload, autoscaling gives you one or both of these benefits at the same time. Note that the cluster size can go below the minimum number of workers selected when the cloud provider terminates instances. In this case, Databricks continuously retries to increase the cluster size.

**Note:** Autoscaling works best with Databricks Runtime 3.4 and above. As a developer, you do not need to worry about overprovisioning a cluster when you use autoscaling. The complexity of your application and queries, the amount of data you are processing, and the VM sizes you select when configuring the cluster all contribute to the number of workers required to complete pending Spark tasks. Clusters that have no _pending_ tasks _do not_ scale up, as this usually indicates the cluster is fully utilized, and adding more nodes will not make the processing faster.

Refer to the [Cluster Size and Autoscaling](https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#how-autoscaling-works) documentation to learn more about how the autoscaling feature works.

### Auto termination

To save cluster resources, you can terminate a cluster either manually, or have it automatically terminated with this option. A terminated cluster cannot run notebooks or jobs, but its configuration is stored so that it can be reused at a later time.

**Note: ** Azure Databricks retains the configuration information for up to 70 interactive clusters terminated in the last 30 days and up to 30 job clusters recently terminated by the job scheduler. To keep an interactive cluster configuration even after it has been terminated for more than 30 days, an administrator can [pin](https://docs.azuredatabricks.net/user-guide/clusters/pin.html#cluster-pin) a cluster to the cluster list.

You can configure your cluster to automatically terminate after a period of inactivity by checking the **Terminate after** checkbox and specifying the number of minutes of inactivity:

![Screenshot showing the auto termination checkbox and number of minutes entered for inactivity](media/autoterminate.png 'Auto termination')

If the difference between the current time and the last command run on the cluster is more than the inactivity period specified, Azure Databricks automatically terminates that cluster.

A cluster is inactive if **all commands** on the cluster have finished executing, including Spark jobs, Structured Streaming, and JDBC.

However, there are a couple of caveats you need to be aware of when it comes to automatic termination of your cluster:

- Auto termination works best in the latest Spark versions. Older versions tend to inaccurately report cluster activity.
- Clusters do not report activity resulting from the use of Discretized Streams (DStreams). This means that an **auto-terminating cluster may be terminated while it is running DStreams**. Turn off auto termination for clusters running DStreams or consider using Structured Streaming. Structured Streaming is the recommended method, and Spark Streaming using DStreams is now considered legacy.

## Next steps

The remaining cluster options, such as Spark Config and Tags, are covered within the [Advanced Settings & Configuration](../advanced-settings-config/advanced-cluster-settings-configuration.md) topic.
