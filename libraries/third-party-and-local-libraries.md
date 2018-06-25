# Creating and using 3rd-party and local libraries

Azure Databricks clusters include a number of Python, R, Java, and Scala libraries that are pre-installed as part of the Databricks Runtime. View the [Databricks Runtime release notes](https://docs.azuredatabricks.net/release-notes/cluster-images/index.html) for your cluster's Databricks Runtime version to view the list of installed libraries. However, despite the long list of pre-installed libraries, you may encounter cases where you need to either add a third-party library or locally-built code to one or more cluster execution environments. The easiest way to do this in Azure Databricks is to create a new [library](https://docs.azuredatabricks.net/user-guide/libraries.html#id1). Libraries can be written in Python, Java, Scala, and R. You can create and manage libraries using the UI, [Databricks CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#databricks-cli), or by invoking the [Libraries API](https://docs.azuredatabricks.net/api/latest/libraries.html#library-api). We will show examples of using each of these options in the sections below.

## Library basics

When you create a library, you can choose the destination for the library within the Workspace, just like you do when creating a notebook or folder. If you want the library to be shared by all users of your Workspace, create the library within the **Shared** folder. This is also true for notebooks and dashboards that you create within the workspace.

![Screenshot showing library being created in the Shared folder](media/create-shared-library.png 'Create shared library')

On the other hand, you may only want the library to be available to your user or someone else. To do this, simply create it within the associated User folder instead.

![Screenshot showing library being created in a User folder](media/create-user-library.png 'Create user library')

Additional things to keep in mind about libraries are:

- Libraries are immutable. They can only be created and deleted.
- To completely delete a library from a cluster you must restart the cluster.
- Azure Databricks stores libraries that you upload in the [FileStore](https://docs.azuredatabricks.net/user-guide/advanced/filestore.html#filestore).
- After you attach a library to a cluster, to use the library you must reattach any notebooks using the cluster.

## Creating a library using the UI

In the [Business Intelligence and Data Visualization article](../business-intelligence-datavis/bi-and-datavis.md#what-are-data-visualizations-in-this-context), we referenced the 3rd-party `d3a` Maven package. We did not go over the details on how to create a library to add the package to the cluster's execution environment. Follow the steps below to add the `d3a` package in a new shared library:

1.  Go to the Workspace folder and right-click the **Shared** folder. Select Create -> Library.

    ![Right-click the Shared folder and select Create then Library](media/create-library-menu.png 'Create shared library')

1.  In the New Library form, select **Maven Coordinates** as the source, then select **Search Spark Packages and Maven Central**. Alternately, if you know the exact Maven coordinate, enter it within the Coordinate field. Maven coordinates are in the form groupId:artifactId:version; for example, `graphframes:graphframes:0.5.0-spark2.1-s_2.11`.

    ![Select Maven Coordinate as the source and select search](media/new-library-maven.png 'New Library')

1.  Your search results should appear within the Search Packages dialog. **Note:** Sometimes you will need to reenter your search in the search box on top of this dialog. The select list to the right will allow you to narrow your search to Spark Packages or Maven Central. In this case, select **Spark Packages**. The Releases select list allows you to select the package release that is compatible with your Spark version. Select the latest release, then click **+ Select** under the Options column.

    ![Screenshot showing the graphframes searh results](media/new-library-maven-search-packages.png 'Search Packages')

1.  After selecting the package, the Search Packages dialog will close. You should see the graphframes coordinate listed, based on your selection. The Advanced Options allow you to select a specific repository URL you would like to use to obtain the package as an alternative, such as `https://oss.sonatype.org/content/repositories`. The Excludes field enables you to exclude specific dependencies from the selected package by providing the `groupId` and the `artifactId` of the dependencies that you want to exclude; for example, log4j:log4j. Select **Create Library**.

    ![Screenshot showing selected graphframes Maven artifact](media/new-library-maven-create-library.png 'New Library')

1.  The Library details are displayed after creating the new library. It is here that you can view its artifacts, including dependencies, delete the library, and select which clusters to which the library should be attached. Check the **Attach automatically to all clusters** checkbox to attach this library to all existing clusters and any new clusters that are created in the future.

    ![Screenshot displaying the new library details for the installed graphframes Maven artifact](media/new-library-maven-details.png 'Library details')

If you have any notebooks attached to a cluster to which you attached the library, you must first detach then reattach that notebook to the cluster for it to be able to access the library.

### Use the library within a notebook

Now that the required `d3a` Maven package has been attached to the cluster through the new library, it can be used within a notebook as follows:

```javascript
import d3a._

graphs.force(
  height = 800,
  width = 1200,
  clicks = sql("select src, dst as dest, count(1) as count from departureDelays_geo where delay <= 0 group by src, dst").as[Edge])
```

The `import` command is able to locate the package because the files have been uploaded and references to it attached to the cluster. You must remember to detach and reattach the cluster to the notebook if you added the library after the notebook had been attached to the cluster.

### Upload a Java JAR or Scala JAR

If you wanted to install a Java or Scala JAR, also referred to as a local library, follow these steps in the New Library form that appears starting with step 2 above:

1.  In the Source drop-down list, select **Upload Java/Scala JAR**.

1.  Enter a library name.

1.  Click and drag your JAR to the JAR File text box.

    ![Screenshot showing New Library form with JAR file uploaded](media/upload-jar.png 'New Library')

1.  Select **Create Library**. The library detail screen will display.

1.  In the Attach column, select the clusters to attach the library to, or select Attach automatically to all clusters.

### Upload a Python PyPI package or Python Egg

You can also install a PyPI package or upload a Python Egg. To do so, follow these steps in the New Library form:

1.  In the Source drop-down list, select **Upload Python Egg or PyPI**.

    1.  If installing a PyPI package, enter a PyPI package name and select **Install Library**. The library detail screen will display.

        ![Screenshot showing isntalling a PyPi package](media/new-library-python.png 'New Library')

        **Note:** PyPI has a specific format for installing specific versions of libraries. For example, to install a specific version of simplejson, use this format for the library: `simplejson==3.15.0`.

    1.  If installing a Python Egg:
        - Enter a **Library Name**.
        - Click and drag the egg and optionally the documentation egg to the **Egg File** box.
        - Select **Create Library**. The library detail screen will display.

1.  In the Attach column, select the clusters to attach the library to, or select Attach automatically to all clusters.

### Upload a CRAN library

R has a [rich ecosystem of packages](https://cran.r-project.org/web/packages/available_packages_by_name.html) called the Comprehensive R Archive Network, or CRAN. To install CRAN libraries that you can use on your Azure Databricks clusters, follow these steps in the New Library form:

1.  In the Source drop-down list, select **R Library**.

    ![Screenshot showing the New Library form and adding a CRAN library](media/cran-library.png 'New Library')

1.  In the **Install from** drop-down list, CRAN-like Repository is the only option and is selected by default. This option covers both CRAN and bioconductor repositories.

1.  In the **Repository** field, enter the CRAN repository URL.

1.  In the **Package** field, enter the name of the package.

1.  Select **Create Library**. The library detail screen will display.

1.  In the Attach column, select the clusters to attach the library to, or select Attach automatically to all clusters.

## Install libraries using cluster node initialization scripts

Some libraries require lower-level configuration and cannot be uploaded using the methods described in this article. To install these libraries, you can write a custom UNIX script that runs at cluster creation time, using cluster node initialization scripts. An initialization script is a shell script that runs during startup for each cluster new _before_ the Spark driver or worker JVM starts.

> [!IMPORTANT]
> To install Python packages, use the Azure Databricks pip binary located at /databricks/python/bin/pip to ensure that Python packages install into Databricks Python virtual environment rather than the system Python environment. For example, /databricks/python/bin/pip install <packagename>.

If you want to install libraries for all clusters, then you need to use a **global init script**. You do this by storing the scripts in the `dbfs:/databricks/init/` directory.

Conversely, if you only want to install libraries to a specific cluster, then use **cluster-specific init scripts**. These scripts also reside within the `dbfs:/databricks/init/` directory, but under subdirectories named the same as the cluster name. For instance, if your cluster name is `lab`, then init scripts for that cluster would be stored within `dbfs:/databricks/init/lab`. You must create the directory if it does not already exist.

As you can see from the file paths above, which are prefixed with `dbfs:`, all initialization scripts are created and managed from the [Databricks File System -DBFS](https://docs.azuredatabricks.net/user-guide/dbfs-databricks-file-system.html#dbfs).

**Things of note:** Any change to an init script will require a cluster restart to take effect. Also, if you are using cluster-specific init scripts, avoid spaces in your cluster names as they are used in the script and output paths.

### Install a library using a global init script

To install a library using a global init script, perform the following steps from a notebook:

1.  Create dbfs:/databricks/init/ if it doesn’t exist.

    ```python
    dbutils.fs.mkdirs("dbfs:/databricks/init/")
    ```

    You can display a list of existing global init scripts with the following:

    ```python
    display(dbutils.fs.ls("dbfs:/databricks/init/"))
    ```

1.  Create the init script for your library. In this case, we are installing PostgreSQL:

    ```python
    dbutils.fs.put("/databricks/init/postgresql-install.sh","""
    #!/bin/bash
    wget --quiet -O /mnt/driver-daemon/jars/postgresql-42.2.2.jar http://central.maven.org/maven2/org/postgresql/postgresql/42.2.2/postgresql-42.2.2.jar
    wget --quiet -O /mnt/jars/driver-daemon/postgresql-42.2.2.jar http://central.maven.org/maven2/org/postgresql/postgresql/42.2.2/postgresql-42.2.2.jar""", True)
    ```

Every time a cluster launches it will execute this append script.

### Install a library using a cluster-specific init script

To install a library for a specific cluster, perform the following steps from a notebook:

1.  Create dbfs:/databricks/init/ if it doesn’t exist.

    ```python
    dbutils.fs.mkdirs("dbfs:/databricks/init/")
    ```

1.  Configure a cluster name variable. This should be the name of the cluster you want to initialize with this script.

    ```python
    clusterName = "lab"
    ```

1.  Create a directory named `lab` (or your cluster name).

    ```python
    dbutils.fs.mkdirs("dbfs:/databricks/init/%s/"%clusterName)
    ```

1.  Create the init script for your library.

    ```python
    dbutils.fs.put("/databricks/init/lab/postgresql-install.sh","""
    #!/bin/bash
    wget --quiet -O /mnt/driver-daemon/jars/postgresql-42.2.2.jar http://central.maven.org/maven2/org/postgresql/postgresql/42.2.2/postgresql-42.2.2.jar
    wget --quiet -O /mnt/jars/driver-daemon/postgresql-42.2.2.jar http://central.maven.org/maven2/org/postgresql/postgresql/42.2.2/postgresql-42.2.2.jar""", True)
    ```

    Notice that the only difference in the init script here vs. the global init script is that the cluster-specific one includes the cluster name (in this case, `lab`) in the file path.

1.  Check to make sure the cluster-specific init script exists.

    ```python
    display(dbutils.fs.ls("dbfs:/databricks/init/%s/postgresql-install.sh"%clusterName))
    ```

    The output should look similar to the following:

    ![File list showing new cluster-specific init script](media/cluster-specific-init-script.png 'Cluster-specific init script')
