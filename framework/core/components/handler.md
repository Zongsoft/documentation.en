---
description: "Zongsoft.Components Handler handler abstraction, positioning and selection."
icon: hand-pointer
---

# Handler

`Handler` is used to express "an object capable of handling a certain type of context or request". It is looser than a concrete service interface and is suitable for mounting extensible handler collections in plugin trees, messaging systems, event channels, and schedulers.

The handler model focuses on "who can handle this." The caller can get a handler collection, and then the locator, selector or host process selects the appropriate handler for execution.

## Key Types

| Type | Description |
| --- | --- |
| `IHandler`、`IHandler<TContext>` | handler interface. |
| `IHandler<TArgument, TResult>` | Handler interface with return value. |
| `IHandleable`、`IHandleable<TContext>` | Processable object interface, used to expose processing capabilities. |
| `IHandlerLocator` | handler locator. |
| `HandlerBase<TContext>`、`HandlerBase<TContext, TResult>` | handler base class. |
| `HandlerSelector` | Select a handler based on URL or context. |
| `HandlerAttribute` | Declare the URL template and order for the handler. |
| `HandlerUtility` | handler URL, name, and metadata helper methods. |
| `Handler` | Static factory, which can wrap delegate into handler proxy. |

Source: [framework/Zongsoft.Diagnostics/protocols/server/samples/MetricHandler.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Diagnostics/protocols/server/samples/MetricHandler.cs#L12) (excerpt; see source for context).

{% code title="MetricHandler.cs" %}
```csharp
public class MetricHandler : HandlerBase<IEnumerable<Zongsoft.Diagnostics.Telemetry.Metrics.Meter>>
{
	protected override ValueTask OnHandleAsync(IEnumerable<Telemetry.Metrics.Meter> meters, Parameters parameters, CancellationToken cancellation)
	{
		foreach(var meter in meters)
			Terminal.WriteLine(CommandOutletDumper.Dump(meter));

		return ValueTask.CompletedTask;
	}
}
```
{% endcode %}

This handler comes from the diagnostics protocol server example: the input is a set of Meters that have been converted into a framework model, and the processing methods are output to the terminal one by one. It does not declare a HandlerAttribute; the listener obtains it through the plugin collection. The URL template capability of HandlerUtility is another mechanism for selecting handlers by address, and it cannot be inferred that all handlers require URLs.

## Plugin Architecture Handler Collection

The diagnostics example hooks the MetricHandler into the collection exposed by the Metrics listener. The path, object name and type here are all from the same sample plugin; see [OTLP receive](../../diagnostics/otlp.md) for the startup sequence and listening address.

Source: [framework/Zongsoft.Diagnostics/protocols/server/samples/Zongsoft.Diagnostics.Protocols.Server.Samples.plugin](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Diagnostics/protocols/server/samples/Zongsoft.Diagnostics.Protocols.Server.Samples.plugin#L19) (excerpt; see source for context).

{% code title="Zongsoft.Diagnostics.Protocols.Server.Samples.plugin" %}
```xml
<extension path="/Workbench/Diagnostics/Telemetry/Listener/Metrics">
	<object name="MetricHandler" type="Zongsoft.Diagnostics.Protocols.Server.Samples.MetricHandler, Zongsoft.Diagnostics.Protocols.Server.Samples" />
</extension>
```
{% endcode %}

The focus of this model is not "how many methods a certain interface has", but "which handlers a certain extension point can receive." As long as the business module implements the handler and mounts it to the collection, it can participate in the message processing process.

## Differences from Commands and Events

| model | focus |
| --- | --- |
| Commands | The caller submits an expression, which is parsed and executed by the command tree. |
| Events | The publisher declares "what happened" and the handler is decoupled and extended via event nodes. |
| handler | The extension point maintains a handler collection, and the appropriate handler is selected by a locator or selector. |

The handler is suitable for plugin architecture extension point, message response, event delivery and scheduling tasks. If the caller explicitly relies on a domain service and a strongly typed business contract is required, it is often clearer to define the service interface directly. `HandlerSelector` is a local selector based on the handler URL template. Complex routing, permission judgment or load balancing should usually be left to more specialized mechanisms.

## Reference Implementation

* [Handler related source code](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Components)
* [Zongsoft.Messaging.ZeroMQ.plugin](https://github.com/Zongsoft/framework/blob/main/messaging/zero/src/Zongsoft.Messaging.ZeroMQ.plugin)
