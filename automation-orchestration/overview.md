# Automation and Orchestration

Most big data solutions consist of repeated data processing operations, encapsulated in workflows. A pipeline orchestrator is a tool that helps to automate these workflows. An orchestrator can schedule jobs, execute workflows, and coordinate dependencies among tasks.

Automation in this context is the act of taking a manual process, such as managing a cluster through the Azure Databricks UI, and making it repeatable and configurable with parameters through scripting.

## Pre-requisites

The sub-topics below require you to use a generated [Personal Access Token](https://docs.azuredatabricks.net/administration-guide/admin-settings/tokens.html) for authentication. Follow the steps in [Setup](setup.md) to create one if needed.

## Sub-topics

- **Automation**
  - [REST API](rest-api.md)
  - [Databricks CLI](databricks-cli.md)
- **Orchestration**
  - [Azure Data Factory](azure-data-factory.md)
  - [Apache Airflow](apache-airflow.md)
