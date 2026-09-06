---
description: "SQL Server data driver deployment, connection configuration and ad hoc validation."
icon: database
---

# SQL Server

Connected to SQL Server via Microsoft.Data.SqlClient and was responsible for SQL Server's identifiers, parameters, paging and writing dialect. The application uses it through [Data Access Interfaces](../data-access.md) or [Data Services](../services.md), and the public mapping, conditions and data schema still follow the organization of the data engine.

| Item | value |
| --- | --- |
| source code directory | `Zongsoft.Data/drivers/mssql` |
| NuGet package | `Zongsoft.Data.MsSql` |
| Connection and Script Driver Key | `MsSql` |

## Deployment and Connection

Append the following snippet to the existing host's deployment manifest. If Data has been deployed, there is no need to declare the same package again; the driver's manifest, assembly and runtime dependencies should be delivered together, see [Deployment Tool](../../../tools/deployer.md).

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft data]
nuget:Zongsoft.Data

[plugins zongsoft data mssql]
nuget:Zongsoft.Data.MsSql
```
{% endcode %}

## Traceable Integration Example

Discussions provides the corresponding [database script](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/database/Zongsoft.Discussions-mssql.sql). Follow the real Feedback, Forum, Thread, Post and other models and [Discussions.mapping](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.mapping). For query and writing, see [Data Services](../services.md). The existence of the script does not mean that end-to-end verification has been completed for all tables and all driver combinations; in particular, model members, column lengths, default values, relationships, and tables used by the current business must be checked.

Source: [framework/Zongsoft.Data/drivers/mssql/test/ConnectionSettingsTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/mssql/test/ConnectionSettingsTest.cs#L10) (excerpt; see source for context).

{% code title="ConnectionSettingsTest.cs" %}
```csharp
public void MaximumPoolSizeMatchesProviderDefault()
{
	var settings = Configuration.MsSqlConnectionSettingsDriver.Instance.GetSettings("Server=localhost");
	var builder = new SqlConnectionStringBuilder();

	Assert.Equal((uint)builder.MaxPoolSize, settings.MaximumPoolSize);
}
```
{% endcode %}

This offline test only verifies that the connection settings are consistent with the provider's default connection pool limit; Server=localhost does not establish a database connection here. A complete connection also requires database name, identity credentials, and operating environment related settings.

The plugin host's connection items are still placed in /Data/ConnectionSettings, and the driver keys use the canonical names in the index of this page. The accessor name obtained by the business must correspond to the deployment configuration; Discussions obtains the accessor by module name, and the test fixture explicitly passes in the connection settings in the code. The difference between the two entrances is shown in [Connections and data sources](../connections.md). The named SQL is still determined by its script's driver, and switching connections will not translate handwritten SQL.

## What Should You Pay Attention to with This Driver?

{% hint style="info" %}
💡 This example uses integrated authentication, and the database sees the identity of the actual running host. The interactive terminal, Windows service and Web application pool may use different accounts; successful local debugging does not mean that the service account also has database permissions.
{% endhint %}

## How to Verify After Integration

First authenticate the connection with the running authentication of the deployment environment, and then check the object schema, table permissions, auto-increment ID return and transaction isolation. Connection encryption and certificate verification are determined by the client configuration and the server. They should be configured according to the deployment environment. Certificate issues cannot be covered up by turning off verification.

Validation should cover a query, a set of actual written data, failure recovery, and then scale to production workloads. Common constraints can be checked against [Driver selection and acceptance](../drivers.md), [Connection Configuration](../connections.md), and [Transactions and Consistency](../transactions.md).

## Project Resources

[source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/mssql) · [Plugin Manifest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/mssql/src/Zongsoft.Data.MsSql.plugin) · [Chinese project description](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/mssql/README.zh-Hans.md)
