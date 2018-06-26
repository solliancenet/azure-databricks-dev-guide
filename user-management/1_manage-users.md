# Azure Databricks User Managment

When you deploy an instance of the premium edition of Azure Databricks, you can plan and implement fine-grained access control of your deployment.  Controls include configuring object-level access permissions for users and roles.  You can implement control over workspace storage and other objects, such as cluster actions, tables, etc...

## Understanding User Roles

Users are grouped as follows: regular users and administrative users.  In Azure Databricks, there are three types of administrators: account owner, account admins, and cloud configuration admins.

1. **Account Owner** - The account owner can view and make changes to your Azure Databricks service and Azure subscription. This user is typically the person who signed up for or created your Azure Databricks service. If you don’t know who your account owner is, contact Azure Databricks support.  To see step-by-step instructions on how to manage an Azure Databricks account, including deleting an Azure Databricks service or cancelling an Azure subscription, see this [link](https://docs.azuredatabricks.net/administration-guide/account-settings/account.html)  

2. **Account Admins** - Account admins can manage users, workspace storage, and access control for your Azure Databricks instance. Admins can delegate admin privileges to other users.  

3. **Cloud Configuration Admins** - Cloud configuration admins manage networking for your Azure Databricks instance.

## Manage Users with Admin Console

The Azure Databricks Admin Console is the central location for administrators to manage users, workspace storage, and access control for their Azure Databricks instance.  To access the Admin Console click the Account user account icon at the top right of the product and then click on 'Admin Console' to open the Admin Console.

Manage Users
The following section describes how to add and remove users and manage their permissions.

Managing Users
Administrator users manage user accounts on the Admin Console page.

../../_images/users-list.png
An administrator can:

Add and remove users.
Grant and revoke permission to create clusters subject to cluster access control configuration.
Grant and revoke administrator rights by selecting the Admin checkbox.
In this workspace, William is an administrator and Greg can create clusters.

Add a user
Go to the Admin Console.

Click Add User.

Provide the user email ID.

You can add any user who belongs to the Azure Active Directory tenant of your Azure Databricks workspace.

../../_images/users-email-azure.png
If cluster access control is enabled, the user is added without cluster creation permission.

Remove a user




## AAD Authentication / AAD Conditonal Access / Guest Users

Azure Active Directory Authentication, Azure Active Directory Conditional Access
Guest users...
Role-based pass through access (this is a future topic)

## Next Steps

Read next: [The Databricks workspace](../workspace/workspace-overview.md)


----Integrate from here


Go to the Admin Console.
Click the Remove User Icon at the far right of the user row.
Click Remove User to confirm.

Single Sign-On
Single sign-on (SSO) in the form of Azure Active Directory-backed login is available in Azure Databricks for all customers. Here's a [link](https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnect-sso)

Managing Workspace Storage
Admin users manage workspace storage on the Workspace Storage tab in the Admin Console page.

Purge workspace
You can delete workspace objects such as entire notebooks, individual notebook cells, and individual notebook comments, but they are recoverable.

To permanently purge deleted workspace objects:

Click the Purge button.

../../_images/purge-workspace.png
Click Yes, purge to confirm.

IMPORTANT: Once purged, workspace objects are not recoverable.

Purge notebook revision history
To permanently purge notebook revision history:

In the Timeframe drop-down, select the timeframe to purge:
Click the Purge button.
../../_images/purge-revision-history.png
Click Yes, purge to confirm.
Warning

Once purged, revision history is not recoverable.

Purge cluster logs
To permanently purge Spark driver logs and historical metrics snapshots for all clusters in the workspace:

Click the Purge button.
../../_images/purge-cluster-logs.png
Click Yes, purge to confirm.
Warning

IMPORTANT: Once purged, cluster logs are not recoverable.

Manage Access Control
The following section describes how to set up access control and enable token-based authentication for your Azure Databricks instance.

Workspace Access Control
Note

Access control is available only in the Premium SKU.

By default, all users can create and modify workspace objects unless an administrator enables workspace access control. With workspace access control, individual permissions determine a user’s abilities. This topic describes the individual permissions and how to enable and configure workspace access control.

Workspace permissions
Enable workspace access control
Configure workspace permissions
Library and jobs access control
Workspace permissions
You can assign five permission levels to notebooks and folders: No Permissions, Read, Run, Edit, and Manage. The tables list the abilities for each permission.

Notebook Notebook permissions
Ability	No Permissions	Read	Run	Edit	Manage
View cells	 	x	x	x	x
Comment	 	x	x	x	x
Run commands	 	 	x	x	x
Attach/detach notebooks	 	 	x	x	x
Edit cells	 	 	 	x	x
Change permissions	 	 	 	 	x
Folder Folder permissions
Ability	No Permissions	Read	Run	Edit	Manage
View items	 	x	x	x	x
Create, clone, import, export items	 	x	x	x	x
Run commands on notebooks	 	 	x	x	x
Attach/detach notebooks	 	 	x	x	x
Delete items	 	 	 	x	x
Move/rename items	 	 	 	x	x
Change permissions	 	 	 	 	x
All notebooks in a folder inherit all permissions settings of that folder. For example, a user that has Run permission on a folder has Run permission on all notebooks in that folder.

Enable workspace access control
Go to the Admin Console.

Select the Access Control tab.

../../_images/access-control-tab.png
Click the Enable button next to Workspace Access Control.

Click Confirm to confirm the change.

Default permissions
Independent of workspace access control, the following permissions exist:

All users have Manage permission for items in the Workspace > Shared folder. You can grant Manage permission to notebooks and folders by moving them to the Shared folder.
All users have Manage permission for objects the user creates.
With workspace access control disabled, the following permissions exist:

All users have Edit permission for items in the Workspace folder.
With workspace access control enabled, the following permissions exist:

Workspace folder
Only administrators can create new items in the Workspace folder.
Existing items in the Workspace folder - Manage. For example, if the Workspace folder contained the Folder Documents and Folder Temp folders, all users continue to have the Manage permission for these folders.
New items in the Workspace folder - No Permissions.
A user has the same permission for all items in a folder, including items created or moved into the folder after you set the permissions, as the permission the user has on the folder.
User home directory - The user has Manage permission. All other users have No Permissions permission.
Configure workspace permissions
Open the permissions dialog:

Notebook - click Permissions in the notebook context bar.
Folder - select Permissions in the folder’s drop-down menu:
Permissions Drop Down
Grant permissions. All users in your account belong to the group all users. Administrators belong to the group admins, which has Manage permissions on all items.

To grant permissions to a user or group, select from the Add Users and Groups drop-down, select the permission, and click Add:

Add Users
To change the permissions of a user or group, select the new permission from the permission drop-down:

Change Permissions
Click Save Changes to save your changes or click Cancel to discard your changes.

Library and jobs access control
Library All users can view libraries. To control who can attach libraries to clusters, see Cluster Access Control.

Jobs To control who can run jobs and see the results of job runs, see Jobs Access Control.

Cluster Access Control
Note

Access control is available only in the Premium SKU.

By default, all users can create and modify clusters unless an administrator enables cluster access control. With cluster access control, permissions determine a user’s abilities. This topic describes the permissions and how to enable and configure cluster access control.

Types of permissions
Individual cluster permissions
Enable cluster access control
Configure cluster creation permission
Configure individual cluster permissions
Types of permissions
You can configure two types of cluster permissions:

Cluster creation permission controls your ability to create clusters.
Individual cluster permissions control your ability to use and modify a specific cluster.
When cluster access control is enabled:

An administrator can configure whether a user can create clusters.
Any user with Can Manage permission can configure whether a user can attach to, restart, resize, and manage existing clusters.
Individual cluster permissions
There are four permission levels for a cluster: No Permissions, Can Attach To, Can Restart, and Can Manage. The table lists the abilities for each permission.

Ability	No Permissions	Can Attach To	Can Restart	Can Manage
Attach notebook to cluster	 	x	x	x
View Spark UI	 	x	x	x
View cluster metrics	 	x	x	x
Terminate cluster	 	 	x	x
Start cluster	 	 	x	x
Restart cluster	 	 	x	x
Edit cluster	 	 	 	x
Attach library to cluster	 	 	 	x
Resize cluster	 	 	 	x
Modify permissions	 	 	 	x
Note

You have Can Manage permission for any cluster that you create.

Enable cluster access control
Go to the Admin Console.

Select the Access Control tab.

../../_images/access-control-tab.png
Click the Enable button next to Cluster and Jobs Access Control.

ClusterAndJobsACLs

Click Confirm to confirm the change.

Configure cluster creation permission
Cluster access control must be enabled.

Go to the Admin Console.

In the Users page, select or deselect the Allow cluster creation checkbox in the user’s row.

../../_images/users-list.png
Click Confirm to confirm the change.

Configure individual cluster permissions
Cluster access control must be enabled and you must have Can Manage permission for that cluster.

Click the clusters icon Clusters Menu Icon in the sidebar.

Click the Permissions Icon lock icon under the Actions column of an existing cluster.

ClusterACLsButton
In the pop-up dialog box, assign cluster permissions using the drop-down menu beside a user’s name.

IndvClusterACLs
Click Done.

Jobs Access Control
Note

Access control is available only in the Premium SKU.

By default, all users can create and modify jobs unless an administrator enables jobs access control. With jobs access control, individual permissions determine a user’s abilities. This topic describes the individual permissions and how to enable and configure jobs access control.

Job permissions
Enable jobs access control
Configure job permissions
Job permissions
There are five permission levels for jobs: No Permissions, Can View, Can Manage Run, Is Owner, and Can Manage. The Can Manage permission is reserved for administrators. The table lists the abilities for each permission.

Ability	No Permissions	Can View	Can Manage Run	Is Owner	Can Manage (admin)
View job details and settings	x	x	x	x	x
View results, Spark UI, logs of a job run	 	x	x	x	x
Run now	 	 	x	x	x
Cancel run	 	 	x	x	x
Edit job settings	 	 	 	x	x
Modify permissions	 	 	 	x	x
Delete job	 	 	 	x	x
Change owner	 	 	 	 	x
Note

The creator of a job has Is Owner permission.
Jobs triggered through Run Now assume the permissions of the job owner and not the user who issued Run Now. For example, even if job A is configured to run on an existing cluster accessible only to the job owner (user A), a user (user B) with Can Manage Run permission can start a new run of the job.
You can view notebook run results only if you have the Can View or higher permission on the job. This allows jobs access control to be intact even if the job notebook was renamed, moved, or deleted.
Jobs access control applies to jobs displayed in the Databricks Jobs UI and their runs. It doesn’t apply to runs spawned by notebook workflows or runs submitted by API whose ACLs are bundled with the notebooks.
Enable jobs access control
Go to the Admin Console.

Select the Access Control tab.

../../_images/access-control-tab.png
Click the Enable button next to Cluster and Jobs Access Control.

ClusterAndJobsACLs

Click Confirm to confirm the change.

Configure job permissions
You must be an administrator or have Is Owner permission.

Go to the details page for a job.

Click Advanced.

../../_images/job-advanced.png
Click the Edit link next to Permissions.

../../_images/job-permissions.png
In the pop-up dialog box, assign job permissions via the drop-down menu beside a user’s name.

JobManageACL

Click Save Changes.

 Table Access Control
Table access control (table ACLs) lets you programmatically grant and revoke access to your data from SQL, Python, and PySpark.

By default, all users have access to all data stored in a cluster’s managed tables unless an administrator enables table access control for that cluster. Once table access control is enabled for a cluster, users can set permissions for data objects on that cluster.

Topics:

Enable Table Access Control
Set Permissions on a Data Object
Note: Access control requires the Premium SKU.

Enable Table Access Control
Note

Access control requires the Premium SKU.

Table access control lets you programmatically grant and revoke access to your data using the Azure Databricks view-based access control model.

To set up table access control, you must:

Step 1: Enable table access control for a cluster
Step 2: Enforce table access control
This topic describes how to enable and enforce table access control.

For information about how to set permissions on a data object once table access control is enabled, see Set Permissions on a Data Object.

Step 1: Enable table access control for a cluster
Table access control is available in two versions:

SQL-only table access control, which:
Is generally available.
Restricts cluster users to SQL commands. Users are restricted to the SparkSQL API, and therefore cannot use Python, Scala, R, RDD APIs, or clients that directly read the data from cloud storage, such as DBUtils.
Requires that clusters run Databricks Runtime 3.1 or above.
Python and SQL table access control (beta), which:
Is in beta.
Allows users to run SQL, Python, and PySpark commands. Users are restricted to the SparkSQL API and DataFrame API, and therefore cannot use Scala, R, RDD APIs, or clients that directly read the data from cloud storage, such as DBUtils.
Requires that clusters run Databricks Runtime 3.5 or above.
SQL-only table access control
This version of table access control restricts users on the cluster to SQL commands only.

To enable SQL-only table access control on a cluster and restrict that cluster to use only SQL commands, set the following flag in the cluster’s Spark conf:

Copy to clipboardCopy
spark.databricks.acl.sqlOnly true
Python and SQL table access control (beta)
This version of table access control lets users run Python and PySpark commands that use the DataFrame API, as well as SQL. When it is enabled on a cluster or serverless pool, users on that cluster or pool:

Can access Spark only via the Spark SQL API or DataFrame API. In both cases, access to tables and views is restricted by administrators according to the Azure Databricks View-based access control model.
Cannot acquire direct access to data in the cloud via DBFS or by reading credentials from the cloud provider’s metadata service.
Must run their commands on cluster nodes as a low-privilege user forbidden from accessing sensitive parts of the filesystem or creating network connections to ports other than 80 and 443.
Attempts to get around these restrictions will fail with an exception. These restrictions are in place so that your users can never access unprivileged data through the cluster.

There are two steps to enabling a cluster or serverless pool for Python and SQL table access control.

Enable table access control at the account level
Create a cluster enabled for table access control
Enable table access control at the account level
Note

Starting with Databricks Platform version 2.68, all access controls are enabled by default for new accounts in the Premium SKU. For accounts that were opened before version 2.68, an administrator must enable table access control using the steps listed in this section.

Log in to the Admin Console.
Go to the Access Control tab.
Ensure that Cluster Access Control is enabled. You cannot enable table access control without having cluster access control already enabled.
Next to Table Access Control, click the Enable button.
Create a cluster enabled for table access control
When you create a cluster, click the Enable table access control and only allow Python and SQL commands option.

../../../_images/table-acl-enable-cluster-azure.png
To create the cluster using the REST API, see Enable table access control example.

Step 2: Enforce table access control
To ensure that your users access only the data that you want them to, you must restrict your users to clusters with table access control enabled. In particular, you should ensure that:

Users do not have permission to create clusters. If they create a cluster without table access control, they can access any data from that cluster.

../../../_images/table-acl-no-allow-cluster-create-azure.png
Users do not have Can Attach To permission for any cluster that is not enabled for table access control.

See Cluster Access Control for more information.

Now you can Set Permissions on a Data Object.

Set Permissions on a Data Object
Note

Access control requires the Premium SKU.

The Azure Databricks view-based data governance model lets you programmatically grant and revoke access to your data from the Spark SQL API.

This data governance model lets you control access to securable objects like tables, databases, views, and functions. It also allows for fine-grained access control (to a particular subset of a table, for example) by setting permissions on derived views created from arbitrary queries. The Azure Databricks SQL query analyzer enforces these access control policies at runtime on clusters with table access control enabled.

This topic describes the privileges, objects, and ownership rules that make up the Azure Databricks view-based data access control model. It also describes how to grant and revoke object privileges.

In this topic:

Requirements
View-based access control model
Grant and revoke object privileges
Frequently asked questions
Requirements
Before you can grant or revoke privileges on data objects, an administrator must enable table access control for the cluster. For information about how to create a cluster that follows this security model, see Enable Table Access Control.

View-based access control model
This section describes the Azure Databricks view-based data access control model.

In this section:

Privileges
Objects
Object ownership
Users and groups
Privilege hierarchy
Privileges
SELECT – gives read access to an object
CREATE – gives ability to create an object (for example, a table in a database)
MODIFY – gives ability to add/delete/modify data to/from an object
READ_METADATA – gives ability to view an object and its metadata
CREATE_NAMED_FUNCTION – gives ability to create a named UDF in an existing catalog or database
ALL PRIVILEGES – gives all privileges (gets translated into all the above privileges)
Objects
The privileges above can apply to the following classes of objects:

CATALOG - controls access to the entire data catalog.
DATABASE - controls access to a database.
TABLE - controls access to a managed or external table.
VIEW - controls access to SQL views.
FUNCTION - controls access to a named function.
ANONYMOUS FUNCTION - controls access to anonymous or temporary functions.
ANY FILE - controls access to the underlying filesystem.
Object ownership
For some actions, the ownership of the object (table/view/database) determines if you are authorized to perform the action. The user who creates the table, view, or database becomes its owner. In the case of tables and views, the owner is granted all privileges, and can grant those privileges to other users.

For example, ownership determines whether or not you can grant permissions on derived objects to other users. Suppose User A owns Table T and grants User B SELECT permission on Table T. Now, even though User B can select from Table T, User B cannot grant SELECT permission on Table T to User C, because User A is still the owner of the underlying Table T.

Furthermore, User B cannot circumvent this restriction simply by creating a View V on Table T and granting permissions on that view to User C. When Databricks checks for permissions for User C to access View V, it also checks that the owner of V and underlying Table T are the same. If the owners are not the same, User C must also have select permissions on underlying Table T.

To summarize, the owner of an object controls access to that object. This applies to tables as well as to views that are used for fine-grained access to tables.

Users and groups
Privileges can be granted to users or groups that are created via the groups API. Each user is uniquely identified by their username (which typically maps to their email address) in Databricks. Users who are workspace administrators in Databricks belong to a special admin role and can also access objects that they haven’t been given explicit access to.

Privilege hierarchy
Privileges on objects are hierarchical. This means that granting a privilege on the entire CATALOG automatically grants the privilege to all of the databases (as well as all tables and views). Similarly, granting a privilege to a given DATABASE automatically grants the privilege to all tables and views in that database.

Grant and revoke object privileges
In this section:

Commands
View-based access control
Commands
You use the following commands to manage object privileges:

GRANT
Copy to clipboardCopy
GRANT
  privilege_type [, privilege_type ] ...
  ON (CATALOG | DATABASE db_name | [TABLE] table_name | [VIEW] view_name | [FUNCTION] function_name | ANONYMOUS FUNCTION | ANY FILE)
  TO user [, user] ...

privilege_type
  : SELECT | CREATE | MODIFY | READ_METADATA | CREATE_NAMED_FUNCTION | ALL PRIVILEGES
REVOKE
Copy to clipboardCopy
REVOKE
  privilege_type [, privilege_type ] ...
  ON (CATALOG | DATABASE db_name | [TABLE] table_name | [VIEW] view_name | [FUNCTION] function_name | ANONYMOUS FUNCTION | ANY FILE)
  FROM user [, user] ...

privilege_type
  : SELECT | CREATE | MODIFY | READ_METADATA | CREATE_NAMED_FUNCTION | ALL PRIVILEGES
Examples

Copy to clipboardCopy
GRANT SELECT ON table_name to `user1@databricks.com`;
GRANT SELECT ON ANONYMOUS FUNCTION to `user2@databricks.com`;
GRANT SELECT ON ANY FILE to `user3@databricks.com`;
REVOKE ALL PRIVILEGES ON DATABASE default FROM `user4@databricks.com`
Note

Azure Databricks does not support an explicit DENY command for objects.

SHOW GRANT
Copy to clipboardCopy
SHOW GRANT [user] ON (CATALOG | DATABASE db_name | [TABLE] table_name | [VIEW] view_name | [FUNCTION] function_name | ANONYMOUS FUNCTION | ANY FILE)
Example

Copy to clipboardCopy
SHOW GRANT `user1@databricks.com` ON DATABASE default
View-based access control
You can configure fine-grained access control (to rows and columns matching specific conditions, for example) by granting access to derived views that contain arbitrary queries.

Example

Copy to clipboardCopy
CREATE OR REPLACE VIEW view_name AS SELECT columnA, columnB FROM table_name WHERE columnC > 1000;
GRANT SELECT ON VIEW view_name to `user1@databricks.com`;
Privileges required for SQL operations
The following table roughly maps privileges to SQL operations:

Operations ↓ / Privilege →	SELECT	CREATE	MODIFY	READ_METADATA	CREATE_NAMED_FUNCTION	Ownership	Admin
CREATE TABLE	 	x	 	 	 	x	x
DROP TABLE	 	 	x	 	 	x	x
DESCRIBE TABLE	 	 	 	x	 	x	x
ALTER TABLE	 	 	x	 	 	x	x
DROP TABLE	 	 	x	 	 	x	x
CREATE VIEW	 	x	 	 	 	x	x
DROP VIEW	 	 	x	 	 	x	x
SELECT	x	 	 	 	 	x	x
CREATE FUNCTION	 	 	 	 	x	x	x
MSCK	 	 	 	 	 	x	x
CREATE DATABASE	 	x	 	 	 	x	x
ExPLAIN	 	 	 	x	 	x	x
DROP DATABASE	 	 	x	 	 	x	x
GRANT	 	 	 	 	 	x	x
REVOKE	 	 	 	 	 	x	x
Frequently asked questions
I created a table/view but now I can’t query it, drop it or modify it.
This error may occur because you created that table on a cluster without Table ACLs enabled. When Table ACLs are disabled on a cluster, owners are not registered when a table, view, or database is created. To mitigate this problem an admin must assign an owner to the table via the following command:

Copy to clipboardCopy
ALTER TABLE <table-name> OWNER TO `<user-name>@<user-domain>.com`;
If you are not the new owner, the owner or an admin must provide SELECT permission to you via the following command:

Copy to clipboardCopy
GRANT SELECT ON TABLE <table-name> to `<user-name>@<user-domain>.com`;
How do I grant permissions on global and local temporary views?
Unfortunately permissions on global and local temporary views are not supported. Local temporary views will only be visible within the same session, and views created in the global_temp database will be visible to all users sharing a cluster. However, do note that permissions on the underlying tables and views referenced by any temporary views, will be enforced.
How do I grant a user or group permissions on multiple tables at once?
A grant or revoke statement can only be applied to one object at a time. The recommended way to organize and grant permissions on multiple tables to a principal is via databases. Granting a principal SELECT permission on a database implicitly grants that principal SELECT permissions on all tables and views in that database. For example, if a database D has tables T1 and T2, and an admin issues the following grant command:

Copy to clipboardCopy
GRANT SELECT ON DATABASE D to `<user>@databricks.com`
the principal <user>@databricks.com can now select from tables T1 and T2, as well as any tables and views that will be created in database D in the future.

How do I grant a user permissions on all tables except one?
If you want to deny a principal permissions on a particular table, the only way to do this is by issuing grant commands for the tables and databases that the principal should in fact have permissions on. Databricks does not support a DENY command.
I granted a user SELECT permissions on a view, but when that user tries to SELECT from that view, they get the error User does not have permission SELECT on table.
This common error is caused by the way Object ownership affects the ability to grant permissions. You see this behavior because you are not the owner of Table T, for one of the following reasons:

Someone else created Table T and is the owner of Table T.
Table T has no registered owner because it was created using a cluster for which Table ACL is disabled.
Suppose there is a View V on Table T and User U tries to select from View V. Databricks checks that:

User U has SELECT permission on Table T.
One of the following is true:
View V and Table T have the same owner. In this case the owner of Table T has granted fine-grained access to user U.
User U can select directly from Table T. If the owner of Table T has allowed User U to select from all of Table T, User U can select from any view on Table T.
As described in the Object ownership section, these conditions ensure that only the owner of an object can grant other users access to that object.

You can test if a table has an owner by using SHOW GRANT ON <table-name>. If you do not see an entry with ActionType OWN, then the table does not have an owner. If this is the case, any admin user can set the table owner by running the following:

Copy to clipboardCopy
ALTER TABLE <table-name> OWNER TO `<user-name>@<user-domain>.com`;
I tried to run sc.parallelize on a cluster with Table ACLs enabled and it doesn’t work.
On clusters with Table ACLs enabled users can use only the Spark SQL and Python DataFrame APIs. The RDD API is disallowed for security reasons, since Databricks does not have the ability to inspect and authorize code within an RDD.

Enable Token-based Authentication
Azure Databricks administrators must enable token-based authentication in order for users to generate personal access tokens and authenticate to the Databricks REST API. Personal access tokens have an expiration date and can be revoked.

Token-based authentication is disabled by default.

For more information about using token-based authentication, see Authentication.

Enable tokens
Go the Admin Console.

Select the Access Control tab.

../../_images/access-control-tab.png
Click the Enable button next to Personal Access Tokens.

Click Confirm to confirm the change.





