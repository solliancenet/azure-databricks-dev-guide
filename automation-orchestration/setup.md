# Automation and orchestration setup

The first step you must complete before you can use automation in Azure Databricks, such as the REST API, CLI, and integrated services in Azure Data Factory, is to generate a personal access token. Before you can do this, your administrator must enable personal access tokens for your organization's Azure Databricks account, as token-based authentication is disabled by default.

### Enable token-based authentication

Log in to the Azure Databricks workspace as an administrator, then follow these steps to enable personal access tokens:

1.  Access the Admin Console by clicking on the user **Account** icon and selecting **Admin Console**.

    ![Click on the user account icon and select Admin Console](media/access-admin-console.png 'Admin Console under user account')

2.  Select the **Access Control** tab.

    ![Select the Access Control tab](media/access-control-tab.png 'Access Control tab')

3.  Select the **Enable** button next to **Personal Access Tokens**.

4.  Select **Confirm** to confirm your change.

### Create an access token

Personal access tokens are similar to passwords; you should treat them with care. They expire after 90 days by default and can be revoked. Generate an access token from the Azure Databricks workspace, following these steps:

- In your Azure Databricks workspace, select the **Account** icon in the top right corner, and then select **User Settings**.

  ![Click on the user account icon and select User Settings](media/access-user-settings.png 'User Settings under user account')

- On the **User Settings** screen, select **Generate New Token**.

  ![Azure Databricks User Settings screen](media/azure-databricks-account-user-settings.png 'Azure Databricks User Settings')

- In the **Generate New Token** dialog, enter a comment, such as "REST API", and then select **Generate**.

  ![The Azure Databricks Generate New Token dialog is displayed, with "REST API" entered into the Comment field, and the Lifetime set to 90 days.](media/generate-new-token-rest.png 'Generate New Token')

- Copy the generated token and save it for later. Once you've copied and pasted the token value, you can select Done on the Generate New Token dialog.

  ![The Azure Databricks Generate New Token dialog is displayed, with a message stating, "Your token has been created successfully." The generated token is selected.](media/azure-databricks-copy-token.png 'Copy generated token')
