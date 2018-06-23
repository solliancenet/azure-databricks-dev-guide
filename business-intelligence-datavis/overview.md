# Business intelligence and data visualization

The Apache Spark engine, the underlying platform for Azure Databricks, helps you ingest, manipulate, and display raw data with incredible speed. Increasingly, business leaders recognize the importance of data-driven decision making, and how having a data-oriented mindset can provide them a competitive advantage. The fact that you are using Azure Databricks to begin with is likely the result of this realization, by you or a decision maker within your organization.

## What is business intelligence, and why should I care?

Business intelligence, oftentimes referred to as BI, is simply a process for collecting and analyzing data with the goal of extracting actionable insights from that data. These insights help inform business decisions that are backed by analysis of data that is most often represented by visualizations.

## What are data visualizations in this context?

Data visualization is a visual snapshot of the data, that can be either static or interactive. It is usually in the form of a chart, map, or graph that helps one process the underlying raw data in a much more rapid and natural way. Our brains are wired to process visual information. Studies show that approximately 90% of the information our brains process and store is visual. This is due to how quickly and efficiently our brains can interpret visual data compared to other forms, such as text. In fact, studies show that we process images 60,000 times faster than text. This is why analysts and business decision makers rely so heavily on visuals. They are the most effective and efficient way to process and share raw data.

Here's an example we will use to drive home the point. Below is a subset of income tax data extracted from flat text files and stored in a Spark DataFrame. There are millions of rows of data, broken down by state.

| state | zipcode | sngle_returns | joint_returns | numdep | total_income_amount | taxable_interest_amount | net_capital_gains | biz_net_income |
| ----- | ------- | ------------- | ------------- | ------ | ------------------- | ----------------------- | ----------------- | -------------- |
| AL    | 0       | 488030        | 122290        | 571240 | 11444868            | 77952                   | 23583             | 824487         |
| AL    | 0       | 195840        | 155230        | 383240 | 17810952            | 81216                   | 54639             | 252768         |
| AL    | 0       | 72710         | 146880        | 189340 | 16070153            | 80627                   | 84137             | 259836         |
| AL    | 0       | 24860         | 126480        | 134370 | 14288572            | 71086                   | 105947            | 214668         |
| AL    | 0       | 16930         | 168170        | 177800 | 26053920            | 149150                  | 404166            | 567439         |
| AL    | 0       | 3530          | 42190         | 48270  | 20752068            | 271416                  | 1569967           | 822565         |
| AL    | 3500    | 950           | 260           | 710    | 19851               | 183                     | 4                 | 1657           |
| AL    | 3500    | 590           | 410           | 860    | 49338               | 172                     | 54                | 788            |
| AL    | 3500    | 290           | 490           | 620    | 56170               | 185                     | 139               | 584            |
| AL    | 3500    | 90            | 490           | 530    | 52977               | 89                      | 173               | 339            |
| AL    | 3500    | 40            | 460           | 450    | 64329               | 205                     | 709               | 1720           |
| AL    | 3500    | 0             | 40            | 30     | 15359               | 130                     | 0                 | 0              |
| AL    | 3500    | 800           | 190           | 860    | 19011               | 135                     | -1                | 840            |
| AL    | 3500    | 360           | 300           | 850    | 36996               | 87                      | 42                | -193           |
| AL    | 3500    | 140           | 230           | 340    | 28664               | 95                      | 22                | 233            |
| AL    | 3500    | 50            | 170           | 200    | 19583               | 38                      | 62                | -48            |
| AL    | 3500    | 0             | 160           | 170    | 25136               | 67                      | 48                | 84             |
| AL    | 3500    | 0             | 0             | 0      | 0                   | 0                       | 0                 | 0              |
| AL    | 3500    | 250           | 110           | 200    | 5515                | 26                      | 0                 | 543            |
| AL    | 3500    | 110           | 150           | 230    | 11906               | 32                      | 0                 | 188            |
| AL    | 3500    | 40            | 130           | 160    | 11854               | 17                      | 6                 | 49             |
| AL    | 3500    | 20            | 110           | 130    | 10911               | 6                       | 0                 | -33            |
| AL    | 3500    | 0             | 130           | 140    | 18399               | 14                      | 0                 | 0              |
| AL    | 3500    | 0             | 0             | 0      | 0                   | 0                       | 0                 | 0              |
| AL    | 3500    | 2720          | 620           | 2360   | 52779               | 260                     | 18                | 5331           |
| AL    | 3500    | 1150          | 830           | 2100   | 98951               | 306                     | 218               | 3335           |
| AL    | 3500    | 530           | 1050          | 1610   | 116810              | 261                     | 382               | 2197           |

