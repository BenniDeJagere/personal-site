---
title: "Connect to Fabric Lakehouses & Warehouses from Python code"
date: 2023-06-27T23:14:24+02:00
draft: true
tags: ["Microsoft Fabric", "Python", "ODBC", "data lakehouse", "data warehouse", "SQL"]
---

In this post, I will show you how to connect to your Microsoft Fabric Lakehouses and Warehouses from Python.

<!--more-->

## Packages & dependencies

To connect to Fabric, we'll use the Microsoft ODBC Driver. This driver is available for Windows, Linux, and macOS. Click on your operating system to download and install the driver:

* [Windows](https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver15#download-for-windows)
* [Linux](https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver15&tabs=alpine18-install%2Calpine17-install%2Cdebian8-install%2Credhat7-13-install%2Crhel7-offline#18)
* [macOS](https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos?view=sql-server-ver15#microsoft-odbc-18) (uses Homebrew)

Next, we'll need a Python package to connect using ODBC and a Python package to authenticate with Azure Active Directory. We can install them like so:

```bash
pip install pyodbc azure-identity
```

## Authentication

Next, we have to decide how you want to authenticate. Fabric relies on Azure Active Directory for authentication. While in the future it will be possible to use fancy mechanisms like Service Principals and Managed Identities, for now, you can only authenticate as yourself. This still leaves us with many authentication options on the table:

* Using your login session from the Azure CLI
* Using your login session from the Azure Developer CLI
* Using your login session from Azure PowerShell
* Using your login session from Visual Studio Code
* Opening a browser to authenticate

All of the options above use an external factor to authenticate, which makes our code quite simple. For example, if you want to use your Azure CLI login session, you can use the following code:

```python
from azure.identity import AzureCliCredential

credential = AzureCliCredential()
```

The code is similar for the other options. If you want to use a browser to authenticate, you can use the following code:

```python
from azure.identity import InteractiveBrowserCredential

credential = InteractiveBrowserCredential()
```

## Building the connection string

Now that we have our authentication mechanism in place, we can build the connection string. The connection string is a standard ODBC connection string and for Fabric, it looks like this:

```python
sql_endpoint = "" # copy and paste the SQL endpoint from any of the Lakehouses or Warehouses in your Fabric Workspace
database = "" # copy and paste the name of the Lakehouse or Warehouse you want to connect to

connection_string = f"Driver={{ODBC Driver 18 for SQL Server}};Server={sql_endpoint},1433;Database=f{database};Encrypt=Yes;TrustServerCertificate=No"
```

## Building the connection

To build the connection, we have to first use our credentials from [above](#authentication) to retrieve an access token we can pass to Fabric. This is all very technical, but you can follow along with the comments in the code below:

```python
import struct
from itertools import chain, repeat
import pyodbc


# prepare the access token

token_object = credential.get_token("https://database.windows.net//.default") # Retrieve an access token valid to connect to SQL databases
token_as_bytes = bytes(token.token, "UTF-8") # Convert the token to a UTF-8 byte string
encoded_bytes = bytes(chain.from_iterable(zip(value, repeat(0)))) # Encode the bytes to a Windows byte string
token_bytes = struct.pack("<i", len(encoded_bytes)) + encoded_bytes # Package the token into a bytes object
attrs_before = {1256: token_bytes}  # Attribute pointing to SQL_COPT_SS_ACCESS_TOKEN to pass access token to the driver


# build the connection

connection = pyodbc.connect(connection_string, attrs_before=attrs_before)
```

Now we can use the connection to run SQL queries:

```python
cursor = connection.cursor()
cursor.execute("SELECT * FROM sys.tables")
rows = cursor.fetchall()
print(rows) # this will print all the tables available in the lakehouse or warehouse
```

Make sure to close the cursor and the connection when you're done:

```python
cursor.close()
connection.close()
```

The whole API to query databases from Python is documented in [PEP 249](https://peps.python.org/pep-0249/). More `pyodbc` documentation is available in [the project's wiki](https://github.com/mkleehammer/pyodbc/wiki).

## Putting it all together

Below you can find the complete example, merging all the steps from above:

```python
import struct
from itertools import chain, repeat

import pyodbc
from azure.identity import AzureCliCredential

credential = AzureCliCredential() # use your authentication mechanism of choice
sql_endpoint = "" # copy and paste the SQL endpoint from any of the Lakehouses or Warehouses in your Fabric Workspace
database = "" # copy and paste the name of the Lakehouse or Warehouse you want to connect to

connection_string = f"Driver={{ODBC Driver 18 for SQL Server}};Server={sql_endpoint},1433;Database=f{database};Encrypt=Yes;TrustServerCertificate=No"

token_object = credential.get_token("https://database.windows.net//.default") # Retrieve an access token valid to connect to SQL databases
token_as_bytes = bytes(token.token, "UTF-8") # Convert the token to a UTF-8 byte string
encoded_bytes = bytes(chain.from_iterable(zip(value, repeat(0)))) # Encode the bytes to a Windows byte string
token_bytes = struct.pack("<i", len(encoded_bytes)) + encoded_bytes # Package the token into a bytes object
attrs_before = {1256: token_bytes}  # Attribute pointing to SQL_COPT_SS_ACCESS_TOKEN to pass access token to the driver

connection = pyodbc.connect(connection_string, attrs_before=attrs_before)
cursor = connection.cursor()
cursor.execute("SELECT * FROM sys.tables")
rows = cursor.fetchall()
print(rows)

cursor.close()
connection.close()
```
