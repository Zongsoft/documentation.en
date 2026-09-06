---
description: "Understanding plugin builtin and container services from Discussions module instances, validators and command dependencies."
icon: book-open
---

# Builtins and Services


Builtins are objects in the plugin tree and their assembly descriptions, and services are capabilities resolved through the container. A module instance can be referenced by the plugin tree at the same time and organize business objects through its own service container.

## Reuse Existing Objects

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

Use value here to refer to the existing Module.Current to avoid creating a second module. The type form is used to let the builder create new objects; validators and filters use this method.

## Services Are Discovered from Assembly

Source: [src/Services/ForumService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ForumService.cs#L41) (excerpt; see source for context).

{% code title="ForumService.cs" %}
```csharp
[Service(nameof(ForumService))]
[DataService(typeof(ForumCriteria))]
public class ForumService : DataServiceBase<Forum>
{
	#region 构造函数
	public ForumService(IServiceProvider serviceProvider) : base(serviceProvider) { }
```
{% endcode %}

After the assembly is loaded and scanned, the Service attribute description is registered. The constructor of ForumService receives the service provider, and subsequent accessors and associated services use this scope.

## Commands Depend on Business Services

Source: [src/Services/Commands/MessageSendCommand.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/Commands/MessageSendCommand.cs#L58) (excerpt; see source for context).

{% code title="MessageSendCommand.cs" %}
```csharp
[ServiceDependency(Provider = Module.NAME)]
public MessageService Service { get; set; }
```
{% endcode %}

Property injection specifies the Discussions module provider. The existence of the command class does not mean that it has entered the terminal command tree; currently Discussions.plugin does not mount MessageSendCommand, so you cannot directly fabricate a executable send command tutorial. See [Commands](../core/components/commands.md) for the actual logic of the command itself.

## Lifecycle and Ownership

Don't put request identities into shared builtin, and don't release container-owned services incrementally. Explicit calling context should be used when stateful operations are required. When the service cannot be found, first determine whether the failure occurred in the assembly scanning, module selection or builtin injection stage. For details, see [service resolution](../core/services/locating.md).