For context, here is the data source:

```javascript
val taxes2013 = spark
  .read.format("csv")
  .option("header", "true")
  .load("dbfs:/databricks-datasets/data.gov/irs_zip_code_data/data-001/2013_soi_zipcode_agi.csv")
```

What is the quickest way to determine, from this data, the average income for each state? Let's display this same data on a map that has each state shaded according to the average income for that state. With a glance it is easy to determine which states have a higher income and the overall spread of income across the United States.

![Screenshot of a map of the United States, with each state shaded to indicate the average income range](media/average-us-income.png 'Average US income')

This map visualization is one of the default options that come with Azure Databricks out of the box. All notebooks, regardless of their language, support Databricks visualization using the `display` function. The display function includes support for visualizing multiple data types. As opposed to the handful of basic visualizations that other notebook engines provide, Azure Databricks includes several out of the box that you traditionally would need to rely on an external library such as `matplotlib` to obtain. However, if you wish to use external libraries to augment the default ones, you are free to do so.

![](../overview/media/azure-databricks-visualizations.png)

These visualizations are suitable for many situations, but sometimes you need to pull in a 3rd-party library or package. Here's an example of referencing the `d3a` package. You can find out more about using 3rd-party and custom packages in [Creating and Using 3rd-party and Local Libraries](../libraries/README.md). This example also uses the 3rd-party `graphframes` package to provide DataFrame-based graphs.

```javascript
// On-time and Early Arrivals
import d3a._

graphs.force(
  height = 800,
  width = 1200,
  clicks = sql("select src, dst as dest, count(1) as count from departureDelays_geo where delay <= 0 group by src, dst").as[Edge])
```

The output of the code above is an interactive visualization displayed right in the notebook. You can hover over a circle on the map to view the vertices and edges that represent the relationship between origin and destination airports.

![Screenshot of an interactive map displaying the relationships between origin and destination airports, using the d3a package](media/d3a-interactive-map.png 'Interactive d3a map')

## Business intelligence and Azure Databricks

