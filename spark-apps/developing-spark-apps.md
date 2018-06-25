# Developing Spark apps

Azure Databricks is the premiere place to run Spark workloads in Azure, and provides data scientists, data architects, data engineers, and developers with an environment that supports the entire data science process. Owing to their deep integration with Azure Databricks, Azure Databricks notebooks provide the most comprehensive notebook experience, supporting the entire data exploration and modeling life-cycle including purpose built features that support notebook authoring, dependency management, workspaces for organization and sharing of notebooks and artifacts, version control and automation.

Databricks notebooks support Python, R, SQL and Scala. Out of the box they include interactive and eye-pleasing visualizations, but also support the open source charting libraries like `ggplot`, `matplotlib`, `bokeh`, and `seaborn` that data scientists are already familiar with.

Databricks notebooks provide an enhanced experience for the execution of Spark code that includes things like live-updating status bars that report on distributed job progress, error highlighting that make it easier to see what went wrong both in the code and with clear error messages, and built-in support for running structured streaming jobs and showing updated data in tables and visualizations in near real-time and widgets that allow notebook authors to provide better interactive experiences when providing inputs to the notebook. Notebooks authored in Azure Databricks can be exported to a file and downloaded, or checked directly into version control repositories (such as GitHub).

When it comes to model training and evaluation, Spark in Azure Databricks includes Spark MLlib, a framework and library for machine learning. The Microsoft Machine Learning library for Spark (MMLSpark) is also available and provides scale-out deep learning algorithm support for the training and evaluation of predictive models in Spark.

For these reasons, we will demonstrate creating a Spark app using a notebook.

## Language choice

As stated above, Databricks notebooks support Python, R, SQL and Scala. The choice is yours as to which language, or languages, you can use to meet your challenges. In general terms, data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships within it. Those with a SQL background tend to use Spark SQL syntax in place of the fluent notation used by RDDs and DataFrames. Developers who are used to working with statically typed languages will tend to gravitate toward Scala, versus Python's dynamically typed language constructs.

Aside from language preference, what you choose as the primary language used in a notebook comes down to the type of problem you are trying to solve. In general, Scala is known to have a slight edge in performance over Python. One tactic when using Python as your primary language, is to create user-defined functions (UDFs) in Scala and reference those from Python. How and if this benefits your use case will depend on your performance tests if, again, performance is your primary concern.

R has such a long history and vast library of useful packages, thanks to CRAN and similar repositories, that it can be very beneficial using R to speed up data transformation tasks, as an example.

The great thing about Databricks notebooks is that you can **use several languages** within a single notebook, simply by specifying the language for a cell with the language magic command `%<language>` at the beginning of the cell. `%<language>` allows you to execute `<language>` code even if that notebook's primary language is not `<language>`. The supported magic commands are: `%python`, `%r`, `%scala`, and `%sql`. Additionally:

`%sh` allows you to execute shell code in your notebook. Add the -e option in order to fail this cell (and subsequently a job or a run all command) if the shell command does not success. By default, `%sh` alone will not fail a job even if the `%sh` command does not completely succeed. Only `%sh -e` will fail if the shell command has a non-zero exit status.

