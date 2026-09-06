---
description: "SQLite data driver deployment, connection configuration and ad hoc validation."
icon: database
---

# SQLite

Access the in-process database through Microsoft.Data.Sqlite. Application deployment needs to retain the native runtime library, and the file library also needs correct directory and file permissions. The application uses it through [Data Access Interfaces](../data-access.md) or [Data Services](../services.md), and the public mapping, conditions and data schema still follow the organization of the data engine.

| Item | value |
| --- | --- |
| source code directory | `Zongsoft.Data/drivers/sqlite` |
| NuGet package | `Zongsoft.Data.SQLite` |
| Connection and Script Driver Key | `SQLite` |

## Deployment and Connection

Append the following snippet to the existing host's deployment manifest. If Data has been deployed, there is no need to declare the same package again; the driver's manifest, assembly and runtime dependencies should be delivered together, see [Deployment Tool](../../../tools/deployer.md).

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft data]
nuget:Zongsoft.Data

[plugins zongsoft data sqlite]
nuget:Zongsoft.Data.SQLite
```
{% endcode %}

## Traceable Integration Example

Discussions There is currently no database script for this driver. The integration example uses the test project of the framework driver. Test models, mapping and initialization scripts should be used as a set, and Discussions table building scripts in other dialects cannot be directly considered compatible.

Source: [framework/Zongsoft.Data/drivers/sqlite/test/DatabaseFixture.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/sqlite/test/DatabaseFixture.cs#L13) (excerpt; see source for context).

{% code title="DatabaseFixture.cs" %}
```csharp
private static readonly string DATABASE_FILE = Path.Combine(AppContext.BaseDirectory, "test.db");
private static readonly string CONNECTION_STRING = $"DataSource={DATABASE_FILE};PRAGMA:optimize;PRAGMA:journal_mode=WAL;PRAGMA:synchronous=NORMAL;PRAGMA:temp_store=MEMORY;";
```
{% endcode %}

Source: [framework/Zongsoft.Data/drivers/sqlite/test/DatabaseFixture.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/sqlite/test/DatabaseFixture.cs#L35) (excerpt; see source for context).

{% code title="DatabaseFixture.cs" %}
```csharp
this.ConnectionSettings = Configuration.SQLiteConnectionSettingsDriver.Instance.GetSettings(CONNECTION_STRING);
this.Accessor = DataAccessProvider.Instance.GetAccessor("Test", new DataAccessOptions([this.ConnectionSettings]));
```
{% endcode %}

Here Test is the test accessor name, CONNECTION_STRING and DATABASE_FILE are from the same fixture. This fixture deletes the test files and executes the init script, which can only be run in the test output directory. SQLite's WAL parameters and DuckDB's file paths are both part of existing tests, which does not mean that Discussions has been adapted to both databases.

The plugin host's connection items are still placed in /Data/ConnectionSettings, and the driver keys use the canonical names in the index of this page. The accessor name obtained by the business must correspond to the deployment configuration; Discussions obtains the accessor by module name, and the test fixture explicitly passes in the connection settings in the code. The difference between the two entrances is shown in [Connections and data sources](../connections.md). The named SQL is still determined by its script's driver, and switching connections will not translate handwritten SQL.

## What Should You Pay Attention to with This Driver?

{% hint style="info" %}
💡 The relative database path is affected by the operating environment, so you should confirm the location of the final opened file. Memory libraries with different connections may not necessarily share data; creating a database file does not mean that the business table has been created.
{% endhint %}

## How to Verify After Integration

You can first complete the first query without a table, and then prepare the business table in an independent file library. Then verify the results of foreign keys, concurrent writes, lock waits, and reopening after closing. When checking backup and replacement files, you must consider log files and connections that are still occupied by the database.

Validation should cover a query, a set of actual written data, failure recovery, and then scale to production workloads. Common constraints can be checked against [Driver selection and acceptance](../drivers.md), [Connection Configuration](../connections.md), and [Transactions and Consistency](../transactions.md).

## Project Resources

[source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/sqlite) · [Plugin Manifest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/sqlite/src/Zongsoft.Data.SQLite.plugin) · [Chinese project description](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/sqlite/README.zh-Hans.md)
