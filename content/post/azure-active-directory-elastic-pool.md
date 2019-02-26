+++
title = "Azure Active Directory with Azure SQL"
categories = ["azure", "sql server", "active directory", "elastic pool"]
description = "Access Azure SQL Server in Elastic Pool via Azure Active Directory."
date = "2017-12-04"
aliases = ["articles/azure-active-directory-elastic-pool"]
+++

>
> This post describes how to set up user access to _Azure SQL Server_ within an *Elastic Pool* via *Azure Active Directory*.  Specifically, I want to allow an AD user to connect to an Azure-hosted database with his own Connection String.
>

So you've got users and groups in Azure Active Directory, and you have one or more Azure SQL Servers in an Elastic Pool.  How can you grant those AD Users access to your SQL Server databases?

First, each SQL Server or Elastic Pool has to have an "Active Directory Admin" assigned.  If there's no AD Admin, this won't work.

# Create the Admin

The AD Admin Setup for Elastic Pool wasn't obvious to me---I had to ask Microsoft Support where it was:

1. Select your existing Elastic Pool in the portal

2. From "Overview", click on your "Server Name"

3. Select "Active Directory Admin" -> "Set Admin".  You can set a single user or an AD Group as the Administrator.

{{<figure src="/images/azure/elastic_pool_1.png" caption="Add an AD Admin" caption-effect="fade" caption-position="bottom">}}

You can also do this [from the CLI](https://docs.microsoft.com/en-us/cli/azure/sql/server/ad-admin?view=azure-cli-latest#az_sql_server_ad_admin_create):

```bash
az sql server ad-admin create --object-id &lt;Object-Id-OfUserOrGroup&gt;
    -s &lt;Database-Name&gt;
    -g &lt;Resource-Group&gt;
    -u &lt;NameOrEmailAkaDisplayName&gt;
```

# Create the User or Group in SQL Server

You should now be able to log in to a database in the Elastic Pool
with your AD Admin login via SSMS.  In this case I log in via "Active 
Directory Password Authentication", but I could have used
"Universal Authentication" or "Integrated Authentication"
instead:

{{<figure src="/images/azure/ssms_login_azure.png" caption="Add an AD Admin" caption-effect="fade" caption-position="bottom">}}

Once you're logged in as the AD Administrator, you can create
a new SQL Server USER that corresponds to an AD Group.  I've already
created an AD group called "SQL Developers" so I mapped it to a SQL
Server database and added `db_datareader` permission like this:

```sql
CREATE USER [SQL Developers] FROM EXTERNAL PROVIDER 
ALTER ROLE db_datareader ADD MEMBER [SQL Developers]
```

For some reason, when I re-logged in as the AD Admin after being already
logged-in via the admin connection string, it gave me the error *Principal 'SQL Developers' could not be created. Only connections established with Active Directory accounts can create other Active Directory users.*.  I logged out and restarted SSMS and then it worked fine.

## Login as a "SQL Developer"

At that point, I was able to log out from the AD Admin account and then reconnect to my database from SSMS using one of the AD logins from my "SQL Developers" group.  This also allows me to log in with a personalized Connection string:

<pre>
Server=tcp:example.database.windows.net,1433;
Initial Catalog=example-database;Persist Security Info=False;
User ID=mike@example.org;Password=mypassword;MultipleActiveResultSets=False;
Encrypt=True;TrustServerCertificate=False;
Authentication="Active Directory Password";
</pre>


