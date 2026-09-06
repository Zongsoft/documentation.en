---
description: "Understand the business model, service registration, plugin manifest and deliverables from existing modules in Discussions."
icon: puzzle-piece
---

# Write Your First Business Plugin


This article directly reads and builds the Discussions business plugin. First understand the division of labor of the framework from the existing implementation, and then expand the requirements on the same business model. The prerequisites are the SDK and discussions source code required by [Prerequisites](prerequisites.md).

## 1. Build an Existing Business Library

The project supports .NET 8, 9, and 10; NuGet versions are defined in the root Directory.Packages.props. See [Prerequisites](prerequisites.md) to prepare the SDK and source directories. The following example uses local references: from the discussions root, first build [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) and Web in the sibling framework repository, then build Discussions. All commands below use the default Debug configuration, target .NET 10, and disable package generation:

{% code title="Build Discussions" %}
```powershell
dotnet build ../framework/Zongsoft.Core/src/Zongsoft.Core.csproj -f net10.0 -p:GeneratePackageOnBuild=false
dotnet build ../framework/Zongsoft.Web/src/Zongsoft.Web.csproj -f net10.0 -p:GeneratePackageOnBuild=false
dotnet build src/Zongsoft.Discussions.csproj -f net10.0 -p:ZongsoftFrameworkPathReferenced=true -p:GeneratePackageOnBuild=false
dotnet build src/api/Zongsoft.Discussions.Web.csproj -f net10.0 -p:ZongsoftFrameworkPathReferenced=true -p:GeneratePackageOnBuild=false
```
{% endcode %}

`-p:ZongsoftFrameworkPathReferenced=true` selects local assemblies but does not build framework automatically. Paths, configuration, and target frameworks must match its output. If your NuGet source already provides the required versions of [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) and the other dependencies, you can omit the property and use the default package references.

## 2. The Module Is the Composition Entry Point

Source: [src/Module.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Module.cs#L31) (excerpt; see source for context).

{% code title="Module.cs" %}
```csharp
[assembly: ApplicationModule(Zongsoft.Discussions.Module.NAME)]

namespace Zongsoft.Discussions;

public class Module : ApplicationModule<Module.EventRegistry>
{
	#region 常量定义
	/// <summary>表示论坛模块的名称常量值。</summary>
	public const string NAME = nameof(Discussions);
	#endregion
```
{% endcode %}

The assembly declaration places the service into the Discussions module. Module.Current provides a shared module instance; do not recreate it for each request. See [service resolution](../framework/core/services/locating.md) for the relationship between module accessors and service containers.

## 3. Services Carry Business Actions

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L41) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
[Service(nameof(ThreadService))]
[DataService(typeof(ThreadCriteria))]
public class ThreadService : DataServiceBase<Models.Thread>
{
	#region 成员字段
	private PostService _posting;
	#endregion

	#region 构造函数
	public ThreadService(IServiceProvider serviceProvider) : base(serviceProvider) { }
```
{% endcode %}

The data model of ThreadService is Thread, and the query condition model is ThreadCriteria. The service also implements business actions such as moderation, locking, and pinning. The controller is only responsible for transferring requests to the service. Read [Data Services](../framework/data/services.md) first, do not copy the moderation rules directly to each controller.

## 4. Connect the Module to the Framework Through Its Manifest

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L20) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<extension path="/Workbench/Modules">
	<object name="Discussions" value="{static:Zongsoft.Discussions.Module.Current, Zongsoft.Discussions}">
		<expose name="Accessor" value="{path:../@Accessor}">
			<expose name="Filters" value="{path:../@Filters}" />
		</expose>

		<expose name="Events" value="{path:../@Events}" />
		<expose name="Properties" value="{path:../@Properties}" />
	</object>
</extension>
```
{% endcode %}

What is registered here is the existing Module.Current. Accessor.Filters allows the plugin tree to continue to mount query filters; Events and Properties expose the event registry and module properties respectively. The manifest also includes data validators, security challengers and identity converters, see [plugin manifest](../framework/plugins/plugin-file.md) for details.

## 5. Deliver Assembly and Metadata

Source: [src/Zongsoft.Discussions.deploy](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.deploy#L1) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.deploy" %}
```ini
artifacts/Zongsoft.Discussions.plugin
artifacts/Zongsoft.Discussions.option
artifacts/Zongsoft.Discussions.mapping
lib/$(Framework)/Zongsoft.Discussions.*
```
{% endcode %}

Simply copying the DLL is not enough to run the forum. The mapping defines the entity relationship, the options define the base path for file storage, and the plugin manifest defines where objects are mounted. The Web layer also has independent manifests and archive templates, which are delivered together according to [Deploy Your First Plugin](deploy-first-plugin.md).

## Where to Start When Expanding

When adding business fields, check the model, mapping, four SQL scripts and query modes at the same time; when adding business actions, place them in the service first, and then do HTTP adaptation. For forum data, any extension must preserve SiteId isolation, moderation visibility, and author permissions; see [real case](../cases.md).
