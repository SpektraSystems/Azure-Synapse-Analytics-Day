# Exercise 2 - Build a Modern Data Warehouse with Azure Synapse Pipelines

### Estimated Duration: 90 minutes

In this exercise, you examine various methods for ingesting data into Azure Synapse Analytics and Azure Data Lake Storage Gen2. You use notebooks and Data Flows to ingest, transform, and load data.

## Lab Objectives

The tasks you will perform in this exercise are:

- Task 1 - Explore and modify a notebook
- Task 2 - Explore, modify, and run a Pipeline containing a Data Flow
- Task 3 - Monitor pipelines
- Task 4 - Monitor Spark applications

---

**Important**:

In the tasks below, you will be asked to enter a unique identifier in several places. You can find your unique identifier by looking at the username you were provided for logging into the Azure portal. Your username is in the format `odl_user_UNIQUEID@msazurelabs.onmicrosoft.com`, where the _UNIQUEID_ component looks like `206184`, `206137`, or `205349`, as examples.

Please locate this value and note it for the steps below.

---

## Task 1 - Explore and modify a notebook

In this task, you see how easy it is to write into an SQL Pool table with Spark thanks to the SQL Analytics Connector. Notebooks are used to write the code required to write to SQL Pool tables using Spark.

1. **Note:** If you still have your notebook open from the end of Exercise 1, **skip ahead** to step 3 below. Otherwise, in Synapse Studio, select **Develop** from the left-hand menu.

   ![Develop is selected and highlighted in the Synapse Analytics menu.](media/develop-hub.png "Develop hub")

2. Select **+ (1)**, then **Notebook (2)** to add a new notebook.

   ![The new notebook menu item is highlighted.](media/newimg1.png "New notebook")

3. If not already attached, attach your Spark Compute **SparkPool02** by selecting it from the **Attach to** drop-down list and attach **PySpark(Python)** by selecting it from **Language** drop-down list.

   ![The Spark pool is selected in the Attach to drop-down.](media/ex2tsk2stp3.png "Add code")

4. Paste the following into the new cell and **replace** `YOUR_DATALAKE_NAME` with your storage account name **<inject key="Storage Account Name"></inject>**.

    ```scala
    %%spark

    // Set the path to read the WWI Sales files
    import org.apache.spark.sql.SparkSession

    // Set the path to the ADLS Gen2 account
    val adlsPath = "abfss://wwi@YOUR_DATALAKE_NAME.dfs.core.windows.net"
    ```

    Select the **Run cell** button to execute the new cell:

    ![The new cell is displayed.](media/ex02-notebook-cell1.png "Run cell")

    > This cell imports required libraries and sets the `adlsPath` variable, which defines the path used to connect to an Azure Data Lake Storage (ADLS) Gen2 account. Connecting to ADLS Gen2 from a notebook in Azure Synapse Analytics uses the power of Azure Active Directory (AAD) pass-through between compute and storage. The `%%spark` "magic" sets the cell language to Scala, which is required to use the `SparkSession` library.

5. Hover over the area just below the cell in the notebook, then select **+ Code** to add a new cell.

    ![The add code button is highlighted.](media/synapse-workspace-add-code.png "Add code")

6. Paste the following and run the new cell:

    ```scala
    %%spark

    // Read the sales into a dataframe
    val sales = spark.read.format("csv").option("header", "true").option("inferSchema", "true").option("sep", "|").load(s"$adlsPath/factsale-csv/2012/Q4")
    sales.show(5)
    sales.printSchema()
    ```

    This code loads data from CSV files in the data lake into a DataSet. Note the `option` parameters in the `read` command. These options specify the settings to use when reading the CSV files. The options tell Spark that the first row of each file containers the column headers, the separator in the files in the `|` character, and that we want Spark to infer the schema of the files based on an analysis of the contents of each column. Finally, we display the first five records of the data retrieved and print the inferred schema to the screen.

7. When the cell finishes running, take a moment to review the associated output.

    > The output of this cell provides some insight into the structure of the data and the data types that have been inferred. The `show(5)` command results in the first five rows of the data read being output, allowing you to see the columns and a sample of data contained within each. The `printSchema()` command outputs a list of columns and their inferred types.

    ![The output from the execution the cell is displayed, with the result of the show(5) command shown first, followed by the output from the printSchema() command.](media/ex02-notebook-ingest-cell-2-output.png "Cell output")

8. Hover over the area just below the cell in the notebook, then select **+ Code** to add a new cell.

    ![The add code button is highlighted.](media/synapse-workspace-add-code.png "Add code")

