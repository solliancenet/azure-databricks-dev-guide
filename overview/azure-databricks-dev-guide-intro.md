# Azure Databricks Developer Guide

This guide presents a structured approach for building solutions with [Azure Databricks](https://azure.microsoft.com/services/databricks/). It is a developer-focused supplement to the [Azure Databricks documentation](https://docs.azuredatabricks.net/), covering the big picture concepts common to Azure Databricks development scenarios. Throughout this guide, we will focus on several core scenarios to demonstrate the capabilities and power of the Azure Databricks service, including Machine Learning & Advanced Analytics, Modern Data Warehouse, and Real-time Streaming. Where appropriate, we will provide readers with example Notebooks, step-by-step code examples, and the details necessary to start a proof-of-concept (POC) for these scenarios.

## Purpose of this guide

AI and Big Data are driving innovation across all industries, drastically changing the way businesses operate. Many development teams, however, find it challenging to effectively translate their data engineering efforts into business outcomes. The purpose of this guide is to empower developers to unleash to power of data by providing guidance on how to build applications using Azure Databricks that enable the extraction of valuable analytics from their Big Data and achieve the full potential of AI.

The Azure Databricks product documentation provides an excellent, high-level introduction to the topics relating to getting started. This guide goes a step further, aiming to go beyond the scope of the product documentation, and delving deeper into the details of how software and data engineers can actually create and implement solutions using Azure Databricks. Throughout, we will provide guidance around things to consider while developing applications, and the thought processes involved in common Databricks activities, such as ingesting data, performance tuning, integration, and automation and orchestration, among others.

## Introduction to Azure Databricks

Azure Databricks is a fully-managed, cloud-based Big Data and Machine Learning platform, which empowers developers to accelerate AI and innovation by simplifying the process of building enterprise-grade production data applications. Built as a joint effort by the team that started Apache Spark and Microsoft, Azure Databricks provides data science and engineering teams with a single platform for Big Data processing and Machine Learning.

By combining the power of Databricks, an end-to-end, managed Apache Spark platform optimized for the cloud, with the enterprise scale and security of Microsoft's Azure platform, Azure Databricks makes it simple to run large-scale Spark workloads.

### Optimized environment

To address the problems seen on other Big Data platforms, Azure Databricks was optimized from the ground up, with a focus on performance and cost-efficiency in the cloud. The Databricks Runtime adds several key capabilities to Apache Spark workloads that can increase performance and reduce costs by as much as 10-100x when running on Azure, including:

- High-speed connectors to Azure storage services, such as Azure Blob Store and Azure Data Lake
- Auto-scaling and auto-termination of Spark clusters to minimize costs
- Caching
- Indexing
- Advanced query optimization

By providing an optimized, easy to provision and configure environment, Azure Databricks gives developers a performant, cost-effective platform that enables them to spend more time building applications, and less time focused on managing clusters and infrastructure.

### Collaboration

Notebooks in Azure Databricks are multi-editable, providing a live and shared means for data engineers and scientists to engage in real-time collaboration. Dashboards and integration with Power BI enable business users to call an existing jobs, passing in parameters.

### Easy to use

Azure Databricks features one-click setup and includes several features that allow you to get started quickly.

- Interactive notebooks to quickly learn the basics of Apache Spark
- An integrated debugging environment to let you analyze the progress of your Spark jobs from within interactive notebooks
- Powerful tools to analyze past jobs

### Familiar languages and libraries

Databricks supports Python, R, Scala, and SQL to allow developers to work with the data languages they are most familiar with to get started quickly.

In addition, common analytics libraries, such as the Python and R data science stacks, are preinstalled so that you can use them with Spark to derive insights. Other open source or cluster libraries can easily be imported using the UI, the CLI, or by invoking the Libraries API.

## How this guide is structured

This guide is structured so that your entry point to the content can be at the level of the common scenarios, or the technology choices for a particular scenario.

At the end of each article, a read next link is provided that you can follow to take a linear path through the guide. In addition, links to alternate and related content are provided to guide you to material that provides additional perspective on the options, as well as to provide links back up to the related parent topics and links to drill down into further detail.

### Next Steps

Read next: [Comparing Azure Databricks to HDInsight Spark](./overview/compare-to-hdinsight-spark.md)