`%fs` allows you to use [Databricks Utilities](https://docs.azuredatabricks.net/user-guide/dev-tools/dbutils.html#dbutils) filesystem commands. Read more on the [Databricks File System - DBFS](https://docs.azuredatabricks.net/user-guide/dbfs-databricks-file-system.html#dbfs) pages.

**Note:** In our sample below, you will see language switching in action. Although the primary language of the notebook is Python, we oftentimes use SQL (`%sql`), and use R (`%r`) for some very helpful data transformation functions.

## The sample scenario

A travel agency wants to provide on-demand flight delay predictions to their customers, based on the origin and destination airports, travel date, and weather forecast. They have obtained decades of flight delay information and historic weather records at airports around the United States. They wish to import, analyze, join the data sets, and prepare the data so they can build and train a machine learning model. Ultimately, the model will be operationalized for on-demand and batch scoring. For now, they want to get started with data preparation and training a machine learning model.

**Note:** If you wish to follow along, you will need to first follow the instructions in the [Data Sources](../data-sources/data-sources-overview.md) article for importing data into Azure Databricks tables.

## Code walkthrough

If you are following the step-by-step instructions in this walkthrough, **create a new Python notebook** and attach a cluster to it.

### Prepare flight delay data

To start, let's import the Python libraries and modules we will use in this notebook.

```python
import pprint, datetime
from pyspark.sql.types import *
from pyspark.sql.functions import unix_timestamp
import math
from pyspark.sql import functions as F
```

First, let's execute the below command to make sure all three tables were created.

You should see an output like the following:

| database | tableName            | isTemporary |
| -------- | -------------------- | ----------- |
| default  | airport_code_loca... | false       |
| default  | flight_delays_wit... | false       |
| default  | flight_weather_wi... | false       |

```python
spark.sql("show tables").show()
```

Now execute a SQL query using the `%sql` magic to select all columns from flight_delays_with_airport_codes. By default, only the first 1,000 rows will be returned.

```sql
%sql
select * from flight_delays_with_airport_codes
```

You will see an output similar to the following:

![Screenshot showing the flight delays table output](media/flight-delays-table-output.png 'Flight delays table output')

Now let's see how many rows there are in the dataset.

```sql
%sql
select count(*) from flight_delays_with_airport_codes
```

Based on the `count` result, you can see that the dataset has a total of 2,719,418 rows (also referred to as examples in Machine Learning literature). Looking at the table output from the previous query, you can see that the dataset contains 20 columns (also referred to as features).

Because all 20 columns are displayed, you can scroll the grid horizontally. Scroll until you see the **DepDel15** column. This column displays a 1 when the flight was delayed at least 15 minutes and 0 if there was no such delay. In the model you will construct, you will try to predict the value of this column for future data.

Let's execute another query that shows us how many rows do not have a value in the DepDel15 column.

```sql
%sql
select count(*) from flight_delays_with_airport_codes where DepDel15 is null
```

Notice that the `count` result is 27444. This means that 27,444 rows do not have a value in this column. Since this value is very important to our model, we will need to eliminate any rows that do not have a value for this column.

Next, scroll over to the **CRSDepTime** column within the table view above. Our model will approximate departure times to the nearest hour, but departure time is captured as an integer. For example, 8:37 am is captured as 837. Therefore, we will need to process the CRSDepTime column, and round it down to the nearest hour. To perform this rounding will require two steps, first you will need to divide the value by 100 (so that 837 becomes 8.37). Second, you will round this value down to the nearest hour (so that 8.37 becomes 8).

Finally, we do not need all 20 columns present in the flight_delays_with_airport_codes dataset, so we will pare down the columns, or features, in the dataset to the 12 we do need.

Using `%sql` magic allows us view and visualize the data, but for working with the data in our tables, we want to take advantage of the rich optimizations provided by DataFrames. Let's execute the same query using Spark SQL, this time saving the query to a DataFrame.

```python
dfFlightDelays = spark.sql("select * from flight_delays_with_airport_codes")
```

Let's print the schema for the DataFrame.

```python
pprint.pprint(dfFlightDelays.dtypes)
```

You should see the following output:

```
[('Year', 'string'),
 ('Month', 'string'),
 ('DayofMonth', 'string'),
 ('DayOfWeek', 'string'),
 ('Carrier', 'string'),
 ('CRSDepTime', 'string'),
 ('DepDelay', 'string'),
 ('DepDel15', 'string'),
 ('CRSArrTime', 'string'),
 ('ArrDelay', 'string'),
 ('ArrDel15', 'string'),
 ('Cancelled', 'string'),
 ('OriginAirportCode', 'string'),
 ('OriginAirportName', 'string'),
 ('OriginLatitude', 'string'),
 ('OriginLongitude', 'string'),
 ('DestAirportCode', 'string'),
 ('DestAirportName', 'string'),
 ('DestLatitude', 'string'),
 ('DestLongitude', 'string')]
```

Notice that the DepDel15 and CRSDepTime columns are both `string` data types. Both of these features need to be numeric, according to their descriptions above. We will cast these columns to their required data types next.

### Perform data munging

To perform our data munging, we have multiple options, but in this case, we’ve chosen to take advantage of some useful features of R to perform the following tasks:

- Remove rows with missing values
- Generate a new column, named “CRSDepHour,” which contains the rounded down value from CRSDepTime
- Pare down columns to only those needed for our model

SparkR is an R package that provides a light-weight frontend to use Apache Spark from R. To use SparkR we will call `library(SparkR)` within a cell that uses the `%r` magic, which denotes the language to use for the cell. The SparkR session is already configured, and all SparkR functions will talk to your attached cluster using the existing session.

```r
%r
library(SparkR)

# Select only the columns we need, casting CRSDepTime as long and DepDel15 as int, into a new DataFrame
dfflights <- sql("SELECT OriginAirportCode, OriginLatitude, OriginLongitude, Month, DayofMonth, cast(CRSDepTime as long) CRSDepTime, DayOfWeek, Carrier, DestAirportCode, DestLatitude, DestLongitude, cast(DepDel15 as int) DepDel15 from flight_delays_with_airport_codes")

# Delete rows containing missing values
dfflights <- na.omit(dfflights)

# str(dfflights)

# Round departure times down to the nearest hour, and export the result as a new column named "CRSDepHour"
dfflights$CRSDepHour <- floor(dfflights$CRSDepTime / 100)

# Trim the columns to only those we will use for the predictive model
dfflightsClean = dfflights[, c("OriginAirportCode","OriginLatitude", "OriginLongitude", "Month", "DayofMonth", "CRSDepHour", "DayOfWeek", "Carrier", "DestAirportCode", "DestLatitude", "DestLongitude", "DepDel15")]

createOrReplaceTempView(dfflightsClean, "flight_delays_view")
```

When you examine the temporary `flight_delays_view`, you should see that the rows with missing data for DepDel15 have been removed, and that the other data transformations have taken place. Now save the contents of the temporary view into a new DataFrame.

```python
dfFlightDelays_Clean = spark.sql("select * from flight_delays_view")
```

### Export the prepared data to persistent a global table

There are two types of tables in Databricks.

- Global tables, which are accessible across all clusters
- Local tables, which are available only within one cluster

To create a global table, you use the `saveTableAs()` method. To create a local table, you would use either the `createOrReplaceTempView()` or `registerTempTable()` method.

The `flight_delays_view` table was created as a local table using `createOrReplaceTempView`, and is therefore temporary. Local tables are tied to the Spark/SparkSQL Context that was used to create their associated DataFrame. When you shut down the SparkSession that is associated with the cluster (such as shutting down the cluster) then local, temporary tables will disappear. If we want our cleansed data to remain permanently, we should create a global table.

Run the following to copy the data from the source location into a global table named `flight_delays_clean`.

```python
dfFlightDelays_Clean.write.mode("overwrite").saveAsTable("flight_delays_clean")
```

### Prepare the weather data

To begin, take a look at the `flight_weather_with_airport_code` data that was imported to get a sense of the data we will be working with.

```sql
%sql
select * from flight_weather_with_airport_code
```

You should see an output like the following:

![Screenshot of the weather table output select command](media/weather-table-output.png 'Weather table output')

Next, count the number of records so we know how many rows we are working with.

```sql
%sql
select count(*) from flight_weather_with_airport_code
```

Observe that this data set has 406,516 rows and 29 columns. For this model, we are going to focus on predicting delays using WindSpeed (in MPH), SeaLevelPressure (in inches of Hg), and HourlyPrecip (in inches). We will focus on preparing the data for those features.

Let's start out by taking a look at the **WindSpeed** column. You may scroll through the values in the table above, but reviewing just the distinct values will be faster.

```sql
%sql
select distinct WindSpeed from flight_weather_with_airport_code
```

Your output should represent the following:

| WindSpeed |
| --------- |
| 7         |
| 51        |
| 15        |
| 11        |
| 29        |
| 3         |
| 30        |
| 34        |
| 8         |
| 22        |
| 28        |
| 16        |
| 0         |
| 47        |
| null      |
| 43        |
| 5         |
| 31        |
| 18        |
| 27        |
| 17        |
| 26        |
| 46        |
| M         |
| 6         |
| 19        |
| 23        |
| 41        |
| 38        |
| 40        |
| 25        |
| 44        |
| 53        |
| 33        |
| 9         |
| 24        |
| 32        |
| 20        |
| 36        |
| 10        |
| 37        |
| 39        |
| 62        |
| 13        |
| 14        |
| 21        |
| 45        |

Observe that the values are all numbers, with the exception of some having `null` values and a string value of `M` for Missing. We will need to ensure that we remove any missing values and convert WindSpeed to its proper type as a numeric feature.

Next, let's take a look at the **SeaLevelPressure** column in the same way, by listing its distinct values.

```sql
%sql
select distinct SeaLevelPressure from flight_weather_with_airport_code
```

Click on the **SeaLevelPressure** column header to sort the values in ascending and then descending order. Observe that many of the features are of a numeric value (e.g., 29.96, 30.01, etc.), but some contain the string value of M for Missing. We will need to replace this value of "M" with a suitable numeric value so that we can convert this feature to be a numeric feature.

Finally, let's observe the **HourlyPrecip** feature by selecting its distinct values.

```sql
%sql
select distinct HourlyPrecip from flight_weather_with_airport_code
```

Click on the column header to sort the list and ascending and then descending order. Observe that this column contains mostly numeric values, but also `null` values and values with `T` (for Trace amount of rain). We need to replace T with a suitable numeric value and convert this to a numeric feature.

### Clean up weather data

To perform our data cleanup, we will execute a Python script, in which we will perform the following tasks:

- WindSpeed: Replace missing values with 0.0, and “M” values with 0.005
- HourlyPrecip: Replace missing values with 0.0, and “T” values with 0.005
- SeaLevelPressure: Replace “M” values with 29.92 (the average pressure)
- Convert WindSpeed, HourlyPrecip, and SeaLevelPressure to numeric columns
- Round “Time” column down to the nearest hour, and add value to a new column named “Hour”
- Eliminate unneeded columns from the dataset

Let's begin by creating a new DataFrame from the table. While we're at it, we'll pare down the number of columns to just the ones we need (AirportCode, Month, Day, Time, WindSpeed, SeaLevelPressure, HourlyPrecip).

```python
dfWeather = spark.sql("select AirportCode, cast(Month as int) Month, cast(Day as int) Day, cast(Time as int) Time, WindSpeed, SeaLevelPressure, HourlyPrecip from flight_weather_with_airport_code")
```

```python
# Round Time down to the next hour, since that is the hour for which we want to use flight data. Then, add the rounded Time to a new column named "Hour", and append that column to the dfWeather DataFrame.
df = dfWeather.withColumn('Hour', F.floor(dfWeather['Time']/100))

# Replace any missing HourlyPrecip and WindSpeed values with 0.0
df = df.fillna('0.0', subset=['HourlyPrecip', 'WindSpeed'])

# Replace any WindSpeed values of "M" with 0.005
df = df.replace('M', '0.005', 'WindSpeed')

# Replace any SeaLevelPressure values of "M" with 29.92 (the average pressure)
df = df.replace('M', '29.92', 'SeaLevelPressure')

# Replace any HourlyPrecip values of "T" (trace) with 0.005
df = df.replace('T', '0.005', 'HourlyPrecip')

# Be sure to convert WindSpeed, SeaLevelPressure, and HourlyPrecip columns to float
# Define a new DataFrame that includes just the columns being used by the model, including the new Hour feature
dfWeather_Clean = df.select('AirportCode', 'Month', 'Day', 'Hour', df['WindSpeed'].cast('float'), df['SeaLevelPressure'].cast('float'), df['HourlyPrecip'].cast('float'))
```

Now let's take a look at the new `dfWeather_Clean` DataFrame.

```python
display(dfWeather_Clean)
```

Your output should look as follows:

![Screenshot of the clean weather DataFrame output](media/clean-weather-output.png 'Clean weather output')

Observe that the new DataFrame only has 7 columns. Also, the WindSpeed, SeaLevelPressure, and HourlyPrecip fields are all numeric and contain no missing values.

Now let's persist the cleaned weather data to a persistent global table.

```python
dfWeather_Clean.write.mode("overwrite").saveAsTable("flight_weather_clean")
```

### Join the Flight and Weather datasets

With both datasets ready, we want to join them together so that we can associate historical flight delays with the weather data at departure time.

```python
dfFlightDelaysWithWeather = spark.sql("SELECT d.OriginAirportCode, \
                 d.Month, d.DayofMonth, d.CRSDepHour, d.DayOfWeek, \
                 d.Carrier, d.DestAirportCode, d.DepDel15, w.WindSpeed, \
                 w.SeaLevelPressure, w.HourlyPrecip \
                 FROM flight_delays_clean d \
                 INNER JOIN flight_weather_clean w ON \
                 d.OriginAirportCode = w.AirportCode AND \
                 d.Month = w.Month AND \
                 d.DayofMonth = w.Day AND \
                 d.CRSDepHour = w.Hour")
```

Persist the combined dataset to a new persistent global table.

```python
dfFlightDelaysWithWeather.write.mode("overwrite").saveAsTable("flight_delays_with_weather")
```

### Train the machine learning model

AdventureWorks Travel wants to build a model to predict if a departing flight will have a 15-minute or greater delay. In the historical data they have provided, the indicator for such a delay is found within the DepDel15 (where a value of 1 means delay, 0 means no delay). To create a model that predicts such a binary outcome, we can choose from the various Two-Class algorithms provided by Spark MLlib. For our purposes, we choose Decision Tree. This type of classification module needs to be first trained on sample data that includes the features important to making a prediction and must also include the actual historical outcome for those features.

The typical pattern is to split the historical data so a portion is shown to the model for training purposes, and another portion is reserved to test just how well the trained model performs against examples it has not seen before.

To start, let's import the Python libraries and modules we will use in this notebook.

```python
from pyspark.ml import Pipeline, PipelineModel
from pyspark.ml.feature import OneHotEncoderEstimator, StringIndexer, VectorAssembler
from pyspark.sql.functions import array, col, lit
```

Load the data from the global table and save the column list to a variable.

```python
dfDelays = spark.sql("select OriginAirportCode, cast(Month as int) Month, cast(DayofMonth as int) DayofMonth, CRSDepHour, cast(DayOfWeek as int) DayOfWeek, Carrier, DestAirportCode, DepDel15, WindSpeed, SeaLevelPressure, HourlyPrecip from flight_delays_with_weather")
cols = dfDelays.columns
```

To begin, let's evaluate the data to compare the flights that are delayed (DepDel15) to those that are not. What we're looking for is whether one group has a much higher count than the other.

```python
dfDelays.groupBy("DepDel15").count().show()
```

Your output should look similar to the following:

+--------+-------+
|DepDel15| count|
+--------+-------+
| 1| 591608|
| 0|2267686|
+--------+-------+

Judging by the delay counts, there are almost four times as many non-delayed records as there are delayed.

We want to ensure our model is sensitive to the delayed samples. To do this, we can use stratified sampling provided by the `sampleBy()` function. First we create fractions of each sample type to be returned. In our case, we want to keep all instances of delayed (value of 1) and downsample the not delayed instances to 30%.

```python
fractions = {0: .30, 1: 1.0}
trainingSample = dfDelays.sampleBy("DepDel15", fractions, 36L)
trainingSample.groupBy("DepDel15").count().show()
```

Your new output should look as follows:

+--------+------+
|DepDel15| count|
+--------+------+
| 1|591608|
| 0|680186|
+--------+------+

You can see that the number of delayed and not delayed instances are now much closer to each other. This should result in a better-trained model.

### Select an algorithm and transform features

Because we are trying to predict a binary label (flight delayed or not delayed), we need to use binary classification. For this, we will be using the [Decision Tree](https://spark.apache.org/docs/latest/ml-classification-regression.html#decision-tree-classifier) classifier algorithm provided by the Spark MLlib library. We will also be using the [Pipelines API](https://spark.apache.org/docs/latest/ml-guide.html) to put our data through all of the required feature transformations in a single call. The Pipelines API provides higher-level API built on top of DataFrames for constructing ML pipelines.

In the data cleaning phase, we identified the important features that most contribute to the classification. The `flight_delays_with_weather` is the result of the data preparation and feature identification process. The features are:

| OriginAirportCode | Month | DayofMonth | CRSDepHour | DayOfWeek | Carrier | DestAirportCode | WindSpeed | SeaLevelPressure | HourlyPrecip |
| ----------------- | ----- | ---------- | ---------- | --------- | ------- | --------------- | --------- | ---------------- | ------------ |
| LGA               | 5     | 2          | 13         | 4         | MQ      | ORD             | 6         | 29.8             | 0.05         |

We also have a label named `DepDelay15` which equals 0 if no delay, and 1 if there was a delay.

As you can see, this dataset contains nominal variables like OriginAirportCode (LGA, MCO, ORD, ATL, etc). In order for the machine learning algorithm to use these nominal variables, they need to be transformed and put into Feature Vectors, or vectors of numbers representing the value for each feature.

For simplicity's sake, we will use One-Hot Encoding to convert all categorical variables into binary vectors. We will use a combination of StringIndexer and OneHotEncoderEstimator to convert the categorical variables. The `OneHotEncoderEstimator` will return a `SparseVector`.

Since we will have more than 1 stage of feature transformations, we use a Pipeline to tie the stages together. This simplifies our code.

The ML package needs the label and feature vector to be added as columns to the input dataframe. We set up a pipeline to pass the data through transformers in order to extract the features and label. We index each categorical column using the `StringIndexer` to a column of number indices, then convert the indexed categories into one-hot encoded variables with at most a single one-value. These binary vectors are appended to the end of each row. Encoding categorical features allows decision trees to treat categorical features appropriately, improving performance. We then use the `StringIndexer` to encode our labels to label indices.

```python
categoricalColumns = ["OriginAirportCode", "Carrier", "DestAirportCode"]
stages = [] # stages in our Pipeline
for categoricalCol in categoricalColumns:
    # Category Indexing with StringIndexer
    stringIndexer = StringIndexer(inputCol=categoricalCol, outputCol=categoricalCol + "Index")
    # Use OneHotEncoderEstimator to convert categorical variables into binary SparseVectors
    encoder = OneHotEncoderEstimator(dropLast=False, inputCols=[stringIndexer.getOutputCol()], outputCols=[categoricalCol + "classVec"])
    # Add stages.  These are not run here, but will run all at once later on.
    stages += [stringIndexer, encoder]

# Convert label into label indices using the StringIndexer
label_stringIdx = StringIndexer(inputCol="DepDel15", outputCol="label")
stages += [label_stringIdx]
```

Now we need to use the `VectorAssembler` to combine all the feature columns into a single vector column. This includes our numeric columns as well as the one-hot encoded binary vector columns.

```python
# Transform all features into a vector using VectorAssembler
numericCols = ["Month", "DayofMonth", "CRSDepHour", "DayOfWeek", "WindSpeed", "SeaLevelPressure", "HourlyPrecip"]
assemblerInputs = [c + "classVec" for c in categoricalColumns] + numericCols
assembler = VectorAssembler(inputCols=assemblerInputs, outputCol="features")
stages += [assembler]
```

### Create and train the Decision Tree model

Before we can train our model, we need to randomly split our data into test and training sets. As is standard practice, we will allocate a larger portion (70%) for training. A seed is set for reproducibility, so the outcome is the same (barring any changes) each time this cell and subsequent cells are run.

Remember to use our stratified sample (`trainingSample`).

```python
### Randomly split data into training and test sets. set seed for reproducibility
(trainingData, testData) = trainingSample.randomSplit([0.7, 0.3], seed=100)
# We want to have two copies of the training and testing data, since the pipeline runs transformations and we want to run a couple different iterations
trainingData2 = trainingData
testData2 = testData
print(trainingData.count())
print(testData.count())
```

Sample output from above:

```
891096
380698
```

Our pipeline is ready to be built and run, now that we've created all the transformation stages. We just have one last stage to add, which is the Decision Tree. Let's run the pipeline to put the data through all the feature transformations within a single call.

Calling `pipeline.fit(trainingData)` will transform the test data and use it to train the Decision Tree model.

```python
from pyspark.ml.classification import DecisionTreeClassifier

# Create initial Decision Tree Model
dt = DecisionTreeClassifier(labelCol="label", featuresCol="features", maxDepth=3)
stages += [dt]

# Create a Pipeline.
pipeline = Pipeline(stages=stages)
# Run the feature transformations.
#  - fit() computes feature statistics as needed.
#  - transform() actually transforms the features.
pipelineModel = pipeline.fit(trainingData)
trainingData = pipelineModel.transform(trainingData)
# Keep relevant columns
selectedcols = ["label", "features"] + cols
trainingData = trainingData.select(selectedcols)
display(trainingData)
```

Let's make predictions on our test dataset using the `transform()`, which will only use the 'features' column. We'll display the prediction's schema afterward so you can see the three new prediction-related columns.

```python
# Make predictions on test data using the Transformer.transform() method.
predictions = pipelineModel.transform(testData)
```

To properly train the model, we need to determine which parameter values of the decision tree produce the best model. A popular way to perform model selection is k-fold cross validation, where the data is randomly split into k partitions. Each partition is used once as the testing data set, while the rest are used for training. Models are then generated using the training sets and evaluated with the testing sets, resulting in k model performance measurements. The model parameters leading to the highest performance metric produce the best model.

We can use `BinaryClassificationEvaluator` to evaluate our model. We can set the required column names in `rawPredictionCol` and `labelCol` Param and the metric in `metricName` Param.

Let's evaluate the Decision Tree model with `BinaryClassificationEvaluator`.

```python
from pyspark.ml.evaluation import BinaryClassificationEvaluator
# Evaluate model
evaluator = BinaryClassificationEvaluator()
evaluator.evaluate(predictions)
```

Sample output: `0.6207880706887883`

Now we will try tuning the model with the `ParamGridBuilder` and the `CrossValidator`.

As we indicate 3 values for maxDepth and 3 values for maxBin, this grid will have 3 x 3 = 9 parameter settings for `CrossValidator` to choose from. We will create a 3-fold CrossValidator.

```python
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator

# Create ParamGrid for Cross Validation
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
paramGrid = (ParamGridBuilder()
             .addGrid(dt.maxDepth, [1, 2, 6, 10])
             .addGrid(dt.maxBins, [20, 40, 80])
             .build())
```

```python
# Create 3-fold CrossValidator
cv = CrossValidator(estimator=pipeline, estimatorParamMaps=paramGrid, evaluator=evaluator, numFolds=3)

# Run cross validations (this can take several minutes to execute)
cvModel = cv.fit(trainingData2)
```

Now let's create new predictions with which to measure the accuracy of our model.

```python
predictions = cvModel.transform(testData2)
```

We'll use the predictions to evaluate the best model. `cvModel` uses the best model found from the Cross Validation.

```python
evaluator.evaluate(predictions)
```

Sample output: `0.6210830851404006`

We need to take the best model from `cvModel` and generate predictions for the entire dataset (dfDelays), then evaluate the best model.

```python
bestModel = cvModel.bestModel
finalPredictions = bestModel.transform(dfDelays)
evaluator.evaluate(finalPredictions)
```

Sample output: `0.6216832285847772`

### Save the model to disk

There are two reasons for saving the model in this scenario. The first is so you can access the trained model later if your cluster restarts for any reason, and also from within another notebook. Secondly, you can make the model externally by exporting it. This allows you to operationalize your model using a service such as Azure Machine Learning.

Save the model to the local DBFS file store:

```python
bestModel.write().overwrite().save("/dbfs/FileStore/models/pipelineModel")
```