9. Paste the following and run the new cell:

    ```scala
    %%spark

    // Import libraries for the SQL Analytics connector
    import com.microsoft.spark.sqlanalytics.utils.Constants
    import org.apache.spark.sql.SqlAnalyticsConnector._
    import org.apache.spark.sql.SaveMode

    // Set target table name
    var tableName = s"SQLPool01.wwi_staging.Sale"

    // Write the retrieved sales data into a staging table in Azure Synapse Analytics.
    sales.limit(10000).write.mode(SaveMode.Append).sqlanalytics(tableName, Constants.INTERNAL)
    ```
    
    This code writes the data retrieved from Blob Storage into a staging table in Azure Synapse Analytics using the SQL Analytics connector. Using the connector simplifies connecting to Azure Synapse Analytics because it uses AAD pass-through. There is no need to create a password, identity, external table, or format sources, as it is all managed by the connector.

10. As the cell runs, select the arrow icon below the cell to expand the details for the Spark job. Once it completes move on to the next step.

    > This pane allows you to monitor the underlying Spark jobs, and observe the status of each. As you can see, the cell is split into two Spark jobs, and the progress of each can be observed. We will take a more in-depth look at monitoring Spark applications in Task 4 below.

    ![The Spark job status pane is displayed below the cell, with the progress of each Spark job visible.](media/ex02-notebook-ingest-cell-3-spark-job.png "Spark Job status")

11. Close the notebook by selecting the **X** in the top right of the tab, select **Stop session**. Then select **Close + discard changes** Closing the notebook will ensure you free up the allocated resources on the Spark Pool.
     
    ![The Close + discard changes button is highlighted.](media/closenotebook.png "closenotebook")

    ![The Close + discard changes button is highlighted.](media/notebook-close-discard-changes.png "Discard changes?")
    
    > **Note**: If you get an error while performing this step, you can change the sparkpool to sparkpool02 and try re-running the steps.

12. Now, select **Data** from the left-hand menu.

    ![Data is selected and highlighted in the Synapse Analytics menu.](media/data-hub.png "Data hub")

13. Under **Workspace (1)** tab expand **SQL databases (2)** and then expand the **SQLPool01 (3)** database.

    ![The Databases folder is expanded, showing a list of databases within the Azure Synapse Analytics workspace. SQLPool01 is expanded and highlighted.](media/ex2tsk2stp13.png "Synapse Analytics Databases")

14. Expand **Tables (1)** and locate the table named `wwi_staging.Sale` **(2)**.

    > **Note**: If you do not see the table, select the Actions ellipsis next to Tables and then select **Refresh** from the fly-out menu.

    ![The new wwi_staging.Sale table is displayed.](media/data-staging-sales.png "New Sale table")

15. To the right of the `wwi_staging.Sale` table, select the Actions ellipsis.

    ![The Actions ellipsis button is highlighted next to the wwi_staging.Sale_UNIQUEID table.](media/ex02-data-sqlpool01-tables-staging-wwi-sales-data-actions.png "Synapse Analytics Databases")

16. In the Actions menu, select **New SQL script (1) > Select TOP 100 rows (2)**.

    ![In the Actions menu for the wwi_staging.Sale table, New SQL script > Select TOP 100 rows is highlighted.](media/ex02-data-sqlpool01-tables-staging-wwi-sales-data-actions-select.png "Synapse Analytics Databases")

17. Select **Run (1)** to execute the query. Observe the **Results (2)** in the output pane, and notice how easy it was to use Spark notebooks to write data from Blob Storage into Azure Synapse Analytics in Steps 9 and 10.

    ![The output of the SQL statement is displayed.](media/staging-sale-output.1.png "Sale script output")

18. Close the SQL script generated by `wwi_staging.Sale`.

  > **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
	
  - Hit the Validate button for the corresponding task. If you receive a success message, you can proceed to the next task. 
  - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
  - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.

  <validation step="13e4cc15-01f7-4923-9295-d9b559271984" />

## Task 2 - Explore, modify, and run a Pipeline containing a Data Flow

In this task, you use a Pipeline that implements Code-free AI to do sentiment analysis on customer feedback and contains a Data Flow to explore, transform, and load data into an Azure Synapse Analytics table. Using Cognitive Services and data flows in Pipelines allows you to handle code-free AI workloads, and perform data ingestion and transformations, similar to what you did in Task 1, but without writing any code

1. In Synapse Studio select **Integrate** from the left-hand menu.

   ![Integrate hub.](media/integrate-hub.png "Integrate hub")

2. In the Integrate menu, expand **Pipelines**, then select **Exercise 2 - Enrich Data**.

   ![The Enrich Data pipeline is selected.](media/ex2tsk2stp2.png "Pipelines")

   > Selecting a pipeline opens the pipeline canvas, where you can review and edit the pipeline using a code-free, graphical interface. This view shows the various activities within the pipeline and the links and relationships between those activities. The `Exercise 2 - Enrich Data` pipeline contains four activities;

   > - a lookup activity named `ReadComments` reading customer comments
   > - a ForEach loop named `ForEachComment` iterating through comments and running Sentiment Analysis with Azure Cognitive Services
   > - a copy data activity named `Import Customer dimension`
   > - a mapping data flow activity named `Enrich Customer Data`.

