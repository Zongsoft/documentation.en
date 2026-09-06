---
description: "TDengine data driver deployment, connection configuration and ad hoc verification."
icon: database
---

# TDengine

Time series database integration through TDengine.Connector. Supertables, tags, and time precision are modeling constraints that should be designed together with device data and sampling methods. The application uses it through [Data Access Interfaces](../data-access.md) or [Data Services](../services.md), and the public mapping, conditions and data schema still follow the organization of the data engine.

| Item | value |
| --- | --- |
| source code directory | `Zongsoft.Data/drivers/tdengine` |
| NuGet package | `Zongsoft.Data.TDengine` |
| Connection and Script Driver Key | `TDengine` |

## Deployment and Connection

Append the following snippet to the existing host's deployment manifest. If Data has been deployed, there is no need to declare the same package again; the driver's manifest, assembly and runtime dependencies should be delivered together, see [Deployment Tool](../../../tools/deployer.md).

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft data]
nuget:Zongsoft.Data

[plugins zongsoft data tdengine]
nuget:Zongsoft.Data.TDengine
```
{% endcode %}

## Traceable Integration Example

Discussions There is currently no database script for this driver. The integration example uses the test project of the framework driver. Test models, mapping and initialization scripts should be used as a set, and Discussions table building scripts in other dialects cannot be directly considered compatible.

Source: [framework/Zongsoft.Data/drivers/tdengine/test/DatabaseFixture.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/tdengine/test/DatabaseFixture.cs#L23) (excerpt; see source for context).

{% code title="DatabaseFixture.cs" %}
```csharp
this.ConnectionSettings = Configuration.TDengineConnectionSettingsDriver.Instance.GetSettings(CONNECTION_STRING);
this.ConnectionSettings.Protocol = Configuration.TDengineConnectionProtocol.Native;
this.Accessor = DataAccessProvider.Instance.GetAccessor("Zongsoft.Data.TDengine.Tests", new DataAccessOptions([this.ConnectionSettings]));
```
{% endcode %}

CONNECTION_STRING comes from the test environment; only settings parsing and accessor creation are excerpted here, and the test server address and credentials are not copied. Check the same directory fixtures, mappings and initialization conditions before running the test. The following offline test shows that extended settings are retained in Properties. Retaining text does not mean that the underlying driver must use these settings.

Source: [framework/Zongsoft.Data/drivers/tdengine/test/ConnectionSettingsPropertiesTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/tdengine/test/ConnectionSettingsPropertiesTest.cs#L8) (excerpt; see source for context).

{% code title="ConnectionSettingsPropertiesTest.cs" %}
```csharp
public void UnknownPropertiesArePreserved()
{
	var settings = Configuration.TDengineConnectionSettingsDriver.Instance.GetSettings(
		"CircuitBreaker.Duration=00:01:00;CircuitBreaker.MaximumDuration=00:02:00");

	Assert.Equal("00:01:00", settings.Properties["CircuitBreaker.Duration"]);
	Assert.Equal("00:02:00", settings.Properties["CircuitBreaker.MaximumDuration"]);
}
```
{% endcode %}

The plugin host's connection items are still placed in /Data/ConnectionSettings, and the driver keys use the canonical names in the index of this page. The accessor name obtained by the business must correspond to the deployment configuration; Discussions obtains the accessor by module name, and the test fixture explicitly passes in the connection settings in the code. The difference between the two entrances is shown in [Connections and data sources](../connections.md). The named SQL is still determined by its script's driver, and switching connections will not translate handwritten SQL.

## What Should You Pay Attention to with This Driver?

{% hint style="info" %}
💡 In this example, the Native protocol is selected. When switching to WebSocket, you should also check the settings supported by the connector, the server adaptation service and the port, and cannot just replace the address. Native client and server versions also need to match.
{% endhint %}

## How to Verify After Integration

Verify endpoints and client dependencies first, then check time columns, supertables, subtables, and tags against the database structure. Verify business expectations using repeated writes at the same sampling moment, different time precision, and missing data.

Validation should cover a query, a set of actual written data, failure recovery, and then scale to production workloads. Common constraints can be checked against [Driver selection and acceptance](../drivers.md), [Connection Configuration](../connections.md), and [Transactions and Consistency](../transactions.md).

## Project Resources

[source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/tdengine) · [Plugin Manifest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/tdengine/src/Zongsoft.Data.TDengine.plugin) · [Chinese project description](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/tdengine/README.zh-Hans.md)
