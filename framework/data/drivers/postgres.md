---
description: "PostgreSQL data driver deployment, connection configuration and ad hoc verification."
icon: database
---

# PostgreSQL

Connect to PostgreSQL via Npgsql, providing dialect adaptation of identifiers, parameters, RETURNING, and query expressions. The application uses it through [Data Access Interfaces](../data-access.md) or [Data Services](../services.md), and the public mapping, conditions and data schema still follow the organization of the data engine.

| Item | value |
| --- | --- |
| source code directory | `Zongsoft.Data/drivers/postgres` |
| NuGet package | `Zongsoft.Data.PostgreSql` |
| Connection and Script Driver Key | `PostgreSQL` |

## Deployment and Connection

Append the following snippet to the existing host's deployment manifest. If Data has been deployed, there is no need to declare the same package again; the driver's manifest, assembly and runtime dependencies should be delivered together, see [Deployment Tool](../../../tools/deployer.md).

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft data]
nuget:Zongsoft.Data

[plugins zongsoft data postgres]
nuget:Zongsoft.Data.PostgreSql
```
{% endcode %}

## Traceable Integration Example

Discussions provides the corresponding [database script](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/database/Zongsoft.Discussions-postgres.sql). Follow the real Feedback, Forum, Thread, Post and other models and [Discussions.mapping](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.mapping). For query and writing, see [Data Services](../services.md). The existence of the script does not mean that end-to-end verification has been completed for all tables and all driver combinations; in particular, model members, column lengths, default values, relationships, and tables used by the current business must be checked.

Source: [framework/Zongsoft.Data/drivers/postgres/test/ConnectionSettingsTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/postgres/test/ConnectionSettingsTest.cs#L10) (excerpt; see source for context).

{% code title="ConnectionSettingsTest.cs" %}
```csharp
public void MaximumPoolSizeMatchesProviderDefault()
{
	var settings = Configuration.PostgreSqlConnectionSettingsDriver.Instance.GetSettings("Server=localhost");
	var builder = new NpgsqlConnectionStringBuilder();

	Assert.Equal((uint)builder.MaxPoolSize, settings.MaximumPoolSize);
}
```
{% endcode %}

This offline test only verifies that the connection settings are consistent with the provider's default connection pool limit; Server=localhost does not establish a database connection here. A complete connection also requires database name, identity credentials, and operating environment related settings.

The plugin host's connection items are still placed in /Data/ConnectionSettings, and the driver keys use the canonical names in the index of this page. The accessor name obtained by the business must correspond to the deployment configuration; Discussions obtains the accessor by module name, and the test fixture explicitly passes in the connection settings in the code. The difference between the two entrances is shown in [Connections and data sources](../connections.md). The named SQL is still determined by its script's driver, and switching connections will not translate handwritten SQL.

## What Should You Pay Attention to with This Driver?

{% hint style="info" %}
💡 The source code directory is named postgres, the package name is Zongsoft.Data.PostgreSql, and the driver key registered by the plugin is PostgreSQL. Configuration and mapping scripts should use the driver key and not infer it from the directory name.
{% endhint %}

## How to Verify After Integration

Check the database schema and search paths, as well as the case of mapping identifiers. Arrays, custom types, extensions, and time values need to be validated using actual models; deploying this driver does not automatically install server extensions or migrate database structures.

Validation should cover a query, a set of actual written data, failure recovery, and then scale to production workloads. Common constraints can be checked against [Driver selection and acceptance](../drivers.md), [Connection Configuration](../connections.md), and [Transactions and Consistency](../transactions.md).

## Project Resources

[source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/postgres) · [Plugin Manifest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/postgres/src/Zongsoft.Data.PostgreSql.plugin) · [Chinese project description](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/postgres/README.zh-Hans.md)
