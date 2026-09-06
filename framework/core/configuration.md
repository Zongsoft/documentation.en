---
description: "Start by reading Discussions options and understand configuration sources, object bindings, connection settings, Profiles, Models, and Options."
icon: sliders
---

# Zongsoft.Configuration

Zongsoft.Configuration adds plugin options, object binding, connection settings and Profile support on top of the standard .NET configuration system. The configuration first forms a key-value tree from different sources, and then is bound to the type required by the business; reading the configuration and executing the actions described by the configuration are two stages. If the connection string is parsed successfully, it does not mean that the database has been connected; the storage path exists in the options, nor does it mean that the corresponding file system plugin has been deployed.

The standard configuration entry is [Microsoft.Extensions.Configuration.IConfiguration](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.iconfiguration) _[Source](https://source.dot.net/#Microsoft.Extensions.Configuration.Abstractions/IConfiguration.cs)_. Discussions.Configuration.IConfiguration is the option model declared by the forum itself, and the two have different uses.

## Design Positioning

| part | Problem solved | Typical sources |
| --- | --- | --- |
| Configuration source | Where do configuration values come from and how to overwrite and update them? | Host configuration, environment parameters, plugin option files |
| object binding | How strings and nodes become properties, collections and models | ConfigurationBinder |
| Connection settings | How to combine name, driver and connection parameters | Data-driven and external services |
| XML options | How to map XML structure to configuration tree | *.option |
| Profile | Hierarchical INI entries and imports | *.deploy |
| Models | How model records become configuration sources | Framework model configuration testing |
| Options | How to bind options to integration dependency injection and change monitoring | security services and scheduling services |

Multiple configuration sources can be superimposed on the same application. When judging the final value, you need to check the coverage relationship along the host loading sequence, rather than just looking at a certain file. See [Options file reference](../../references/option-files.md) for file selection, environment and site combinations.

## Actual Reading of Discussions

Source: [src/Zongsoft.Discussions.option](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.option#L3) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.option" %}
```xml
<options>
	<option path="/Discussions">
		<general siteId="1" basePath="zfs.s3:/zongsoft-discussions/" />
	</option>
</options>
```
{% endcode %}

General Saves the default site number and file base path. The zfs.s3 here is the file system scheme used by Discussions’ current options, which depends on the corresponding storage extension and environment configuration; local debugging needs to be replaced with a storage location accessible by itself.

Utility.GetFilePath reads the path value directly:

Source: [src/Utility.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Utility.cs#L283) (excerpt; see source for context).

{% code title="Utility.cs" %}
```csharp
var basePath = ApplicationContext.Current.Configuration.GetOptionValue<string>("/Discussions/General.BasePath");

if(string.IsNullOrWhiteSpace(basePath))
	return string.Empty;
```
{% endcode %}

/Discussions locates module configuration, General.BasePath corresponds to the basePath attribute on the general element. When the value is empty, the path calculation returns an empty string, and this result cannot be regarded as a writable location. For the subsequent stratification process by site and user, see [File and content storage](io.md).

## Basic Binding

Reading a single value is suitable for a small number of independent settings; when there are many configuration items and they are used together, GetOption or Bind can bind a piece of configuration as a model. Zongsoft binder supports property aliases, collections, unrecognized property containers and dynamic models, and the specific conversion is still subject to the target type.

Source: [src/Configuration/IConfiguration.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Configuration/IConfiguration.cs#L35) (excerpt; see source for context).

{% code title="IConfiguration.cs" %}
```csharp
public interface IConfiguration
{
	/// <summary>获取或设置默认的站点编号。</summary>
	uint SiteId { get; set; }

	/// <summary>获取或设置文件存储的基路径。</summary>
	string BasePath { get; set; }
}
```
{% endcode %}

This interface gives the option shape that Discussions has defined. The actual path of the current Utility is a single value read, and the existence of the interface should not be interpreted as that all consumers have obtained the model through dependency injection.

QueueOptions in the framework test shows unrecognized properties and collection naming:

Source: [framework/Zongsoft.Core/test/Configuration/Xml/OptionConfigurationTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/Xml/OptionConfigurationTest.cs#L186) (excerpt; see source for context).

{% code title="OptionConfigurationTest.cs" %}
```csharp
[Configuration(nameof(Properties))]
public class QueueOptions
{
	public string Name { get; set; }
	public SubscriptionOptions Subscription { get; set; }
	public IDictionary<string, string> Properties { get; set; }

	public class SubscriptionOptions
	{
		public Messaging.MessageReliability Reliability { get; set; }

		[ConfigurationProperty]
		public TopicOptionsCollection Topics { get; set; }
	}

	public class TopicOptions
	{
		public string Name { get; set; }

		[ConfigurationProperty("tag")]
		public IList<string> Tags { get; set; }

		public override string ToString() => this.Name;
	}

	public class TopicOptionsCollection() : KeyedCollection<string, TopicOptions>(StringComparer.OrdinalIgnoreCase)
	{
		protected override string GetKeyForItem(TopicOptions topic) => topic.Name;
	}
}

```
{% endcode %}

The QueueOptions above belong to the XML configuration test. Configuration specifies that Properties receive unrecognized items, and ConfigurationProperty provides binding conventions for nested collections and tag properties. When copying an attribute declaration, you also need the corresponding element type and collection key rules. You cannot just copy the attribute name.

BindNonPublicProperties of ConfigurationBinderOptions controls the binding of non-public members, and UnrecognizedError controls whether an error is reported for unrecognized configurations. Repeated binding to an existing collection may add elements; when overloading the configuration, you should clearly rebuild or update the object to avoid unintentional accumulation.

## Connection Settings

Connection settings separate the connection name, driver name, and parameter values. The connection name is used for application search, the driver name determines the parameter model, and the value is parsed by the model into types such as ports, timeouts, endpoints, and collections.

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

This is the db1 configuration in the framework XML test, using only localhost to verify the key-value binding. Discussions Request the accessor by module name, and the actual deployment also requires the corresponding name or default connection; see [Connections and data sources](../data/connections.md) for the complete process, and [Data-driven reading](../data/drivers.md) for driver differences.

### Composite Properties

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

MyDriver, ClusterSettings, and MyConnectionSettings are all fixtures in the same test file. cluster.address and cluster.heartbeat are bound as members of Cluster, other unrecognized keys remain in Properties. This syntax will produce structured objects only if the driver settings model defines the corresponding composite properties. See [Converters](components/converters.md) for specialized conversions of collection elements.

## XML Options File

Source: [framework/Zongsoft.Core/test/Configuration/Xml/OptionConfigurationTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/Xml/OptionConfigurationTest.cs#L14) (excerpt; see source for context).

{% code title="OptionConfigurationTest.cs" %}
```csharp
public static IConfigurationRoot GetConfiguration1()
{
	return new ConfigurationBuilder()
		.AddOptionFile("Configuration/Xml/OptionConfigurationTest-1.option")
		.Build();
}
```
{% endcode %}

The test loads the option file from its own output directory and then reads the configuration node. In the plugin host, this step is usually completed by the option loading process, without the need to create the configuration root again for each business service.

The path of option specifies the mount location; elements form child nodes, and attributes form configuration values. Duplicate elements require a stable key, such as connectionSetting.name; the configuration collection index cannot be inferred solely from the order in the document. See [Options file reference](../../references/option-files.md) for detailed rules and discussion list.

## Profile And.deploy

Profile preserves INI entries, sections, and comments, and supports hierarchical sections separated by spaces or tabs. Here are the framework’s existing Profile test inputs:

Source: [framework/Zongsoft.Core/test/Configuration/Profiles/Profile-1.ini](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/Profiles/Profile-1.ini#L1) (excerpt; see source for context).

{% code title="Profile-1.ini" %}
```ini
﻿[plugins zongsoft data]
nuget:Zongsoft.Data
```
{% endcode %}

It will get the plugins/zongsoft/data three-level section, and nuget:[Zongsoft.Data](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data) is an entry with no value under this section. Profile is only responsible for parsing this structure, NuGet location, replication and version selection are interpreted by the deployment tool.

Discussions' in-package manifest uses relative artifact paths and Framework variables:

Source: [src/Zongsoft.Discussions.deploy](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.deploy#L1) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.deploy" %}
```ini
artifacts/Zongsoft.Discussions.plugin
artifacts/Zongsoft.Discussions.option
artifacts/Zongsoft.Discussions.mapping
lib/$(Framework)/Zongsoft.Discussions.*
```
{% endcode %}

These paths are relative to the package's file layout. Framework variables are provided by the deployment context and are used to select the target assembly. The paragraph target directory, source path, rename, conditional expression, import and variable scope are all uniformly described by [Deployment manifest reference](../../references/deploy-files.md).

The built-in ImportDirective handles importing, and the directive appears as a comment. Relative paths, duplicate files, and circular references in import files should be checked against the current Profile.Load and deployment tool behavior; do not infer their semantics from the normal INI resolver.

## Models Configuration Source

Models provides a set of keyed model records as a configuration tree. Discussions does not have a complete model configuration persistence process, so ModelConfigurationTest is used here:

Source: [framework/Zongsoft.Core/test/Configuration/Models/ModelConfigurationTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/Models/ModelConfigurationTest.cs#L163) (excerpt; see source for context).

{% code title="ModelConfigurationTest.cs" %}
```csharp
public abstract class ConfigurationEntity
{
	public abstract int TenantId { get; set; }
	public abstract int BranchId { get; set; }
	public abstract string Module { get; set; }
	public abstract string Key { get; set; }
	public abstract string Value { get; set; }
}

public static class ConfigurationEntityExtension
{
	public static string GetInfo(this ConfigurationEntity entity)
	{
		if(entity == null)
			return string.Empty;

		return $"[{entity.TenantId}-{entity.BranchId}:{entity.Module}]" + Environment.NewLine + $"{entity.Key}={entity.Value}";
	}
}
```
{% endcode %}

Source: [framework/Zongsoft.Core/test/Configuration/Models/ModelConfigurationTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/Models/ModelConfigurationTest.cs#L71) (excerpt; see source for context).

{% code title="ModelConfigurationTest.cs" %}
```csharp
var models = GetModels(dictionary);

return new ConfigurationBuilder()
	.AddModels(models, source =>
	{
		source.OnChange(model => System.Diagnostics.Debug.WriteLine(model.GetInfo()));

		if(persistent != null)
			source.OnChange(persistent);
	})
	.Build();
```
{% endcode %}

GetModels constructs a ConfigurationEntity from an in-memory dictionary in the same test and populates the tenant, branch and module fields. OnChange receives an updated or newly created model; the example callback only records or checks changes, there is no real database transaction. When persistence is required, the caller must implement callbacks and assume write failures, permissions, concurrency, and reload policies.

Configuration sources can read dynamic models, data dictionaries, and ordinary objects. The default key value member is Key/Value, and different field names can be specified through Mapping. When working with [Data Services](../data/services.md), you should first determine the identity and tenant scope where the configuration is read. The configuration table cannot be naturally regarded as globally visible data.

## Options Integration

OptionsConfigurationExtension aggregates the Zongsoft binder integration service and provides named options and configuration change tokens. There is already an actual Options property injected into the security credential provider:

Source: [framework/Zongsoft.Security/src/CredentialProvider.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/CredentialProvider.cs#L65) (excerpt; see source for context).

{% code title="CredentialProvider.cs" %}
```csharp
[Options("Security/Authority")]
public Configuration.AuthenticationOptions Options { get; set; }
```
{% endcode %}

The framework scanning and injection process recognizes this attribute and obtains AuthenticationOptions from the specified configuration path. The tag attribute itself does not start the configuration source, nor does it allow any object created manually with new to be automatically injected; the object should be obtained through the service build process that supports this injection convention, see [service registration and injection](services.md).

| object | function |
| --- | --- |
| OptionsConfigurator | Select the configuration by option name and invoke the binder |
| OptionsConfigurationChangeTokenSource | Handle configuration source change notifications to the Options monitoring mechanism |
| OptionsFactory | Create option instances to support framework dynamic models |

Whether configuration changes can be observed by the current consumer also depends on the consumption method and object lifecycle. Getting the value at a time, caching an option object, and using [IOptionsMonitor](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.options.ioptionsmonitor-1) _[Source](https://source.dot.net/#Microsoft.Extensions.Options/IOptionsMonitor.cs)_ to listen for updates are not the same semantics.

## Practical Suggestions

When starting from Discussions, first confirm that the option file enters the host, then check the final key and value, driver and service registration, and finally execute the affected business. Independent parsing testing can only prove that the format and binding are correct; database connection, file writing, and message subscription still need to be verified separately.

Keep configuration keys stable and place sensitive connection values in environment configuration. Unrecognized keys should report an error explicitly or enter a clear extension container. The setting should not be considered to have taken effect just because the resolver does not throw an exception. For the framework test entrance, see [Configuration test directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/test/Configuration).