While Azure Databricks is built for collaboration, making it easy for data scientists, engineers, architects, and developers to share and work with data, there are times when you need to go outside the environment to give access to others to your data. A good example of this is business leaders who want to view an executive summary, or perhaps business analysts who want to create ad-hoc reports but do not have the ability or desire to write code. In other cases, you want visualizations that you can embed in a web page that automatically refreshes as new data becomes available. In all of these cases, an external BI system makes a lot of sense. One option is to use [Power BI](https://powerbi.microsoft.com), Microsoft's premier suite of business analytics tools. With it, you can create individual reports from datasets that draw from Azure Databricks' tables, and static or live (auto-refreshes data) dashboards, all of which can be consumed on the web or across mobile devices. Users can create their own personalized dashboards, and you can embed any dashboard or report into a web application of your choosing.

What you gain from using a BI system such as Power BI, is a way for your users to work with the data you provide through Azure Databricks, without them needing to ever log in to the workspace. You can perform all of the data wrangling and processing within your Databricks notebooks, store the data into tables, then directly connect those tables to Power BI. Within Power BI, you have the option to copy data from those tables and store it within Power BI, or have it dynamically link to the tables so Azure Databricks provides the data in real-time.

Here is a walkthrough on how to connect Azure Databricks tables to Power BI and build some powerful visualizations without the end user writing any code.

### Scenario

In this scenario, a travel agency has created a machine learning model in Azure Databricks, using Spark Machine Language Library (MLlib). They regularly batch process scheduled flight data, joined with weather predictions, by cleaning and transforming the combined data sets and scoring the output with a trained machine learning model. The last step in the data pipeline is to create a global table that summarizes the scored flight delay predictions. They compare this data to historical flight delay data as well, and regularly retrain their model. In this case, they want to analyze scheduled flights for their customers and visualize the likelihood of flight delays, as well as the flights that are most often delayed given the time of year and weather conditions.

Here is a screenshot of the data they want to use in Power BI:

![Screenshot of the raw flight delay data](media/raw-scored-flight-delay-data.png 'Raw flight delay data')

The next step is to save this query output to a global table.

```python
summary = spark.sql("SELECT  OriginAirportCode, Month, DayofMonth, CRSDepHour, Sum(prediction) NumDelays,     CONCAT(Latitude, ',', Longitude) OriginLatLong FROM scoredflights s INNER JOIN airport_code_location_lookup_clean a ON s.OriginAirportCode = a.Airport GROUP BY OriginAirportCode, OriginLatLong, Month, DayofMonth, CRSDepHour  Having Sum(prediction) > 1 ORDER BY NumDelays DESC")

summary.write.mode("overwrite").saveAsTable("flight_delays_summary")
```

### Obtaining the JDBC connection string to your Azure Databricks cluster

Before you begin, you must first obtain the JDBC connection string to your Azure Databricks cluster.

1.  In Azure Databricks, go to Clusters and select your cluster.

1.  On the cluster edit page, scroll down and select the JDBC/ODBC tab.

    ![Select the JDBC/ODBC tab](media/databricks-power-bi-jdbc.png 'JDBC strings')

1.  On the JDBC/ODBC tab, copy and save the JDBC URL.

    - Construct the JDBC server address that you will use when you set up your Spark cluster connection in Power BI Desktop.

    - Take the JDBC URL that you copied and saved in step 3 and do the following:

    - Replace jdbc:hive2 with https.

    - Remove everything in the path between the port number and sql, retaining the components indicated by the boxes in the image below.

    ![Select the parts to create the Power BI connection string](media/databricks-power-bi-spark-address-construct.png 'Construct Power BI connection string')

    - In our example, the server address would be:

    <https://eastus.azuredatabricks.net:443/sql/protocolv1/o/1707858429329790/0614-124738-doubt405> or <https://eastus.azuredatabricks.net:443/sql/protocolv1/o/1707858429329790/lab> (if you choose the aliased version)

### Connect to Azure Databricks using Power BI Desktop

1.  Install and launch [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/).

1.  When Power BI Desktop opens, you will need to enter your personal information, or Sign in if you already have an account.

    ![The Power BI Desktop Welcome page displays.](media/image177.png 'Power BI Desktop Welcome page')

1.  Select Get data on the screen that is displayed next. ![On the Power BI Desktop Sign in page, in the left pane, Get data is selected.](media/image178.png 'Power BI Desktop Sign in page')

1.  Select **Other** from the left, and select **Spark (Beta)** from the list of available data sources.

    ![In the left pane of the Get Data page, Other is selected. In the right pane, Spark (Beta) is selected.](media/pbi-desktop-get-data.png 'Get Data page')

1.  Select **Connect**.

1.  You will receive a prompt warning you that the Spark connector is still in preview. Select **Continue**.

    ![A warning reminds you that the app is still under development.](media/image180.png 'Preview connector warning')

1.  On the next screen, you will be prompted for your Spark cluster information.

1.  Paste the JDBC connection string you constructed a few steps ago into the **Server** field.

1.  Select the **HTTP** protocol.

1.  Select **DirectQuery** for the Data Connectivity mode, and select **OK**. This option will offload query tasks to the Azure Databricks Spark cluster, providing near-real time querying.

    ![Configure your connection to the Spark cluster](media/pbi-desktop-connect-spark.png 'Spark form')

1.  Enter your credentials on the next screen as follows.

    a. User name: **token**

    b. Password: Create a new [personal access token](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management). **Paste the new token here**.

    ![Enter "token" for the user name and paste user token into the password field](media/pbi-desktop-login.png 'Enter credentials')

1.  Select **Connect**.

1.  In the Navigator dialog, check the box next to **flight_delays_summary**, and select **Load**.

    ![In the Navigator dialog box, in the left pane under Display Options, the check box for flight_delays_summary is selected. In the right pane, the table of flight delays summary information displays.](media/pbi-desktop-select-table-navigator.png 'Navigator dialog box')

1.  It will take several minutes for the data to load into the Power BI Desktop client.

### Create Power BI report

1.  Once the data finishes loading, you will see the fields appear on the far right of the Power BI Desktop client window.

    ![Power BI Desktop fields](media/pbi-desktop-fields.png 'Power BI Desktop Fields')

1.  From the Visualizations area, next to Fields, select the Globe icon to add a Map visualization to the report design surface.

    ![On the Power BI Desktop Visualizations palette, the globe icon is selected.](media/image187.png 'Power BI Desktop Visualizatoins palette')

1.  With the Map visualization still selected, drag the **OriginLatLong** field to the **Location** field under Visualizations. Then Next, drag the **NumDelays** field to the **Size** field under Visualizations.

    ![In the Fields column, the check boxes for NumDelays and OriginLatLong are selected. An arrow points from OriginLatLong in the Fields column, to OriginLatLong in the Visualization's Location field. A second arrow points from NumDelays in the Fields column, to NumDelays in the Visualization's Size field.](media/pbi-desktop-configure-map-vis.png 'Visualizations and Fields columns')

1.  You should now see a map that looks similar to the following (resize and zoom on your map if necessary):

    ![On the Report design surface, a Map of the United States displays with varying-sized dots over different cities.](media/pbi-desktop-map-vis.png 'Report design surface')

1.  Unselect the Map visualization by clicking on the white space next to the map in the report area.

1.  From the Visualizations area, select the **Stacked Column Chart** icon to add a bar chart visual to the report's design surface.

    ![The stacked column chart icon is selected on the Visualizations palette.](media/image190.png 'Visualizations palette')

1.  With the Stacked Column Chart still selected, drag the **DayofMonth** field and drop it into the **Axis** field located under Visualizations.

1.  Next, drag the **NumDelays** field over, and drop it into the **Value** field.

    ![In the Fields column, the check boxes for NumDelays and DayofMonth are selected. An arrow points from NumDelays in the Fields column, to NumDelays in the Visualization's Axis field. A second arrow points from DayofMonth in the Fields column, to DayofMonth in the Visualization's Value field.](media/pbi-desktop-configure-stacked-vis.png 'Visualizations and Fields columns')

1.  Grab the corner of the new Stacked Column Chart visual on the report design surface, and drag it out to make it as wide as the bottom of your report design surface. It should look something like the following.

    ![On the Report Design Surface, under the map of the United States with dots, a stacked bar chart displays.](media/pbi-desktop-stacked-vis.png 'Report Design Surface')

1.  Unselect the Stacked Column Chart visual by clicking on the white space next to the map on the design surface.

1.  From the Visualizations area, select the Treemap icon to add this visualization to the report.

    ![On the Visualizations palette, the Treemap icon is selected.](media/image193.png 'Visualizations palette')

1.  With the Treemap visualization selected, drag the **OriginAirportCode** field into the **Group** field under Visualizations.

1.  Next, drag the **NumDelays** field over, and drop it into the **Values** field.

    ![In the Fields column, the check boxes for NumDelays and OriginAirportcode are selected. An arrow points from NumDelays in the Fields column, to NumDelays in the Visualization's Values field. A second arrow points from OriginAirportcode in the Fields column, to OriginAirportcode in the Visualization's Group field.](media/pbi-desktop-config-treemap-vis.png 'Visualizations and Fields columns')

1.  Grab the corner of the Treemap visual on the report design surface, and expand it to fill the area between the map and the right edge of the design surface. The report should now look similar to the following.

    ![The Report design surface now displays the map of the United States with dots, a stacked bar chart, and a Treeview.](media/pbi-desktop-full-report.png 'Report design surface')

1.  You can cross filter any of the visualizations on the report by clicking on one of the other visuals within the report, as shown below. (This may take a few seconds to change, as the data is loaded.)

    ![The map on the Report design surface is now zoomed in on the northeast section of the United States, and the only dot on the map is on Chicago. In the Treeview, all cities except ORD are grayed out. In the stacked bar graph, each bar is now divided into a darker and a lighter color, with the darker color representing the airport.](media/pbi-desktop-full-report-filter.png 'Report design surface')

1.  You can save the report, by clicking Save from the File menu, and entering a name and location for the file.

1.  Once this report is saved, you can import it into the Power BI online service, so others can access the data and the reports can be embedded elsewhere.
