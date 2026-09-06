---
description: "Deployment, connection, protocol mapping and reliability boundaries of MQTT messaging project."
icon: message
---

# MQTT

Suitable for device telemetry and lightweight message exchange via MQTTnet integration MQTT publish/subscribe. Topic filters and protocol-level QoS are the main configuration semantics. See [publish/subscribe and delivery](../concepts.md) for public concepts, and see [Message Queues](../../messaging.md) for cross-project comparison and complete calling examples.

| Item | value |
| --- | --- |
| source code directory | `messaging/mqtt` |
| NuGet package | `Zongsoft.Messaging.Mqtt` |
| provider and connection driver name | `Mqtt` |

## Deployment and Connection

Append the implementation package to the existing host's [Deployment checklist](../../../references/deploy-files.md), retaining the manifest and running dependencies. Broker address, account and topic permissions are provided by the operating environment.

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft messaging mqtt]
nuget:Zongsoft.Messaging.Mqtt
```
{% endcode %}

Source: [framework/messaging/mqtt/samples/client/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/mqtt/samples/client/Program.cs#L17) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
using var queue = new MqttQueue("MQTT",
	Configuration.MqttConnectionSettingsDriver.Instance.GetSettings("Mqtt", $"server=127.0.0.1:1883;client=Zongsoft.Messaging.Mqtt.Sample-{Guid.NewGuid():N};"));
```
{% endcode %}

Discussions currently does not have a business process that integrates this queue, so the project's existing interactive client is used. The queue is constructed directly here, and the topic is passed in by the subscribe / produce command. Before running, you need to adjust the address, client ID, and permission configuration according to the isolation environment.

## Subscription Entry in the Example

Source: [framework/messaging/mqtt/samples/client/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/mqtt/samples/client/Program.cs#L38) (excerpt; see source for context).

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

Subscriptions can use the MQTT topic filter. Publishing should specify a specific topic and do not use wildcards as the actual publishing topic. Currently publishing directly via the connection manager and handling reconnections and resubscriptions.

## First Integration

1. Set a clear and non-conflicting Client identity for the test client and check the topics allowed by the Broker.
2. Use Mqtt provider to subscribe to filter and publish to a specific topic from another client.
3. Verify required QoS, disconnection recovery, explicit acknowledgments, and duplicate messages after subscription recovery.

Asynchronous business processing uses a public handler contract, and the asynchronous lambda cannot be handed over to the synchronous delegate subscription overload. The queue obtained by the provider may be shared, and the consumer only releases the subscription resources it owns; see [Message Queues](../../messaging.md) for a complete example.

## Reliability Boundary

{% hint style="warning" %}
🚨 The current subscription setting is NoLocal, two clients should be used to verify publishing and receiving, and the same client cannot be relied on to retract its own messages. Protocol-level QoS does not represent business side effects exactly once; applications still need to be idempotent to handle.
{% endhint %}

The fault experiment needs to distinguish between publishing results, Broker acceptance, business commit and message confirmation, see [Reliable Delivery and Message Storage](../reliability.md).

## Project Resources

[Message item index](README.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/messaging/mqtt)
