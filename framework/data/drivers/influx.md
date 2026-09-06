---
description: "InfluxDB data driver deployment, connection configuration and ad hoc verification."
icon: database
---

# InfluxDB

Currently, InfluxDB3.Client integration InfluxDB 3 is used to provide time series data connection and dialect adaptation. The application uses it through [Data Access Interfaces](../data-access.md) or [Data Services](../services.md), and the public mapping, conditions and data schema still follow the organization of the data engine.

| Item | value |
| --- | --- |
| source code directory | `Zongsoft.Data/drivers/influx` |
| NuGet package | `Zongsoft.Data.Influx` |
| Connection and Script Driver Key | `Influx` |

## Deployment and Connection

Append the following snippet to the existing host's deployment manifest. If Data has been deployed, there is no need to declare the same package again; the driver's manifest, assembly and runtime dependencies should be delivered together, see [Deployment Tool](../../../tools/deployer.md).

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft data]
nuget:Zongsoft.Data

[plugins zongsoft data influx]
nuget:Zongsoft.Data.Influx
```
{% endcode %}

## Traceable Integration Example

Discussions There is currently no database script for this driver. The integration example uses the test project of the framework driver. Test models, mapping and initialization scripts should be used as a set, and Discussions table building scripts in other dialects cannot be directly considered compatible.

Source: [framework/Zongsoft.Data/drivers/influx/test/DatabaseFixture.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/influx/test/DatabaseFixture.cs#L23) (excerpt; see source for context).

{% code title="DatabaseFixture.cs" %}
```csharp
this.ConnectionSettings = Configuration.InfluxConnectionSettingsDriver.Instance.GetSettings(CONNECTION_STRING);
this.Accessor = DataAccessProvider.Instance.GetAccessor("Test", new DataAccessOptions([this.ConnectionSettings]));
```
{% endcode %}

CONNECTION_STRING comes from the test environment; only settings parsing and accessor creation are excerpted here, and the test server address and credentials are not copied. Check the same directory fixtures, mappings and initialization conditions before running the test. The following offline test shows that extended settings are retained in Properties. Retaining text does not mean that the underlying driver must use these settings.

Source: [framework/Zongsoft.Data/drivers/influx/test/ConnectionSettingsPropertiesTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/influx/test/ConnectionSettingsPropertiesTest.cs#L8) (excerpt; see source for context).

{% code title="ConnectionSettingsPropertiesTest.cs" %}
```csharp
public void UnknownPropertiesArePreserved()
{
	var settings = Configuration.InfluxConnectionSettingsDriver.Instance.GetSettings(
		"CircuitBreaker.Duration=00:01:00;CircuitBreaker.MaximumDuration=00:02:00");

	Assert.Equal("00:01:00", settings.Properties["CircuitBreaker.Duration"]);
	Assert.Equal("00:02:00", settings.Properties["CircuitBreaker.MaximumDuration"]);
}
```
{% endcode %}

The plugin host's connection items are still placed in /Data/ConnectionSettings, and the driver keys use the canonical names in the index of this page. The accessor name obtained by the business must correspond to the deployment configuration; Discussions obtains the accessor by module name, and the test fixture explicitly passes in the connection settings in the code. The difference between the two entrances is shown in [Connections and data sources](../connections.md). The named SQL is still determined by its script's driver, and switching connections will not translate handwritten SQL.

## What Should You Pay Attention to with This Driver?

{% hint style="info" %}
💡 The current driver declares TransactionSuppressed and cannot be a participant in the local transaction of the relational database. The connection options and query language of older versions of InfluxDB also cannot be directly rolled into the current implementation.
{% endhint %}

## How to Verify After Integration

First check the server version, database and token permissions, and then verify the time accuracy, labels and field types. Failure recovery from multiple writes should be designed by the application; the unified transaction entry will not complete atomic rollback for this driver.

Validation should cover a query, a set of actual written data, failure recovery, and then scale to production workloads. Common constraints can be checked against [Driver selection and acceptance](../drivers.md), [Connection Configuration](../connections.md), and [Transactions and Consistency](../transactions.md).

## Project Resources

[source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/influx) · [Plugin Manifest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/influx/src/Zongsoft.Data.Influx.plugin) · [Chinese project description](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/influx/README.zh-Hans.md)
