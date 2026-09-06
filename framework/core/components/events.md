---
description: "Understand assembly boundaries for module events from the Event Registry entry in Discussions."
icon: bolt
---

# Events


Module events provide explicit event entry for business components. The Discussions Module inherits the [ApplicationModule](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationModule.cs) with event registry type parameters and exposes Events in the plugin tree.

Source: [src/Module.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Module.cs#L56) (excerpt; see source for context).

{% code title="Module.cs" %}
```csharp
public sealed class EventRegistry : EventRegistryBase
{
	#region 构造函数
	public EventRegistry() : base(NAME)
	{
	}
	#endregion
```
{% endcode %}

The EventRegistry currently does not declare a forum business event, so posting, moderation, or message sending cannot be described as having posted an event. It only provides extended locations.

## Plugin Exposure Event Entry

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L26) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<expose name="Events" value="{path:../@Events}" />
<expose name="Properties" value="{path:../@Properties}" />
```
{% endcode %}

Exposing the registry allows the plugin system to discover module events. Adding an event still requires defining the event name, payload, triggering time, processing method and failure strategy. In particular, it is necessary to distinguish between before and after transaction submission to avoid sending uncommitted data to external consumers.

## Full Reference in the Framework

Discussions There is no complete business event process yet, so you can continue to check the actual event definitions of frames [Component event implementation](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Components) and [security module](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Security/src). There needs to be a clear bridge between events and message queues, and it cannot be inferred that the forum has used a Broker just because the framework supports events.

Related concepts: [Handlers](handler.md), [Message Queues](../../messaging.md), [affairs](../../data/transactions.md).
