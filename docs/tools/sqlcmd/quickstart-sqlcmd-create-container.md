---
title: "Quickstart: Create a local copy of a database in a container using sqlcmd"
description: A quickstart that walks through using creating a new container and restoring a database
author: dlevy-msft
ms.author: dlevy
ms.reviewer: maghan, randolphwest
ms.date: 09/12/2023
ms.service: sql
ms.subservice: tools-other
ms.topic: quickstart
monikerRange: ">=aps-pdw-2016 || =azuresqldb-current || =azure-sqldw-latest || >=sql-server-2016 || >=sql-server-linux-2017"
---

# Quickstart: Create a new local copy of a database in a container with sqlcmd

In this quickstart, you'll use a single command in **sqlcmd** to create a new container, and restore a database to that container to create a new local copy of a database, for development or testing.

## Prerequisites

- A container runtime installed, such as [Docker](https://www.docker.com/) or [Podman](https://podman.io/)
- Install the latest **sqlcmd**

## Remarks

Installing **sqlcmd** (Go) via a package manager replaces **sqlcmd** (ODBC) with **sqlcmd** (Go) in your environment path. Any current command line sessions need to be closed and reopened for this change to take to effect. **sqlcmd** (ODBC) isn't removed, and can still be used by specifying the full path to the executable.

You can also update your `PATH` variable to indicate which version takes precedence. To do so in Windows 11, open **System settings** and go to **About > Advanced system settings**. When **System Properties** opens, select the **Environment Variables** button. In the lower half, under **System variables**, select **Path** and then select **Edit**. If the location **sqlcmd** (Go) is saved to (`C:\Program Files\sqlcmd` is default) is listed before `C:\Program Files\Microsoft SQL Server\<version>\Tools\Binn`, then **sqlcmd** (Go) is used.

You can reverse the order to make **sqlcmd** (ODBC) the default again.

## Download and install sqlcmd (Go)

[!INCLUDE [install-go](includes/install-go.md)]

## What problem will we solve?

This quickstart walks through the process of creating a local copy of a database, then querying it to check for data quality issues.

## Create the local copy

Create a new [!INCLUDE [ssnoversion-md](../../includes/ssnoversion-md.md)] instance in a container using the latest version of [!INCLUDE [ssnoversion-md](../../includes/ssnoversion-md.md)]. The command also restores the `WideWorldImporters` database.

1. Open a new terminal window and run the following command:

    ```bash
    sqlcmd create mssql --accept-eula --using https://github.com/Microsoft/sql-server-samples/releases/download/wide-world-importers-v1.0/WideWorldImporters-Full.bak
    ```

## Query the database in Azure Data Studio

Open [!INCLUDE [name-sos-short](../../includes/name-sos-short.md)] and have a look at the data.

1. In the same terminal window, run the following command:

   ```bash
   sqlcmd open ads
   ```

1. Now that you have a local copy of your database, you can run a few queries. Here are a few queries you can use to check for data quality issues:

   ```sql
   --Look for customers that have ordered but not been billed for anything
   SELECT *
   FROM Sales.Customers c
   INNER JOIN Sales.Orders o
       ON c.CustomerID = o.CustomerID
   LEFT JOIN Sales.Invoices i
       ON c.CustomerID = i.CustomerID
   WHERE i.CustomerID IS NULL;

   --Look for customers that have not been billed for anything
   SELECT *
   FROM Sales.Customers c
   LEFT JOIN Sales.Invoices i
       ON c.CustomerID = i.CustomerID
   WHERE i.CustomerID IS NULL;

   --Look for invoices without a customer
   SELECT *
   FROM Sales.Customers c
   RIGHT JOIN Sales.Invoices i
       ON c.CustomerID = i.CustomerID
   WHERE c.CustomerID IS NULL;

   --Look for orders without a customer
   SELECT *
   FROM Sales.Customers c
   RIGHT JOIN Sales.Orders o
       ON c.CustomerID = o.CustomerID
   WHERE c.CustomerID IS NULL;
   ```

## How did we solve the problem?

You were able to quickly create a local copy of a database for development and testing purposes. With a single command, you created a new local instance and restored the most recent backup to it. You then ran another command to connect to it via Azure Data Studio. You then queried the database using [!INCLUDE [name-sos-short](../../includes/name-sos-short.md)] to check for data quality issues.

## Clean up resources

When you're done trying out the database, delete the container with the following command:

```bash
sqlcmd delete --force
```

The `--force` flag is used here for convenience since we are in a demo environment. In most cases, it's better to leave the `--force` flag off to make sure you aren't inadvertently deleting a database you don't mean to.

## Next steps

- [Create and query a SQL Server container](sqlcmd-use-utility.md#create-and-query-a-sql-server-container)
