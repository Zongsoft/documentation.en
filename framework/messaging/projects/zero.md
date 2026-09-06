---
description: "Deployment, connections, protocol mapping and reliability boundaries of the ZeroMQ messaging project."
icon: message
---

# ZeroMQ

Based on NetMQ, it provides queues, built-in Broker and event/request response adaptation, which is suitable for internal message communication where the application controls the protocol and deployment boundaries. See [publish/subscribe and delivery](../concepts.md) for public concepts, and see [Message Queues](../../messaging.md) for cross-project comparison and complete calling examples.

| Item | value |
| --- | --- |
| source code directory | `messaging/zero` |
| NuGet package | `Zongsoft.Messaging.ZeroMQ` |
| provider and connection driver name | `ZeroMQ` |

## Deployment and Connection

Append the implementation package to the existing host's [Deployment checklist](../../../references/deploy-files.md), retaining the manifest and running dependencies. Broker is an additional running role and requires corresponding daemon list and server options; only configuring the following client connections will not automatically complete Broker and storage deployment.

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft messaging zero]
nuget:Zongsoft.Messaging.ZeroMQ
```
{% endcode %}

Source: [framework/messaging/zero/samples/client/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/zero/samples/client/Program.cs#L17) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
using var queue = new ZeroQueue("ZeroMQ",
	Configuration.ZeroConnectionSettingsDriver.Instance.GetSettings("ZeroMQ", "server=127.0.0.1;client=Zongsoft.Messaging.ZeroMQ.Sample;Group=Demo;"));
```
{% endcode %}

Discussions currently does not have a business process that integrates this queue, so the project's existing interactive client is used. The queue is constructed directly here, and the topic is passed in by the subscribe / produce command. Before running, you need to adjust the address, client ID, and permission configuration according to the isolation environment.

## Subscription Entry in the Example

Source: [framework/messaging/zero/samples/client/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/zero/samples/client/Program.cs#L41) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
executor.Command("subscribe", async (context, cancellation) =>
{
	if(context.Arguments.IsEmpty)
		throw new CommandException("Missing the topics for subscribe.");

	for(int i = 0; i < context.Arguments.Count; i++)
	{
		var subscriber = await queue.SubscribeAsync(context.Arguments[i], Handler.Instance, cancellation);

		if(subscriber == null)
			context.Output.WriteLine(CommandOutletColor.DarkRed, $"Failed to subscribe topic: {context.Arguments[i]}");
		else
			context.Output.WriteLine(CommandOutletColor.DarkGreen, $"The subscription to the '{subscriber.Topic}' topic was successful.");
	}
});
```
{% endcode %}

executor and Handler.Instance come from the same Program.cs. The client keeps the interactive loop, and the close command releases the queue it created directly. If the shared queue is obtained from the application provider instead, ownership needs to be handled according to the provider's agreement.

## Protocol Mapping

The same logical topic can be mapped to an online topic with Group prefix, and subscriptions adopt prefix matching. The default at-least-once path is used for broadcast; the at-least-once path requires Broker reliable storage and explicit publish and subscribe options.

## First Integration

1. First deploy and start the Broker role, check the port, and then configure the client connection.
2. Verify at-most-once publishing and subscribing with independent test groups and topics.
3. When reliability is required, press the storage topic to inject the factory and verify it at least once, confirm and restart the Broker for recovery.

Asynchronous business processing uses a public handler contract, and the asynchronous lambda cannot be handed over to the synchronous delegate subscription overload. The queue obtained by the provider may be shared, and the consumer only releases the subscription resources it owns; see [Message Queues](../../messaging.md) for a complete example.

## Reliability Boundary

{% hint style="warning" %}
🚨 LeastOnce only accepts new messages when there is an online matching subscription, and persists Pending first and then delivers; if there is no match, it returns null and does not save this message. The default broadcast has no offline reissue and ExactlyOnce is not supported.
{% endhint %}

The fault experiment needs to distinguish between publishing results, Broker acceptance, business commit and message confirmation, see [Reliable Delivery and Message Storage](../reliability.md). The database storage is also located at [.storages project](storages.md), and the Redis storage is located at [Redis external projects](../../externals/projects/redis.md).

## Project Resources

[Message item index](README.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/messaging/zero)
