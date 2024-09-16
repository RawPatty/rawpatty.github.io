---
layout: post
title:  "Copying Azure SQL Databases across Subscriptions and Servers"
date:   2022-06-18 08:47:29 +0000
categories: [Azure]
excerpt: "Quick method to copy Azure SQL databases across different SQL servers and Subscriptions."

---
In a working environment, oftentimes data needs to be copied from the production database to lower environments.

Below I will show a quick method to copy Azure SQL databases across different SQL servers and subscriptions.

If you have been exporting and importing databases in the past, this method will be much faster and guarantees transaction consistency with Azure’s copy functionality. While in the GUI it looks like you are only able to copy to other servers in the subscription, you can circumvent this restriction with SSMS.

## Prerequisites

You will need administrative access to both Azure SQL servers, and be able to connect to them via SSMS.

## Method

In SSMS Connect to both the source and target SQL servers

In the source server, add the admin account of the target server with DB owner permissions, here are the commands:

Master database

{% highlight ruby %}
CREATE LOGIN sourceadmin (replace this with your target’s admin account name) WITH PASSWORD = ‘**********’;` (Choose a password)
{% endhighlight %}

On the target database:

{% highlight ruby %}
CREATE USER sourceadmin FROM LOGIN sourceadmin ;
exec sp_addrolemember ‘db_owner’, ‘sql1uvadmin’;
GO
{% endhighlight %}

Once the permissions have been set, run the below line to create a new DB

`CREATE DATABASE lowerEnvDBname AS COPY OF SourceServer.DatabaseName;`

That’s it!

You may (or may not) want to clean up the account you created in your production environment at the start