3. Now, take a closer look at each of the activities within the pipeline. On the canvas graph, select the **Lookup (1)** activity named `ReadComments` and switch to the **Settings (2)** tab. The source dataset is set to a CSV file stored in the data lake.

   > A lookup activity reads and returns the content of a file which can be consumed in a subsequent copy, transformation, or control flow activities like ForEach activity. The Lookup activity output supports up to 4 MB in size. In this case, the source is set to a single CSV file that is much smaller. For larger data sets multiple source files can be used.

   ![The Settings tab of the ReadCOmments Lookup Activity is selected. Source dataset is highlighted.](media/enrich-data-pipeline-readcomments.1.png "Read Comments")
   
4. From the canvas graph, select the **ForEach (1)** activity named `ForEachComment` and switch to the **Settings (2)** tab. The **Items** property is set to receive the output of the previous `ReadComments` **(3)** activity. You might have noticed the green connection line between the two activities that defines a dependency between the two activities. The line makes sure comments are read before the ForEach loop can iterate it. Select the edit button **(4)** in the ForEach activity's Activities box to navigate into the loop.

   ![ForEachComments ForEach activity is selected. Settings tab is shown. Items property is highlighted.](media/y8.png "ForEach Loop")

5. Select **Copy data** activity named `Sentiment Analysis` **(1)** and switch to the **Source (2)** tab. The Copy Data activity's Source dataset is set to a REST resource **(3)** backed by Azure Cognitive Services. A POST **(4)** HTTP request will be made to the Azure Cognitive Services endpoint carrying a request body **(5)** that includes the text from the current iteration that will be analyzed for sentiments.

   > **NOTE!** Add the below code in the **Request Body** if not already added. 

   ``` 
   { "documents": [{ "id": "@{item().id}", "language": "en", "text": "@{item().comment}" }] }
   ```

   ![Copy data activity named Sentiment Analysis is selected. Source tab is open. Source dataset is set to a REST Data Source. Request body and method are highlighted.](media/ex2tsk2stp5.png "Copy Data REST Source")

6. Switch to the **Sink (1)** tab. Here, the sink dataset is set to a JSON file location in the data lake. Once the Copy data activity gets the result of the sentiment analysis from the remote REST resource endpoint, the result will be saved as separate JSON files into the Sink dataset **(2)**. The files will include a complete sentiment analysis for the customer comment that can be queried and analyzed further. Select **Exercise 2 - Enrich Data (3)** link to go back to the main pipeline canvas.

   ![Copy data activity named Sentiment Analysis is selected. Sink tab is open. Sink dataset is set to a JSON Data Source. ](media/data-enrichment-pipeline-sentiment-analysis-activity-sink.1.png "Sentiment Sink")

