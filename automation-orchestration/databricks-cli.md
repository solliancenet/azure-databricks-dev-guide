# Databricks CLI

The [Databricks Command Line Interface (CLI)](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html) is an open source tool which provides an easy to use interface to the Azure Databricks platform. The CLI is built on top of the [Databricks REST APIs](rest-api.md), and is organized into command groups based on the [Workspace API](https://docs.azuredatabricks.net/api/latest/workspace.html#workspace-api), [DBFS API](https://docs.azuredatabricks.net/api/latest/dbfs.html#dbfs-api), [Jobs API](https://docs.azuredatabricks.net/api/latest/jobs.html#job-api), [Clusters API](https://docs.azuredatabricks.net/api/latest/clusters.html#cluster-api), [Libraries API](https://docs.azuredatabricks.net/api/latest/libraries.html#library-api), and [Secrets API](https://docs.azuredatabricks.net/api/latest/secrets.html#secrets-api): `workspace`, `fs`, `jobs`, `runs`, `clusters`, `libraries`, and `secrets`.

The CLI provides an easy alternative to the mild programming required to use the REST API. However, if you need to add automation to your data pipeline orchestration or as part of your DevOps processes, using the REST API is preferred.

You can also use the Databricks CLI within the Azure portal using Azure Cloud Shell. Read how to do this under the [Azure Cloud Shell section](#use-databricks-cli-from-azure-cloud-shell).

**NOTE:** This CLI is under active development and is released as an experimental client. This means that interfaces are still subject to change.

## Configure your environment

The first step you must complete before you can use the Databricks CLI is to generate a **personal access token**. If you have not yet generated one, follow the steps in this topic's [setup](setup.md) article to create a personal access token.

You need to have either of the following minimum versions of [Python installed](https://www.python.org/downloads/):

- Python 2 - 2.7.9 and above
- Python 3 - 3.6 and above

**Known Issues:**

> On MacOS, the default Python 2 installation does not implement the TLSv1_2 protocol and running the CLI with this Python installation results in the error: `AttributeError: 'module' object has no attribute 'PROTOCOL_TLSv1_2'`.
>
> You can use [Homebrew](https://brew.sh/) to install a version of Python that has `ssl.PROTOCOL_TLSv1_2`:
>
> Run brew install python2 to install Python 2 or brew install python to install Python 3. Then update your path to prefer the newly installed Python.

## Install the CLI

The CLI is installed using `pip`. It can be run from your workstation, remote machine, Docker image, or the Azure Cloud Shell.

### Install using pip

To install the Databricks CLI, run `pip install databricks-cli` using the appropriate version of pip for your Python installation. If you are using Python 3, run `pip3`.

### Use CLI on Docker

```shell
# build image
docker build -t databricks-cli .

# run container
docker run -it databricks-cli

# run command in docker
docker run -it databricks-cli fs --help
```

### Use Databricks CLI from Azure Cloud Shell

1.  Log in to the [Azure portal](https://portal.azure.com).

2.  From the top-right corner, click the **Cloud Shell** icon.

    ![Launch Cloud Shell](media/launch-azure-cloud-shell.png 'Launch Azure Cloud Shell')

3.  Make sure you select **Bash** for the Cloud Shell environment. You can select from the drop-down option, as shown in the following screenshot.

    ![Select Bash for the Cloud Shell environment](media/select-bash-for-shell.png 'Select Bash')

4.  Create a virtual environment in which you can install the Databricks CLI. In the snippet below, you create a virtual environment called `databrickscli`.

    virtualenv -p /usr/bin/python2.7 databrickscli

5.  Switch to the virtual environment you created.

    source databrickscli/bin/activate

    ![Screenshot shows the virtual environment activated within Cloud Shell](media/azure-cloud-shell-virtual-environment.png 'Virtual Environment activated')

6.  Install the Databricks CLI.

    pip install databricks-cli

7.  Set up authentication with Databricks by using the access token that you must have created, listed as part of prerequisites. Use the following command:

    databricks configure --token

    You will receive the following prompts:

    - First, you are prompted to enter the Databricks host. Enter the value in the format `https://eastus2.azuredatabricks.net`. Here, **East US 2** is the Azure region where you created your Azure Databricks workspace.

    - Next, you are prompted to enter a token. Enter the token that you created earlier.

Once you complete these steps, you can start using Databricks CLI from Azure Cloud Shell.

Here is a screenshot of successful CLI commands run in Azure Cloud Shell:

![Screenshot of successful CLI commands in Cloud Shell](media/cli-commands-in-azure-cloud-shell.png 'Successful CLI commands in Cloud Shell')

## Workspace CLI Examples

The implemented commands for the Workspace CLI can be listed by running `databricks workspace -h`.

Commands are run by appending them to `databricks workspace`.

```
$ databricks workspace -h
Usage: databricks workspace [OPTIONS] COMMAND [ARGS]...

  Utility to interact with the Databricks Workspace. Workspace paths must be
  absolute and be prefixed with `/`.

Options:
  -v, --version
  -h, --help     Show this message and exit.

Commands:
  delete      Deletes objects from the Databricks...
  export      Exports a file from the Databricks workspace...
  export_dir  Recursively exports a directory from the...
  import      Imports a file from local to the Databricks...
  import_dir  Recursively imports a directory from local to...
  list        List objects in the Databricks Workspace
  ls          List objects in the Databricks Workspace
  mkdirs      Make directories in the Databricks Workspace.
  rm          Deletes objects from the Databricks...
```

### Listing Workspace Files

```
$ databricks workspace ls /Users/example@databricks.com
Usage Logs ETL
Common Utilities
guava-21.0
```

### Importing a local directory of notebooks

The `databricks workspace import_dir` command will recursively import a directory from the local filesystem to the Databricks workspace. Only directories and files with the extensions of `.scala`, `.py`, `.sql`, `.r`, `.R` are imported.

When imported, these extensions will be stripped off the name of the notebook.

To overwrite existing notebooks at the target path, the flag `-o` must be added.

```
$ tree
.
├── a.py
├── b.scala
├── c.sql
├── d.R
└── e
```

```
$ databricks workspace import_dir . /Users/example@databricks.com/example
./a.py -> /Users/example@databricks.com/example/a
./b.scala -> /Users/example@databricks.com/example/b
./c.sql -> /Users/example@databricks.com/example/c
./d.R -> /Users/example@databricks.com/example/d
```

```
$ databricks workspace ls /Users/example@databricks.com/example -l
NOTEBOOK   a  PYTHON
NOTEBOOK   b  SCALA
NOTEBOOK   c  SQL
NOTEBOOK   d  R
DIRECTORY  e
```

### Exporting a workspace directory to the local filesystem

Similarly, it is possible to export a directory of notebooks from the Databricks workspace to the local filesystem. To do this, the command is simply:

```
$ databricks workspace export_dir /Users/example@databricks.com/example .
```

## DBFS CLI Examples

The implemented commands for the DBFS CLI can be listed by running `databricks fs -h`.

Commands are run by appending them to `databricks fs` and all dbfs paths should be prefixed with `dbfs:/`. To make the command less verbose, we've gone ahead and aliased `dbfs` to `databricks fs`.

```
$ databricks fs -h
Usage: databricks fs [OPTIONS] COMMAND [ARGS]...

  Utility to interact with DBFS. DBFS paths are all prefixed
  with dbfs:/. Local paths can be absolute or local.

Options:
  -v, --version
  -h, --help     Show this message and exit.

Commands:
  configure
  cp         Copy files to and from DBFS.
  ls         List files in DBFS.
  mkdirs     Make directories in DBFS.
  mv         Moves a file between two DBFS paths.
  rm         Remove files from dbfs.
```

### Copying a file to DBFS

```
dbfs cp test.txt dbfs:/test.txt
# Or recursively
dbfs cp -r test-dir dbfs:/test-dir
```

### Copying a file from DBFS

```
dbfs cp dbfs:/test.txt ./test.txt
# Or recursively
dbfs cp -r dbfs:/test-dir ./test-dir
```

## Jobs CLI Examples

The implemented commands for the jobs CLI can be listed by running `databricks jobs -h`. Job run commands are handled by `databricks runs -h`.

```
$ databricks jobs -h
Usage: databricks jobs [OPTIONS] COMMAND [ARGS]...

  Utility to interact with jobs.

  This is a wrapper around the jobs API
  (https://docs.databricks.com/api/latest/jobs.html). Job runs are handled
  by ``databricks runs``.

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  create   Creates a job.
  delete   Deletes the specified job.
  get      Describes the metadata for a job.
  list     Lists the jobs in the Databricks Job Service.
  reset    Resets (edits) the definition of a job.
  run-now  Runs a job with optional per-run parameters.
```

```
$ databricks runs -h
Usage: databricks runs [OPTIONS] COMMAND [ARGS]...

  Utility to interact with job runs.

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  cancel  Cancels the run specified.
  get     Gets the metadata about a run in json form.
  list    Lists job runs.
  submit  Submits a one-time run.
```

### Listing and finding jobs

The `databricks jobs list` command has two output formats: `JSON` and `TABLE`. The `TABLE` format is output by default and returns a two column table (job ID, job name).

To find a job by name:

```
databricks jobs list | grep "JOB_NAME"
```

### Copying a job

This example requires the program `jq`. See the [jq section](#jq) for more details.

```
SETTINGS_JSON=$(databricks jobs get --job-id 284907 | jq .settings)
# JQ Explanation:
#   - peek into top level `settings` field.
databricks jobs create --json "$SETTINGS_JSON"
```

### Deleting "Untitled" Jobs

```
databricks jobs list --output json | jq '.jobs[] | select(.settings.name == "Untitled") | .job_id' | xargs -n 1 databricks jobs delete --job-id
# Explanation:
#   - List jobs in JSON.
#   - Peek into top level `jobs` field.
#   - Select only jobs with name equal to "Untitled"
#   - Print those job ID's out.
#   - Invoke `databricks jobs delete --job-id` once per row with the $job_id appended as an argument to the end of the command.
```

## Clusters CLI Examples

The implemented commands for the clusters CLI can be listed by running `databricks clusters -h`.

```
$ databricks clusters -h
Usage: databricks clusters [OPTIONS] COMMAND [ARGS]...

  Utility to interact with Databricks clusters.

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  create           Creates a Databricks cluster.
  delete           Removes a Databricks cluster given its ID.
  get              Retrieves metadata about a cluster.
  list             Lists active and recently terminated clusters.
  list-node-types  Lists possible node types for a cluster.
  list-zones       Lists zones where clusters can be created.
  restart          Restarts a Databricks cluster given its ID.
  spark-versions   Lists possible Databricks Runtime versions...
  start            Starts a terminated Databricks cluster given its ID.
```

### Listing runtime versions

```
databricks clusters spark-versions
```

### Listing node types

```
databricks clusters list-node-types
```

## Libraries CLI

You run library subcommands by appending them to `databricks libraries`.

```
$ databricks libraries -h
Usage: databricks libraries [OPTIONS] COMMAND [ARGS]...

  Utility to interact with libraries.

  This is a wrapper around the libraries API
  (https://docs.databricks.com/api/latest/libraries.html).

Options:
  -v, --version  [VERSION]
  -h, --help     Show this message and exit.

Commands:
  all-cluster-statuses  Get the status of all libraries.
  cluster-status        Get the status of all libraries for a specified
                        cluster.
  install               Install a library on a cluster.
  list                  Shortcut to `all-cluster-statuses` or `cluster-
                        status`.
  uninstall             Uninstall a library on a cluster.
```

### Install a JAR from DBFS

```
databricks libraries install --cluster-id $CLUSTER_ID --jar dbfs:/test-dir/test.jar
```

### List library statuses for a cluster

```
databricks libraries list --cluster-id $CLUSTER_ID
```

## Uploading a file to DBFS from Azure Cloud Shell

Follow the steps in the [Azure Cloud Shell portion of the Install section](#use-databricks-cli-from-azure-cloud-shell) above to configure and connect to the Databricks CLI in Azure Cloud Shell.

1.  Select the **Upload file** button on the Cloud Shell toolbar.

    ![Select the Upload file button on the Cloud Shell toolbar](media/upload-file-cloud-shell.png 'Upload file')

2.  Enter `ls` to ensure the file was uploaded to the Cloud Shell environment storage location.

3.  Enter the following to upload your file to the Azure Databricks DBFS:

    `dbfs cp yourfilename dbfs:/yourfilename`

4.  Enter `dbfs ls` to verify the file was uploaded to DBFS from Cloud Shell.

    ![Screenshot showing successful upload of file to DBFS from Cloud Shell](media/upload-file-from-cloud-shell-to-dbfs.png 'Upload file to DBFS from Cloud Shell')

## jq

Some Databricks CLI commands will output the JSON response from the API endpoint. Sometimes it can be useful to parse out parts of the JSON to pipe into other commands. For example, to copy a job definition, we must take the `settings` field of `/api/2.0/jobs/get` and use that as an argument to the `databricks jobs create` command.

In these cases, we recommend you to use the utility `jq`. MacOS users can install `jq` through Homebrew with `brew install jq`.

Reference the [jq documentation](https://stedolan.github.io/jq/) for more information on `jq`.
