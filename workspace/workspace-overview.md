# The Azure Databricks workspace

When talking about the Azure Databricks workspace, we refer to two different things. The first reference is the logical Azure Databricks environment in which clusters are created, data is stored (via DBFS), and in which the server resources are housed. The second reference is the more common one used within the context of Azure Databricks. That is the special root folder for all of your organization's Databricks assets, including notebooks, libraries, and dashboards.

The first aspect is covered in detail within the [architecture](../architecture/) articles. For now, we will focus on the root folder as seen within the Azure Databricks UI:

![Azure Databricks workspace within the UI](media/azure-databricks-workspace.png 'Azure Databricks workspace')

## Workspace fundamentals

As a developer, the workspace is where you will spend most of your time when using the Azure Databricks UI. This is where you will manage and use notebooks, libraries, and dashboards. Oftentimes, you will simply use notebooks within your own private home folder. This provides an isolated place to work on these artifacts without others making changes to your work, if [workspace access control](https://docs.azuredatabricks.net/administration-guide/admin-settings/workspace-acl.html) (Premium SKU) is activated in your account.

Workspace folders behave like folders on your desktop. They can contain notebooks, libraries, and other folders, and allow you to share them with other users. Workspaces are not connected to data and should not be used to store data. They're simply for you to store the notebooks and libraries that you use to operate on and manipulate your data with.

![Workspace folders](media/workspace-folders.png 'Workspace folders')

There are three types of folders you will use, and each has their own purpose:

### Workspace folder

This is the special root folder for all Azure Databricks assets within your organization. It contains links to documentation, release notes, training and tutorials, the Shared folder, and the Users folder.

It is possible to create folders here, which is something you would likely do when you have notebooks that you want to run from other notebooks, or from an external process, like Azure Data Factory.

For example, we have created a folder within the Workspace folder named "diamonds". Within this folder is a notebook named "diamonddataframe". It simply creates a DataFrame from the diamonds data set:

```python
dataPath = "/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv"
diamonds = spark.read.format("csv")\
  .option("header","true")\
  .option("inferSchema", "true")\
  .load(dataPath)
```

We have another notebook named "demo" that runs the diamonddataframe notebook and accesses the `diamonds` DataFrame and displays the data in a bar chart visualization (the `%run` magic needs to be the only command in its cell):

```python
%run /diamonds/diamonddataframe
```

```python
display(diamonds)
```

Notice that the path to the diamonddataframe notebook is `/diamonds/diamonddataframe`. This is because the "diamonds" folder is located at the root, or Workspace folder. Here's a screenshot of the diamonds folder that was created in the Workspace folder, with the diamonddataframe notebook within. The screenshot also shows the demo notebook, which is located within a private user folder, that runs the diamonddataframe notebook:

![This screenshot shows a notebook running another notebook that is located within a folder located within the Workspace folder](media/run-notebook.png 'Running a notebook from within another')

Within the "adf" folder, we have a notebook that executes batch predictions against a flight and weather data set, using a trained Decision Tree model.

The trained machine learning model was created and trained within a separate notebook, and saved to DBFS. This notebook loads the trained model from DBFS to conduct batch scoring on demand (`model = PipelineModel.load("/dbfs/FileStore/models/pipelineModel")`.

![Screenshot showing the BatchScore notebook stored in the adf folder](media/adf-batchscore-notebook.png 'BatchScore notebook stored in adf folder')

The data is located within Azure Storage, which is copied there as part of a data pipeline orchestrated by Azure Data Factory v2. The tail end of the pipeline includes a step that operationalizes the machine learning model by running this notebook with a Databricks Notebook Activity. The notebook path for this activity is set to **/adf/BatchScore**:

![The path to the BatchScore notebook is set within the Databricks Notebook Activity settings](media/adf-databricks-notebook-activity-path.png 'Notebook path within the Databricks Notebook Activity settings')

The Azure Data Factory design surface shows the file copy activity linked to the Databricks Notebook activity that executes the BatchScore notebook to conduct the batch ML scoring:

![Azure Data Factory design surface shows the file copy activity linked to the Databricks Notebook activity](media/adf-design-surface.png 'file copy activity linked to Databricks Notebook activity')

### Shared folder

The shared folder is specifically meant for sharing objects across your organization. This is the area you want to create notebooks, libraries, and dashboards that you wish to give others access to and collaborate. This is especially true when the administrator has enabled [workspace access control](https://docs.azuredatabricks.net/administration-guide/admin-settings/workspace-acl.html), which locks other users out of private user folders.

As we did within the Workspace folder we created above, we've cloned the diamonddataframe notebook into the Shared folder:

![diamonddataframe notebook cloned to the Shared folder](media/workspace-shared-folder.png 'Shared folder')

We are able to run this notebook from another notebook located elsewhere, using the `%run` magic we used previously. Only this time, the path has changed to `/Shared/...`:

```python
%run /Shared/diamonddataframe
```

### User folders

User folders are created for each user who has been added to the Azure Databricks workspace. As mentioned earlier, if you have [workspace access control](https://docs.azuredatabricks.net/administration-guide/admin-settings/workspace-acl.html) activated, which is part of the Premium SKU, then all objects within a user folder are private to that user by default. Otherwise, any user can see the contents of any other user's folder.

One item of note is, if you remove a user, the user folder is _not deleted_.

## Managing your workspace programmatically

The UI for the Workspace is very straightforward and easy to work with. But what if you want to script out certain tasks? For instance, you would like to manipulate the folder structure or import or export a collection of notebooks in one shot? The good news is that Workspace actions such as these can be performed programmatically through the [Databricks CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#databricks-cli) and [Workspace API](https://docs.azuredatabricks.net/api/latest/workspace.html#workspace-api).
