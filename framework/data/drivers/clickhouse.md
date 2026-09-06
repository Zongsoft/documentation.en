---
description: "ClickHouse data driver deployment, connection configuration and ad hoc verification."
icon: database
---

# ClickHouse

Access ClickHouse through ClickHouse.Client, connect to the columnar analysis database and adapt the expressions and import operations of the data engine. The application uses it through [Data Access Interfaces](../data-access.md) or [Data Services](../services.md), and the public mapping, conditions and data schema still follow the organization of the data engine.

| Item | value |
| --- | --- |
| source code directory | `Zongsoft.Data/drivers/clickhouse` |
| NuGet package | `Zongsoft.Data.ClickHouse` |
| Connection and Script Driver Key | `ClickHouse` |

## Deployment and Connection

Append the following snippet to the existing host's deployment manifest. If Data has been deployed, there is no need to declare the same package again; the driver's manifest, assembly and runtime dependencies should be delivered together, see [Deployment Tool](../../../tools/deployer.md).

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft data]
nuget:Zongsoft.Data

[plugins zongsoft data clickhouse]
nuget:Zongsoft.Data.ClickHouse
```
{% endcode %}

## Traceable Integration Example

Discussions provides the corresponding [database script](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/database/Zongsoft.Discussions-clickhouse.sql). Follow the real Feedback, Forum, Thread, Post and other models and [Discussions.mapping](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.mapping). For query and writing, see [Data Services](../services.md). The existence of the script does not mean that end-to-end verification has been completed for all tables and all driver combinations; in particular, model members, column lengths, default values, relationships, and tables used by the current business must be checked.

Source: [framework/Zongsoft.Data/drivers/clickhouse/test/DatabaseFixture.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/clickhouse/test/DatabaseFixture.cs#L23) (excerpt; see source for context).

{% code title="DatabaseFixture.cs" %}
```csharp
this.ConnectionSettings = Configuration.ClickHouseConnectionSettingsDriver.Instance.GetSettings(CONNECTION_STRING);
this.Accessor = DataAccessProvider.Instance.GetAccessor("Test", new DataAccessOptions([this.ConnectionSettings]));
```
{% endcode %}

CONNECTION_STRING comes from the test environment; only settings parsing and accessor creation are excerpted here, and the test server address and credentials are not copied. Check the same directory fixtures, mappings and initialization conditions before running the test. The following offline test shows that extended settings are retained in Properties. Retaining text does not mean that the underlying driver must use these settings.

Source: [framework/Zongsoft.Data/drivers/clickhouse/test/ConnectionSettingsPropertiesTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/clickhouse/test/ConnectionSettingsPropertiesTest.cs#L8) (excerpt; see source for context).

{% code title="ConnectionSettingsPropertiesTest.cs" %}
```csharp
public void UnknownPropertiesArePreserved()
{
	var settings = Configuration.ClickHouseConnectionSettingsDriver.Instance.GetSettings(
		"CircuitBreaker.Duration=00:01:00;CircuitBreaker.MaximumDuration=00:02:00");

	Assert.Equal("00:01:00", settings.Properties["CircuitBreaker.Duration"]);
	Assert.Equal("00:02:00", settings.Properties["CircuitBreaker.MaximumDuration"]);
}
```
{% endcode %}

The plugin host's connection items are still placed in /Data/ConnectionSettings, and the driver keys use the canonical names in the index of this page. The accessor name obtained by the business must correspond to the deployment configuration; Discussions obtains the accessor by module name, and the test fixture explicitly passes in the connection settings in the code. The difference between the two entrances is shown in [Connections and data sources](../connections.md). The named SQL is still determined by its script's driver, and switching connections will not translate handwritten SQL.

## What Should You Pay Attention to with This Driver?

{% hint style="info" %}
💡 The example uses HTTP connection port, which should correspond to the protocol enabled on the server. Updates, deletions, and transaction expectations of analytical storage need to be confirmed separately, and all assumptions of traditional transactional data services cannot be directly migrated.
{% endhint %}

## How to Verify After Integration

Verify aggregation, pagination, time and numeric types with real query shapes, and then validate batch writes and subsequent visibility. For models that require frequent item-by-item updates, you should first check the target table engine, driver implementation, and server behavior.

Validation should cover a query, a set of actual written data, failure recovery, and then scale to production workloads. Common constraints can be checked against [Driver selection and acceptance](../drivers.md), [Connection Configuration](../connections.md), and [Transactions and Consistency](../transactions.md).

## Project Resources

[source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/clickhouse) · [Plugin Manifest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/clickhouse/src/Zongsoft.Data.ClickHouse.plugin) · [Chinese project description](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/clickhouse/README.zh-Hans.md)
