# REST API

The Azure Databricks user interface (UI) is very user friendly and gives you the ability to manage your workspace and clusters in an intuitive way. However, there are times where you need to automate tasks such as cluster creation, terminating clusters, uploading files to a library, and so on.

Some reasons for automating these tasks are, but not limited to:

- DevOps processes where the Azure Databricks environment is created and configured as part of an automated workflow
- You need to terminate a cluster after a data pipeline, such as an Azure Data Factory pipeline, activity ends
- Programmatically bring up a cluster of a certain size at a fixed time of day and shut it down at night
- Create a hook in Jenkins to replace an old version of your library JAR with a newer version

## REST API endpoints

Azure Databricks has two REST APIs that perform different tasks: 2.0 and 1.2.

Use [REST API 2.0](https://docs.azuredatabricks.net/api/latest/index.html) for general administration, like managing clusters and workspaces.

Use [REST API 1.2](https://docs.azuredatabricks.net/api/1.2/index.html) to execute commands directly on Azure Databricks.

Refer to the official REST API documentation for full details:

- [REST API 2.0](https://docs.azuredatabricks.net/api/latest/index.html)
  - [API Examples](https://docs.azuredatabricks.net/api/latest/examples.html)
  - [Authentication](https://docs.azuredatabricks.net/api/latest/authentication.html)
  - [Clusters API](https://docs.azuredatabricks.net/api/latest/clusters.html)
  - [DBFS API](https://docs.azuredatabricks.net/api/latest/dbfs.html)
  - [Groups API](https://docs.azuredatabricks.net/api/latest/groups.html)
  - [Jobs API](https://docs.azuredatabricks.net/api/latest/jobs.html)
  - [Libraries API](https://docs.azuredatabricks.net/api/latest/libraries.html)
  - [Secrets API](https://docs.azuredatabricks.net/api/latest/secrets.html)
  - [Token API](https://docs.azuredatabricks.net/api/latest/tokens.html)
  - [Workspace API](https://docs.azuredatabricks.net/api/latest/workspace.html)
- [REST API 1.2](https://docs.azuredatabricks.net/api/1.2/index.html)
  - [REST API use cases](https://docs.azuredatabricks.net/api/1.2/index.html#rest-api-use-cases)
  - [API categories](https://docs.azuredatabricks.net/api/1.2/index.html#api-categories)
  - [Details](https://docs.azuredatabricks.net/api/1.2/index.html#details)
  - [Get started](https://docs.azuredatabricks.net/api/1.2/index.html#get-started)
  - [API endpoints by category](https://docs.azuredatabricks.net/api/1.2/index.html#api-endpoints-by-category)
  - [Example: Upload and run a Spark JAR](https://docs.azuredatabricks.net/api/1.2/index.html#example-upload-and-run-a-spark-jar)

## Requirements

The first step you must complete before you can use the REST APIs is to generate a **personal access token**. If you have not yet generated one, follow the steps in this topic's [setup](setup.md) article to create a personal access token.

### General usage information

- The REST APIs run over HTTPS
- For retrieving information, use HTTP GET
- For modifying state, use HTTP POST
- For file upload, use `multipart/form-data`. Otherwise `application/x-www-form-urlencoded` is used
- The response content type is JSON
- Basic authentication is used to authenticate the user for every API call
- User credentials are base64 encoded and are in the HTTP header for every API call. For example, `Authorization: Bearer YWRtaW46YWRtaW4=`

### Base URL

In the following examples, replace `<databricks-instance>` with the `<REGION>.azuredatabricks.net` domain name of your Databricks deployment.

If you are unsure of this path, you can retrieve it from the top of the Overview blade of your Azure Databricks service in Azure:

![Finding the URL within the Azure Databricks overview blade in the Azure portal](media/azure-databricks-overview-blade.png 'Azure Datbricks overview blade')

Another location you can find the path is when you are logged in to the Azure Databricks workspace. You use the same base URL:

![Azure Databricks workspace URL](media/azure-databricks-url.png 'Azure Databricks workspace URL')

## Code example using cURL

When you use cURL, either store Azure Databricks API credentials under [.netrc](https://ec.haxx.se/usingcurl-netrc.html) or use BEARER authentication. Replace `<your-token>` with your personal access token you generated above.

Here is an example of a `.netrc` file configured to authenticate to an Azure Databricks cluster with a base URL of `eastus.azuredatabricks.net`. Notice that the `login` value is **token**, and the `password` value is the **personal access token**:

```
machine eastus.azuredatabricks.net
login token
password dapi6c3b0a25824ee4c246d851e672383a69
```

To request cluster information from REST API 2.0, we issue a simple GET request with cURL. Notice that `-n` is being used to tell cURL to use `.netrc`:

```
curl -n https://<databricks-instance>/api/2.0/clusters/get?cluster_id=<cluster-id>
```

You find your Cluster Id by selecting the cluster in the Azure Databricks workspace, then selecting the Tags tab on the bottom:

![Find the Cluster Id under the Tags tab within your cluster details page](media/cluster-tags-clusterid.png 'Finding the Cluster Id under Tags')

```json
{
  "cluster_id": "0626-000230-hie123",
  "spark_context_id": 891669132884809436,
  "cluster_name": "lab-staging",
  "spark_version": "4.1.x-scala2.11",
  "node_type_id": "Standard_DS3_v2",
  "driver_node_type_id": "Standard_DS3_v2",
  "autotermination_minutes": 120,
  "enable_elastic_disk": false,
  "cluster_source": "UI",
  "state": "TERMINATED",
  "state_message": "Inactive cluster terminated (inactive for 120 minutes).",
  "start_time": 1529971350168,
  "terminated_time": 1529978868888,
  "last_state_loss_time": 0,
  "autoscale": {
    "min_workers": 2,
    "max_workers": 8
  },
  "cluster_memory_mb": 0,
  "cluster_cores": 0.0,
  "default_tags": {
    "Vendor": "Databricks",
    "Creator": "myname@myemail.com",
    "ClusterName": "lab-staging",
    "ClusterId": "0626-000230-hie123"
  },
  "creator_user_name": "myname@myemail.com",
  "termination_reason": {
    "code": "INACTIVITY",
    "parameters": {
      "inactivity_duration_min": "120"
    }
  }
}
```

## Code example using Python

The following example shows how to create a [Python 3](https://docs.azuredatabricks.net/user-guide/clusters/python3.html#python-3) cluster using the Databricks REST API and the popular [requests](http://docs.python-requests.org/en/master/) Python HTTP library.

The `/api/2.0/clusters/create` method will acquire new instances from Azure if necessary. This method is asynchronous; the returned `cluster_id` can be used to poll the cluster status. When this method returns, the cluster will be in a PENDING state. The cluster will be usable once it enters a RUNNING state.

```python
import requests

DOMAIN = '<databricks-instance>'
TOKEN = '<your-token>'

response = requests.post(
  'https://%s/api/2.0/clusters/create' % (DOMAIN),
  headers={'Authorization': "Bearer " + base64.standard_b64encode("token:" + TOKEN)},
  json={
  "new_cluster": {
    "cluster_name": "demo",
    "spark_version": "4.1.x-scala2.11",
    "node_type_id": "Standard_D3_v2",
    "spark_conf": {
      "spark.speculation": true
    },
    "autoscale" : {
      "min_workers": 2,
      "max_workers": 50
    },
    "spark_env_vars": {
      "PYSPARK_PYTHON": "/databricks/python3/bin/python3",
    }
  }
)

if response.status_code == 200:
  print(response.json()['cluster_id'])
else:
  print("Error creating cluster: %s: %s" % (response.json()["error_code"], response.json()["message"]))
```

**Note:** Azure Databricks may not be able to acquire some of the requested nodes, due to cloud provider limitations or transient network issues. If it is unable to acquire a sufficient number of the requested nodes, cluster creation will terminate with an informative error message.

## Code example using C# in an Azure function

In this example, we create a new serverless [Azure Function](https://docs.microsoft.com/azure/azure-functions/) that gets triggered through a simple call to its HTTP endpoint, and uses the Azure Databricks REST API to stop a specific cluster. The scenario for this would be for either an Azure Data Factory or Apache Airflow workflow pipeline to make a call to this Function after it is finished running a Notebook in order to terminate the cluster to save resources. Using an Azure Function for this is a great idea, because if it is using a [Consumption plan](https://docs.microsoft.com/azure/azure-functions/functions-scale#consumption-plan), you will likely never have to pay anything to run it, given the [free monthly grant of 1 million requests](https://azure.microsoft.com/pricing/details/functions/). You can adapt this idea to schedule starting a cluster at a certain time at the beginning of the day and terminating it at the end of the day. You can schedule this through an [Azure Automation](https://docs.microsoft.com/azure/automation/automation-intro) runbook, an Azure Function that executes on a recurring schedule, or a [Logic App](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview) that runs on a recurring schedule or is triggered by an event or queue. The possibilities are endless.

This example assumes that you have followed the steps to [create a new Azure function](https://docs.microsoft.com/azure/azure-functions/functions-create-first-azure-function). We are storing the Azure Databricks workspace base URL, Cluster Id, and personal access token within the Function App's [App Settings](https://docs.microsoft.com/azure/azure-functions/functions-app-settings). A best practice to protect secrets within an Azure function is to store them in [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview).

The C# Azure Function code below will terminate the cluster when triggered with an HTTP request:

```c#
#r "Newtonsoft.Json"

using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using Newtonsoft.Json;
using System.Configuration;

private static HttpClient _client;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    // Reuse the HttpClient across calls as much as possible so as not to exhaust all available sockets
    // on the server on which it runs.
    _client = _client ??  new HttpClient();
    log.Info("Terminating Azure Databricks cluster.");

    // Retrieve settings from App Settings.
    var authToken = ConfigurationManager.AppSettings["AccessToken"];
    var baseUrl = ConfigurationManager.AppSettings["DatabricksBaseURL"];
    var clusterId = ConfigurationManager.AppSettings["ClusterId"];

    // Configure the HttpClient request header to add personal access token.
    _client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authToken);

    var dictionary = new Dictionary<string, string>();
    dictionary.Add("cluster_id", clusterId);

    string json = JsonConvert.SerializeObject(dictionary);
    var requestData = new StringContent(json, Encoding.UTF8, "application/json");

    // Construct the REST API Url to delete (terminate) the cluster.
    var requestUrl = $"https://{baseUrl}/api/2.0/clusters/delete";

    var response = await _client.PostAsync(requestUrl, requestData);
    var result = await response.Content.ReadAsStringAsync();

    return req.CreateResponse(HttpStatusCode.OK, result);
}
```

You can get the URL used to trigger the function on top of the function edit page as shown below:

![Select function URL on top of the function page](media/azure-function-get-function-url.png 'Get function URL')

The screenshot below shows the Get Function URL dialog and Postman window showing the function being executed. If the command is successful, an empty JSON result is returned:

![Function successfully executed with Postman](media/function-executed-with-postman.png 'Function successfully executed with Postman')

## Next steps

Learn how to use the [Databricks CLI](databricks-cli.md).
