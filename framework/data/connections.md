---
description: "Configure data connections, drivers and read/write separation."
icon: plug
---

# Connection Configuration

The data connection configuration item name matches the accessor name (see [Data Access Interfaces](data-access.md) for how to obtain it). A `DataAccess` can have multiple data sources for read/write separation and other scenarios.

## Configuration Location

The connection configuration is located under the `/Data/ConnectionSettings` options path. `DataAccessProvider` When getting the accessor, the collection is read from the current application configuration:

```text
/Data/ConnectionSettings
```

If no name is specified when calling `GetService(null)`, the default connection will be used; if the specified name does not exist, it will fallback to the default connection. A data configuration exception will be thrown when there is no default connection and the specified name does not exist.

<details>

<summary>How does the accessor name match the connection configuration?</summary>

Discussions The accessor name passed in comes from Module.NAME. When no name is passed in or the specified name does not exist, the default connection will be used according to the provider rules; therefore, it cannot be inferred which database is actually connected to just based on the module name in the business code. Deployments that require strong isolation should check for final matches to avoid unexpected fallbacks.

</details>

## Single Data Source

Module.Accessor of Discussions gets the accessor by module name:

Source: [src/Module.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Module.cs#L52) (excerpt; see source for context).

{% code title="Module.cs" %}
```csharp
public IDataAccess Accessor => _accessor ??= this.Services.ResolveRequired<IDataAccessProvider>().GetAccessor(this.Name);
```
{% endcode %}

The following db1 configuration is from the framework's XML parsing test, which explains the option structure; it is not a ready-made database connection for Discussions. When deploying Discussions, you should configure the corresponding accessor name or clear the default connection, and check the driver with the actual database environment.

Source: [framework/Zongsoft.Core/test/Configuration/Xml/OptionConfigurationTest-1.option](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/Xml/OptionConfigurationTest-1.option#L30) (excerpt; see source for context).

{% code title="OptionConfigurationTest-1.option" %}
```xml
<option path="/Data">
	<connectionSettings default="db1">
		<connectionSetting connectionSetting.name="db1" driver="mysql" mode="all" value="server=localhost" />
	</connectionSettings>
</option>
```
{% endcode %}

`connectionSetting.name` should be consistent with the data access name. Business modules usually use the module name as the data access name, such as `Security`, `Administratives` or `Discussions`.

## Connection String Properties

`value` is the connection string passed to the corresponding driver, usually consisting of key-value items separated by semicolons. The connection settings object maps these key-value items to driver-defined properties and converts them by property type, such as port, boolean, interval, network endpoint, or collection.

Source: [framework/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs#L16) (excerpt; see source for context).

{% code title="ConnectionSettingsTest.cs" %}
```csharp
private static readonly DateTime DATE = new(1979, 5, 15);
private static readonly string ConnectionString = $" ;; server=192.168.0.1:8080, localhost:8088  ; Integer=100 ; enabled ; double= 1.23; ;  boolean= true ; text= MyString; dateTime={DATE:yyyy-M-d}; ; mapping=s1:t1,s2 = t2, same ";
```
{% endcode %}

The collection attribute can be written as a text value. When placed in the connection string `value`, commas or vertical bars are usually used to separate the elements, because the semicolon has been used as a delimiter by the outer connection item; how the specific elements are converted is determined by the attribute type or the converter declared on the attribute. If the driver declares an element converter for a collection attribute, it can parse short formats such as `mapping=s1:t1,s2=t2,same` into structured entries.

The above DATE and ConnectionString come from ConnectionSettingsTest, and its MyDriver is a test driver and cannot be used to connect to MySQL. This input is used to check for whitespace, Boolean values, endpoint collections, and element transformations.

Driver settings objects can also assemble multiple flat keys into a composite property. The following skeleton test does not require the full cluster value to be present in the connection string, but instead populates the child members of the `Cluster` attribute with the `cluster.` prefix:

Source: [framework/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs#L178) (excerpt; see source for context).

{% code title="ConnectionSettingsTest.cs" %}
```csharp
public void TestConnectionSettingsCompositeProperty()
{
	var settings = MyDriver.Instance.GetSettings("a.b.c=none;cluster.address=192.168.0.100;nothing.property=none;cluster.heartbeat=30s");
	Assert.NotNull(settings);

	Assert.False(settings.Cluster.IsEmpty);
	Assert.Equal("192.168.0.100", settings.Cluster.Address);
	Assert.Equal(TimeSpan.FromSeconds(30), settings.Cluster.Heartbeat);
	Assert.Equal("none", settings.Properties["a.b.c"]);
	Assert.Equal("none", settings.Properties["nothing.property"]);
}
```
{% endcode %}

This type of writing is suitable for describing structured settings such as clusters, certificates, proxies, and retry strategies. Whether it can be used depends on whether the driver's connection setting class defines the corresponding attribute, and whether the attribute type can be automatically created and written to the public member.

## Read/write Separation

The connection name uses `#` appended with the data source identifier. The underlying connection with the same name as the accessor should be preserved because the default accessor factory checks the exact connection name first; configuring only suffix entries may fallback or fail when retrieving the accessor:

Discussions currently do not have a complete use case for configuring read-write replicas. Deployers can set the Mode through the data source under the same accessor name; see [DataSourceProvider](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/src/Common/DataSourceProvider.cs) and [DataSourceSelector](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/src/Common/DataSourceSelector.cs) for the actual filtering logic. Database replication, failover, and read-after-write consistency are still guaranteed by infrastructure and business agreements.

The data source of `mode="WriteOnly"` is used for writing, and the data source of `mode="ReadOnly"` is used for reading. Drivers and data source providers select the appropriate data source based on the type of operation.

## Driver Name

The `driver` attribute must match the name of the deployed driver registration. For example, the MySQL driver plugin will register:

- `/Workbench/Configuration/ConnectionSettings/Drivers/MySql`
- `/Workbench/Data/Drivers/MySql`

If the corresponding driver is not deployed in the plugin directory, the connection string cannot be parsed and executed even if it is correct.

{% content-ref url="drivers.md" %}
[drivers.md](drivers.md)
{% endcontent-ref %}

## Things to Note

The connection string is interpreted by the corresponding database driver. When configuring, you should also confirm that the driver package has been deployed to the host master plugin directory.

In the production environment, it is recommended to hand over sensitive information to the environment configuration, key system or deployment platform for injection, and avoid submitting account passwords to the source code repository.

<details>

<summary>Can I put the real connection string in the configuration file?</summary>

The development environment can use the local test connection string, but in the production environment it is not recommended to write the account, password, and access key into the repository. It is more prudent to inject sensitive values by the deployment platform, environment variables, key management services, or site-level configuration.

</details>

## Routing and Consistency

The data source provider selects the exact name and `name#suffix` items, and then the action selects the data source from a readable or writable collection. Queries, presence checks, and aggregations go from readable sources; named commands are routed based on mutability in the map. Read-only commands should explicitly write `mutability="none"`, see [Mapping Files](mapping.md) for details.

Read/write separation relies on the database's own replication and consistency strategies. Reading immediately after writing may be affected by replication delays; when you need to read this write, you should handle it according to the business transaction and data source scheme. You cannot think that configuring multiple connections will automatically guarantee strong consistency.

## Connection Failure Protection

The current engine manages physical connection establishment via a DataConnector. Connection establishment for the same data source is serially protected, and subsequent connections can be quickly rejected within a period of time after failure to avoid a large number of requests hitting an unavailable database at the same time.

The first physical connection failure usually retains the original exception from the database provider; DataConnectionException is quickly rejected during circuit breaking, which can read RetryAt, RetryAfter, etc. Retry wait times should still be considered after restoring the database, and do not treat each fast rejection as a new network failure.

Achieve positioning: [Connection name filter](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/src/Common/DataSourceProvider.cs), [Delimiters and access modes](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/src/Common/DataSource.cs), [accessor name](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Data/DataAccessProviderBase.cs), [connection protection](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/src/Common/DataConnector.cs).
