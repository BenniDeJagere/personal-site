---
title: "Microsoft Fabric's Auto Discovery: a closer look"
date: 2023-06-28T09:00:00+02:00
tags: ["fabric", "delta", "lakehouse", "onelake", "shortcuts", "azure", "data lake", "data warehouse", "data engineering", "spark", "delta lake", "adls"]
images: ["/img/post/2023/fabric_discovery/empty_lakehouse.png"]
---

In [previous posts](/tags/fabric/), I dug deeper into Microsoft Fabric's SQL-based features and we even [explored OneLake using Azure Storage Explorer]({{< relref "blog/exploring-onelake-with-explorer.md" >}}). In this post, I'll take a closer look at Fabric's **auto-discovery** feature using Shortcuts. Auto-discovery, what's that?

> Fabric's Lakehouses can automatically discover all the datasets already present in your data lake and expose these as tables in Lakehouses (and Warehouses).

Cool, right? At the time of writing, there is a single condition: the tables must be stored in the Delta Lake format. Let's take a closer look.

## Existing data lakes

In this example, I'm starting with an existing data lake that already holds 2 datasets: `green_taxi` and `yellow_taxi`. Both are stored in Delta Lake format in a container called `delta` in an Azure Storage account with Hierarchical Namespaces (Azure Data Lake Storage gen2) enabled and an empty Lakehouse on Fabric.

![data lake](/img/post/2023/fabric_discovery/data_lake.png "existing data lake")

![empty lakehouse](/img/post/2023/fabric_discovery/empty_lakehouse.png "empty Lakehouse")

Fabric is able to link your existing data lakes to the OneLake using a feature called Shortcuts. Shortcuts are logical links pointing to your existing data lakes on Azure (ADLS gen2), GCP (GCS) or AWS (S3). In this example, I'm using Azure Storage. At the time of writing, GCS is not available yet but the process is identical.

## Creating a Shortcut

We can create a Shortcut by right-clicking on the Tables node or by clicking the green button in the work area on the screen. A pop-up appears and we can enter the URL pointing to our existing tables. For ADLS, make sure to use the DFS endpoint and not the blob endpoint. The ADLS URL takes the form `https://<storage_account_name>.dfs.core.windows.net/<container_name>/<path_to_dataset>`. If it's the first time we're creating a Shortcut to this path or any of its parent folders, we'll also have to create a new Connection and tell Fabric how we're authenticating. Possible authentication methods include Azure AD accounts, Service Principals and Shared Access Signatures. To keep it simple, I'm using my own Azure AD account.

![First step of creating a Shortcut](/img/post/2023/fabric_discovery/shortcut_first_step.png "First step of creating a Shortcut")

After confirming the new connection and signing in using Azure AD for authentication, we can proceed to the next step. Here we give our Shortcut a name and point to the exact path inside the storage container where our dataset is located. The name of the Shortcut will also become the name of the table in the Lakehouse. We have to repeat this for every table we want to link to the Lakehouse.

![Second step linking our green_taxi dataset](/img/post/2023/fabric_discovery/shortcut_green_taxi.png "Second step linking our green_taxi dataset")

![Second step linking our yellow_taxi dataset](/img/post/2023/fabric_discovery/shortcut_yellow_taxi.png "Second step linking our yellow_taxi dataset")

If all goes well and Fabric recognizes the Delta Lake dataset at this location, it appears in the *Tables* node of the Lakehouse and we can start exploring and visualizing the data.

![yellow_taxi in the Lakehouse](/img/post/2023/fabric_discovery/delta_load_succeeded.png "yellow_taxi in the Lakehouse")

## Querying the data

Now that we have our data in the Lakehouse, we can start querying it. This can be done using Spark SQL through the Notebooks or Spark Jobs experience or with T-SQL through the SQL Endpoint we get from the Lakehouse. The Lakehouse can even be directly queried from Fabric's Warehouse. **Queries run instantly even though we never ingested a single byte of data into the OneLake!**

![Querying the data](/img/post/2023/fabric_discovery/sql_endpoint.png "Querying the data")

## What if it doesn't work?

What can go wrong? Our data lake might not be accessible at the given URL with the given authentication method. In this case, it's up to you to verify connectivity and authentication.

Something else you might run into is an incompatible version of Delta Lake. See, Delta Lake is an open standard created by Databricks and often receives updates with new features. Therefore, the designers of Delta Lake made it impossible to read delta tables which have been created by a newer version of the Delta Lake library than the one you're using. This prevents data corruption and data loss, but it might cause some headaches when you're trying to link your existing data lake to the Lakehouse. When a table is not properly recognized because of this version mismatch or other compatibility issues, you'll see an *Unidentified* node appearing in the *Tables* section.

![Unidentified node](/img/post/2023/fabric_discovery/unidentified.png "Unidentified node")

You can open this item to spot the error message. If your data was created with an incompatible version of Delta Lake, you'll see a message like this:

![Delta version mismatch](/img/post/2023/fabric_discovery/delta_version_error.png "Delta version mismatch")

Your possible options here are to either recreate the delta table using a compatible version or to wait until Fabric supports the version you're using. You can easily look up the version of Delta Lake that Fabric is using by going to your Workspace Settings. The settings can be found by right-clicking on your Workspace in the Workspaces pane which opens when you click the *Workspaces* button in the left sidebar. Continue by clicking on the Data Engineering/Science option to make the *Spark compute* settings appear. This is where you can configure what's behind the scenes of the Lakehouses in your workspace. Lakehouses are built upon Apache Spark and Microsoft bundled Spark, Delta and a bunch of other packages in a concept called a *Runtime*. The Runtime defines the versions you're using and you can quickly spot the installed version of Delta Lake in the name of the Runtime.

![Check your Delta Lake version](/img/post/2023/fabric_discovery/check_delta_version.png "Check your Delta Lake version")

In the above screenshot, you can see I'm using Runtime 1.1 which supports Apache Spark 3.3 and Delta Lake 2.2. At the time of writing the latest version of Spark is 3.4 and the latest version of Delta Lake is 2.4. So if I want to use Delta Lake 2.3 or 2.4, I'll have to wait until Microsoft updates the Runtime to version 1.2 or higher.

## Conclusion

Right now it can be a bit of a hassle to link all of your existing tables one by one using the UI, but Microsoft is working on ways to automate this using CLIs or APIs. The Auto-Discovery feature of Fabric will make migration to Fabric a breeze and takes away the burden of having to ingest all of your data into the OneLake. It's a great feature and I'm looking forward to seeing it evolve.