7. On the canvas graph, select the **Copy data** activity named `Import Customer dimension`.

   > Below the graph is a series of tabs, each providing additional details about the selected activity. The **General** tab displays the name and description assigned to the activity and a few other properties.

   ![A screenshot of Exercise 2 - Enrich Data pipeline canvas and properties pane is displayed.](https://github.com/solliancenet/azure-synapse-analytics-day/raw/master/media/ex02-orchestrate-copy-data-general.png "Pipeline canvas")

8. Select the **Source** tab. The source defines the location from which data will be copied by the activity. The **Source dataset** field is a pointer to the location of the source data.

   > Take a moment to review the various properties available on the Source tab. Data is being retrieved from files stored in a data lake.

   ![The Source tab for the Copy data activity is selected and highlighted.](media/ex02-orchestrate-copy-data-source.png "Pipeline canvas property tabs")

9. Next, select the **Sink** tab. The sink specifies where the copied data will be written. Like the Source, the sink uses a dataset to define a pointer to the target data store. Select **PolyBase** for the `Copy method`. This improves the data loading speed as compared to the default setting of the bulk insert.

   ![The Sink tab for the Copy data activity is selected and highlighted.](media/sw-sink-polybase.png "Pipeline canvas property tabs")

   > Reviewing the fields on this tab, you will notice that it is possible to define the copy method, and table options, and to provide pre-copy scripts to execute. Also, take special note of the sink dataset, `wwi_staging_dimcustomer_asa`. The dataset requires a parameter named `UniqueId`, which is populated using a substring of the Pipeline Run Id. This dataset points to the `wwi_staging.DimCustomer_UniqueId` table in Synapse Analytics, which is one of the data sources for the Data Flow. We will need to ensure that the copy activity successfully populates this table before running the data flow.

10. Select the **Mapping** tab. On this tab, you can review and set the column mappings. As you can see on this tab, the spaces are being removed from the column names in the sink.

    ![The Mappings tab for the Copy data activity is highlighted and displayed.](media/ex02-orchestrate-copy-data-mapping.png "Pipeline canvas property tabs")

11. Finally, select the **Settings** tab. Check **Enable staging** and expand `Staging settings`. Search and select **asadatalake01** under `Staging account linked service`, then type **staging** into `Storage Path`. Finally, check **Enable Compression**.

    ![The staging settings are configured as described.](media/pic2.png "Settings")

    > Since we are using PolyBase with dynamic file properties, owing to the UniqueId values, we need to [enable staging](https://docs.microsoft.com/azure/data-factory/connector-azure-sql-data-warehouse#staged-copy-by-using-polybase). In cases of large file movement activities, configuring a staging path for the copy activity can improve performance.

12. Switch to the **Data Flow** activity by selecting the `Enrich Customer Data` Data Flow activity on the pipeline design canvas, then select the **Settings** tab.

    ![The data flow activity settings are displayed.](media/pipeline-data-flow-settings.png "Settings")

    > Observe the settings configurable on this tab. They include parameters to pass into the data flow, the Integration Runtime, and compute resource type and size to use. If you wish to use staging, you can also specify that here.

13. Next, select the **Parameters** tab in the configuration panel of the Data Flow activity.

    ![The Parameters table on the configuration panel of the Mapping Data Flow activity is selected and highlighted.](media/ex02-orchestrate-data-flow-parameters.png "Mapping Data Flow activity")

    Notice that the value contains:

    ```sql
    @substring(pipeline().RunId,0,8)
    ```

    > This sets the UniqueId parameter required by the `EnrichCustomerData` data flow to a unique substring extracted from the pipeline run ID.

    > **NOTE!** If the **Parameters** value is absent, please update the value with **```@substring(pipeline().RunId,0,8)```** by selecting **Pipeline expression** and paste the value under **Add dynamic content** then **finish**.
    
    ![Two images showing how to add the parameter value](media/1.png "Adding parameter value")

14. Take a minute to look at the options available on the various tabs in the configuration panel. You will notice the properties here define how the data flow operates within the pipeline.

15. Now, let us take a look at the definition of the data flow the Data Flow activity references. Double-click the `Enrich Customer Data` Data Flow activity on the pipeline canvas to open the underlying Data Flow in a new tab.

    ![The EnrichCustomerData Data Flow canvas is displayed.](media/sw-double-click-enrich.png "Enrich Customer Data")

    > **Important**: Typically, when working with Data Flows, you would want to enable **Data flow debug**. [Debug mode](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-debug-mode) creates a Spark cluster to use for interactively testing each step of the data flow and allows you to validate the output prior to saving and running the data flow. Enabling a debugging session can take up to 10 minutes, so you will not enable this for the purposes of this workshop. Screenshots will be used to provide details that would otherwise require a debug session to view.

    ![The EnrichCustomerData Data Flow canvas is displayed.](media/ex02-orchestrate-data-flow.1.png "Data Flow canvas")

16. The [Data Flow canvas](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-overview#data-flow-canvas) allows you to see the construction of the data flow, and each component contained within it in greater detail.

    > From a high level, the `EnrichCustomerData` data flow is composed of two data sources, multiple transformations, and two sinks. The data source components, `PostalCodes` and `DimCustomer`, ingest data into the data flow. The `EnrichedCustomerData` and `EnrichedCustomerDataAdls` components on the right are sinks, used to write data to data stores. The remaining components between the sources and sinks are transformation steps, which can perform filtering, joins, selection, and other transformational actions on the ingested data.

    ![On the data flow canvas, the components are broken down into three sections. Section number 1 is labeled data sources and contains the PostalCodes and DimCustomer components. Section number 2 is labeled Transformations, and contains the PostCodeFilter, JoinOnPostalCode, and SelectDesiredColumns components. Section number 3 is labeled sinks, and contains the EnrichedCustomerData and EnrichedCustomerDataAdls components.](media/ex02-orchestrate-data-flow-components.1.png "Data flow canvas")

17. To better understand how a data flow functions, let us inspect the various components. Select the `PostalCodes` data source on the data flow canvas.

    > On the **Source settings** tab, we see properties similar to what we saw on the pipeline activities property tabs. The name of the component can be defined, along with the source dataset and a few other properties. The `PostalCodes` dataset points to a CSV file stored in an Azure Data Lake Storage Gen2 account.

    ![The PostalCodes data source component is highlighted on the data flow canvas surface.](media/ex2-t2-step17.png "Data flow canvas")

18. Select the **Projection** tab.

    > The **Projections** tab allows you to define the schema of the data being ingested from a data source. A schema is required for each data source in a data flow to allow downstream transformations to perform actions against the fields in the data source. Note that selecting **Import schema** requires an active debug session to retrieve the schema data from the underlying data source, as it uses the Spark cluster to read the schema. In the screenshot above, notice the `Zip` column is highlighted. The schema inferred by the import process set the column type to `integer`. For US zip code data, the data type was changed to `string` so leading zeros are not discarded from the five-digit zip codes. It is important to review the schema to ensure the correct types are set, both for working with the data and to ensure it is displayed and stored correctly in the data sink.

    ![The Projections tab for the PostalCodes data source is selected, and the Zip column of the imported schema is highlighted.](media/ex02-orchestrate-data-flow-sources-postal-codes-projection.1.png "Data flow canvas")

19. The **Data preview** tab allows you to ingest a small subset of data and view it on the canvas. This functionality requires an active debug session, so for this workshop, a screenshot that displays the execution results for that tab is provided below.

    > **NOTE! This step cannot be performed in the lab environment.**

    > The `Zip` column is highlighted on the Data Preview tab to show a sample of the values contained within that field. Below, you will filter the list of zip codes down to those that appear in the customer dataset.

    ![The Data preview tab is highlighted and selected. The Zip column is highlighted on the Data preview tab.](media/ex02-orchestrate-data-flow-sources-postal-codes-data-preview.1.png "Data flow canvas")

20. Before looking at the `PostalCodeFilter`, quickly select the `+` button to the right of the `PostalCodes` data source to display a list of available transformations.

    > Take a moment to browse the list of transformations available in Data Flows. From this list, you get an idea of the types of transformations that are possible using data flows. Transformations are broken down into three categories, **multiple inputs/outputs**, **schema modifiers**, and **row modifiers**. You can learn about each transformation in the docs by reading the [Data flow transformation overview](https://docs.microsoft.com/azure/data-factory/data-flow-transformation-overview) article.

    ![The + button next to PostalCodes is highlighted, and the menu of available transformations is displayed.](media/ex02-orchestrate-data-flow-available-transformations.1.png "Data flow canvas")

21. Next, select the `PostalCodeFilter` transformation in the graph on the data flow canvas.

    ![The PostalCodeFilter transformation is highlighted on the data flow canvas graph.](media/ex02-orchestrate-data-flow-transformations-filter.png "Data flow canvas")

22. In the **Filter settings** tab of the configuration panel, click anywhere inside the **Filter on (1)** box then click on **Open expression builder (2)**.

    ![The Filter on box is highlighted in the configuration panel for the PostalCodeFilter transformation.](media/pic1.png "Data flow canvas")

23. This will open the Dataflow expression builder.

    > In data flows, many transformation properties are entered as expressions. These expressions are composed of column values, parameters, functions, operators, and literals that evaluate a Spark data type at run time. To learn more, visit the [Build expressions in data flow](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-expression-builder) page in the documentation.

    ![The Visual expression builder is displayed.](media/E2-T2-S23.png "Dataflow expression builder")

24. The filter currently applied ensures all zip codes are between 90000 and 98000. Observe the different expression elements and values in the area below the expression box that help you create and modify filters and other expressions.

25. Select **Cancel** to close the visual expression builder.

26. Select the `DimCustomer` data source on the data flow canvas graph.

    > Take a few minutes to review the various tabs in the configuration panel for this data source to get a better understanding of how it is configured, as you did above. Note that this data source relies on the `wwi_staging.DimCustomer_UniqueId` table from Azure Synapse Analytics for its data. `UniqueId` is supplied by a parameter to the data flow, which contains a substring of the Pipeline Run Id. Before running the pipeline, you will add a dependency to the Data Flow activity to ensure the Copy activity has populated the `wwi_staging.DimCustomer_UniqueId` in Azure Synapse Analytics before allowing the data flow to execute.

    ![The DimCustomer data source is highlighted on the data flow canvas graph.](media/y5.png "Data flow canvas")

27. Next, select the `JoinOnPostalCode` **(1)** transformation and ensure the **Join settings (2)** tab is selected to see how you can join datasets using a simple and intuitive graphical interface.

    > The **Join settings** tab allows you to specify the data sources being joined and the join types and conditions. Notice the **Right stream** points to the `PostalCodeFilter` and not the `PostalCodes` data source directly. By referencing the filtered dataset, the join works with a smaller set of postal codes. For extensive datasets, this can provide performance benefits.

    ![The JoinOnPostalCode transformation is highlighted in the graph, and the Join settings tab is highlighted in the configuration panel.](media/ex02-orchestrate-data-flow-transformations-join.1.png "Data flow canvas")

28. Moving on to the next transformation, the `SelectDesiredColumns` transformation uses a **Select** schema modifier to allow choosing what columns to include.

    > You have probably noticed that the `SelectDesiredColumns` transformation appears twice in the graph. To enable writing the resulting dataset to two different sinks, Azure Synapse Analytics and Azure Data Lake Storage Gen2, a **Conditional split** multiple outputs transformation is required. This split is displayed in the graph as a repeat of the split item.

    ![The SelectDesiredColumns transformation is highlighted in the data flow graph.](media/ex02-orchestrate-data-flow-transformations-select.1.png "Data flow canvas")

29. The last two items in the data flow are the defined sinks. These provide the connection settings necessary to write the transformed data into the desired data sink. Select the `EnrichCustomerData` **(1)** sink and inspect the settings on the **Sink (1)** tab.

    ![The Sink tab is displayed for the EnrichCustomerData sink, which is highlighted in the graph.](media/ex2-t2-step29.png "Data flow canvas")

30. Next, select the **Settings** tab and observe the properties set there.

    > The **Settings** tab defines how data is written into the target table in Azure Synapse Analytics. The Update method has been set only to allow inserts, and the table action is set to recreate the table whenever the data flow runs.

    ![The Settings tab is selected and highlighted in the configuration panel. On the tab, the Allow insert and Recreate table options are highlighted.](media/ex02-orchestrate-data-flow-sink-settings.1.png "Data flow canvas")

31. Now that you have taken the time to review the data flow, let us return to the pipeline. On the canvas, select the **Exercise 2 - Enrich Data** tab.

    ![The Exercise 2 - Enrich Data tab is highlighted on the canvas.](media/ex02-orchestrate-canvas-tabs-pipeline.png "Data pipeline canvas")

32. Before running the pipeline there is one more change we need to make. As mentioned above, the data flow depends on the data written by the copy activity, so you will add a dependency between the two activities.

33. In the data flow canvas graph, select the green box on the right-hand side of the `Import Customer dimension` Copy data activity and drag the resulting arrow up onto the `Enrich Customer Data` Data Flow activity.

    ![The green box on the right-hand side of the Copy data activity is highlighted, and the arrow has been dragged onto the Mapping Data Flow.](media/ex02-orchestrate-pipelines-create-dependency.png "Data pipeline canvas")

34. This creates a requirement that the **Copy data** activity completes successfully before the **Data Flow** can execute, and enforces our requirement of the Synapse Analytics table being populated before running the data flow.

    ![The dependency arrow going from the Copy data activity to the Mapping Data Flow is displayed.](media/y6.png "Data pipeline canvas")

35. The last step before running the pipeline is to publish the changes you have made. Select **Publish all** on the toolbar.

    ![The Publish all button is highlighted on the Synapse Analytics Studio toolbar.](media/ex02-orchestrate-pipelines-publish-all.png "Publish")

36. On the **Publish all** dialog, select **Publish**.

    > This Publish all dialog allows you to review the changes that will be saved.

37. Within a few seconds, you may receive a notification that the publication has been completed. If so, select **Dismiss** in the notification.

    ![The publishing completed notification is displayed.](media/ex02-publishing-completed.png "Publishing completed")

38. Your pipeline is now ready to run. Select **Add trigger** then **Trigger now** on the toolbar for the pipeline.

    ![Add trigger is highlighted on the pipeline toolbar, and Trigger now is highlighted in the fly-out menu.](media/ex02-orchestrate-pipelines-trigger-now.1.png "Trigger pipeline")

39. Select **OK** on the Pipeline run dialog to start the pipeline run.

    ![The OK button is highlighted in the Pipeline run dialog.](media/ex02-orchestrate-pipelines-trigger-run.png "Pipeline run trigger")

40. To monitor the pipeline run, move on to the next task.

## Task 3 - Monitor pipelines

After you finish building and debugging your data flow and its associated pipeline, you will want to be able to monitor the execution of the pipeline and all of the activities contained within it, including the Data Flow activity. In this task, you review the [pipeline monitoring functionality in Azure Synapse Analytics](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-monitoring) using the pipeline run you initiated at the end of the previous task.

1. In Synapse Analytics Studio, select **Monitor** from the left-hand menu.

   ![Monitor is selected and highlighted in the Synapse Analytics menu.](media/monitor-hub.png "Synapse Analytics menu")

2. Under Integration, select **Pipeline runs**.

   ![Pipeline runs is selected and highlighted under the Orchestration section of the monitor resource list.](media/ex02-monitor-pipeline-runs1.png "Synapse Analytics Monitor")

3. Select the `Exercise 2 - Enrich Data_A03` pipeline in the list. This will have a status of `In progress`.

   ![The first "Exercise 2 - Enrich Data" item in the list of pipeline runs is highlighted.](media/ex02-monitoring-pipeline-runs.1.png "Pipeline run list")

4. On the pipeline run details screen, you will see a graphical representation of the activities within the pipeline, as well as a list of the individual activity runs. Both provide status indicators for each activity.

   > This view allows you to monitor the overall status of the pipeline run, and observe the status of each activity contained within the pipeline. The screen will auto-refresh for five minutes. If auto-refresh does not occur or your pipeline run takes longer than five minutes, you can get updates by selecting the Refresh button on the canvas toolbar.

   ![The pipeline run canvas is displayed, with activities list in the graph and in a list for in the Activity runs panel.](media/ex02-monitoring-pipeline-runs-details.1.png "Pipeline run details")

5. To get a better understanding of the types of information you can get from the monitoring capabilities, let us explore what information is available for each of the activities in the Activity runs list. Start by hovering your mouse cursor over the **Import Customer dimension** activity and select the **Output** icon that appears.

   ![The output icon is highlighted on the Import Customer dimension activity row.](media/ex02-monitoring-copy-activity-output.png "Copy activity output")

6. In the **Output** dialog, you will see details about the size of data read and written, the number of rows reads and copied, the duration of the copy activity, and other information relating to the copy activity run. This information can be used for things like troubleshooting. For example, you could compare the copy run to data, such as the number of rows read and written, to expected numbers from the source and sink.

   ![The Output dialog for the Import Customer dimension activity is displayed.](media/ex02-monitoring-copy-activity-output-details.1.png "Copy activity output")

7. Close the Output dialog.

8. Next, hover your mouse cursor over the **Import Customer dimension** activity again, this time selecting the **Details** icon that appears.

   ![The Details icon is highlighted on the Import Customer dimension activity row.](media/ex02-monitoring-copy-activity.png "Copy activity run")

9. The **Details** dialog provides the data found on the Output dialog examined above but expands on that to include graphics for the source, staging storage, and sink, and a more detailed look at the activity run.

   ![The Details dialog for the copy activity is displayed.](media/ex02-monitoring-copy-activity-details.1.png "Copy activity details")

10. Close the Details dialog.

11. When the pipeline execution completes, all activity runs will reflect a status of Succeeded.

    ![A screenshot of the activity runs for the Exercise 2 - Enrich Data pipeline is displayed with all activities showing a status of Succeeded.](media/ex02-monitor-ex2-enrich-data-activity-runs-succeeded.1.png "Pipeline run monitoring")
    
    >It takes around 5 minutes for the pipeline to have a status of Succeeded.

12. When the **Enrich Customer Data** activity has a status of **Complete**, hover your mouse cursor over the **Enrich Customer Data** activity and select the **Details** icon that appears.

    > **Note**: It can take 5-7 minutes for the **Enrich Customer Data** activity to complete. You may need to select **Refresh** on the Monitoring toolbar to see the status update if your pipeline run takes longer than five minutes.

    ![The Details icon is highlighted on the Enrich Customer Data Mapping Data Flow activity row.](media/ex02-monitoring-data-flow.png "Data flow activity")

13. The Details dialog for data flow details takes you to a full-screen view of your data flow.

    > The initial view provides a details panel containing statistics for the sinks defined within the data flow. The information for these includes the number of rows written and the processing times for writing to each sink.

    ![The data flow Details dialog is displayed.](media/ex2-t3-step-13.png "Data Flow activity details")

14. Select the **SelectDesiredColumns** transformation step of the data flow.

    > Selecting any component of the data flow opens a new panel with details related to the processing that occurred for that component.

    ![The SelectDesiredColumns transformation step is highlighted in the graph on the details dialog.](media/ex2-t3-step14.png "Data Flow activity details")

15. Try selecting another component, such as the **EnrichCustomerData** sink, and view the information available.

    ![The EnrichCustomerData sink component is highlighted in the graph, and the associated details panel is displayed on the right-hand side of the screen.](media/ex2-t3-step15.png "Data Flow activity details")

16. Close the data flow activity Details dialog by selecting the **X** on the right-hand side of the toolbar.

    ![The X (close) button is highlighted on the data flow Details dialog toolbar.](media/ex02-monitor-data-flow-close.png "Data flow details")

17. Back on Exercise 2 - Enrich the Data pipeline run screen, and switch to the **Gantt** view. This view provides a graphical representation of the run times of the various activities within the pipeline.

    ![The Gantt view option is selected and highlighted on the pipeline run dialog.](media/y7.png "Pipeline run Gantt view")

  > **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
	
  - Hit the Validate button for the corresponding task. If you receive a success message, you can proceed to the next task. 
  - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
  - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.

   <validation step="b49fd917-9676-43fa-a48e-37d7384b71db" />
  
  
## Task 4 - Monitor Spark applications

In this task, you examine the Apache Spark application monitoring capabilities built into Azure Synapse Analytics. The Spark application monitoring screens provide a view of the logs for the Spark application, including a graphical view of those logs.

1. As you did in the previous task, select **Monitor** from the left-hand menu.

      ![Monitor is selected and highlighted in the Synapse Analytics menu.](media/monitor-hub.png "Synapse Analytics menu")

2. Next, select **Apache Spark applications** under Activities.

     ![Apache Spark applications is selected and highlighted under the Activities section of the monitor resource list.](media/ex2tsk4stp2.png "Synapse Analytics Monitor")

3. On the Apache Spark applications page, select the **Local time** value and observe the available options for limiting the time range for Spark applications that are displayed in the list. In this case, you are looking at the current run, so ensure **Last 24 hours** is selected and then select **OK**.

     ![Last 24 hours is selected and highlighted in the Time range list.](media/ex02-monitor-activities-spark-time-range1.png "Synapse Analytics Monitor")

4. From the list of Spark applications, select the first job, which should have a status of `In progress` or `Succeeded`.

      > **Note**: You may see a status of `Cancelled` or `Stopped`, and this does not prevent you from completing the following steps. Azure Synapse Analytics is still in preview, and the status gets set to `Cancelled` or `Stopped` when the Spark pool used to run the Spark application times out.

      ![The current Spark application is highlighted in the applications list.](media/ex02-monitor-activities-spark-application-list.1.png "Synapse Analytics Monitor")

5. On the **Dataflow** screen, you will see a detailed view of the job, broken into three different sections.

   - The first section is a graphical representation of the stages that make up the Spark application.
   - The second section is a summary of the Spark application.
   - The third section displays the diagnostics and logs associated with the Spark application.

      ![A screenshot of the Log query screen is displayed.](media/ex02-monitor-activities-spark-application-dataflow.1.png "Synapse Analytics Monitor")

6. Select the **Logs** tab to view the log output. You may switch between log sources and types, using the dropdown lists below.

      ![The Spark application logs are displayed.](media/a1.1.png "Logs")
   
7. To look closer at any individual job, you can use the **Job IDs** drop-down to select the job number.

     ![Job 2 is highlighted in the Job IDs drop-down list.](media/ex02-monitor-activities-spark-applications-all-job-ids-2.1.png "Synapse Analytics Monitor")

8. This view isolates the specific job within the graphical view.

     ![Job 2 is displayed.](media/ex02-monitor-activities-spark-applications-job-2.1.png "Synapse Analytics Monitor")

9. Return the view to all jobs by selecting **All job IDs** in the job ID drop-down list.

     ![All job IDs is highlighted in the Job IDs drop-down list.](media/ex02-monitor-activities-spark-applications-all-job-ids.1.png "Synapse Analytics Monitor")

10. Within the graph section, you also have the ability to **Playback** the Spark application.

       ![The Playback button is highlighted.](media/ex02-monitor-activities-spark-applications-playback.1.png "Synapse Analytics Monitor")

       >**Note**: Playback functionality is not available until the job status changes out of the `In progress` status. The job's status will remain listed as `In progress` until the underlying Spark resources are cleaned up by Azure Synapse Analytics, which can take some time.

12. Running a Playback allows you to observe the time required to complete each job, as well as review the rows read or written as the job progresses.

      ![A screenshot of an in-progress playback is displayed. The playback is at 8s into the Spark application run, and Job 2 is showing progress.](media/ex02-monitor-activities-spark-applications-playback-progress.1.png "Synapse Analytics Monitor")

13. You can also perform playback on an individual job. Returning to a view of only Job 2, the **Playback** button shows the rows written at this job, and the progress of reads and writes.

       ![A screenshot of an in-progress playback for Job 2 is displayed.](media/ex02-monitor-activities-spark-applications-playback-job-2.1.png "Synapse Analytics Monitor")

14. You can also change the view to see which jobs involved read and write activities. Select **All job IDs** in the job dropdown, and in the **View** drop-down, select **Read**. You can see which jobs performed reads, with each color-coded by how much data was read.

       ![Read is selected and highlighted in the Display drop-down list.](media/ex02-monitor-activities-spark-applications-display-drop-down-read-graph.1.png "Synapse Analytics Monitor")

## Summary

In this exercise, you explore data ingestion methods into Azure Synapse Analytics and Azure Data Lake Storage Gen2. First, you'll use a notebook to load CSV data from ADLS Gen2 into a Synapse SQL Pool table via Spark and the SQL Analytics Connector. Then, you will modify a pipeline that uses Data Flows and Code-free AI to perform sentiment analysis on customer feedback, loading the data into a Synapse table. This involves configuring data sources, transformations, and sinks in Data Flows to process and load the enriched data into Synapse Analytics.

### You have successfully completed the lab.
