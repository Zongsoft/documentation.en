---
description: "Deployment, connections, protocol mappings and reliability boundaries for the RabbitMQ messaging project."
icon: message
---

# RabbitMQ

RabbitMQ.Client integrates topic switches and message queues, suitable for applications that use routing and work queues to distribute tasks. See [publish/subscribe and delivery](../concepts.md) for public concepts, and see [Message Queues](../../messaging.md) for cross-project comparison and complete calling examples.

| Item | value |
| --- | --- |
| source code directory | `messaging/rabbit` |
| NuGet package | `Zongsoft.Messaging.RabbitMQ` |
| provider and connection driver name | `RabbitMQ` |

## Deployment and Connection

Append the implementation package to the existing host's [Deployment checklist](../../../references/deploy-files.md), retaining the manifest and running dependencies. Broker address, account and topic permissions are provided by the operating environment.

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft messaging rabbit]
nuget:Zongsoft.Messaging.RabbitMQ
```
{% endcode %}

Source: [framework/messaging/rabbit/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/rabbit/samples/Program.cs#L17) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
using var queue = new RabbitQueue("RabbitMQ",
	Configuration.RabbitConnectionSettingsDriver.Instance.GetSettings("RabbitMQ", $"server=127.0.0.1;port=5672;client=Zongsoft.Messaging.RabbitMQ.Sample-{Guid.NewGuid():N};username=program;password=xxxxxx;"));
```
{% endcode %}

Discussions currently does not have a business process that integrates this queue, so the project's existing interactive client is used. The queue is constructed directly here, and the topic is passed in by the subscribe / produce command. xxxxxx in the RabbitMQ sample is the local test password placeholder value, which needs to be adjusted according to the isolation environment before running.

## Subscription Entry in the Example

Source: [framework/messaging/rabbit/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/rabbit/samples/Program.cs#L38) (excerpt; see source for context).

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

This implementation uses topic exchange; Group configures the switch, and Queue specifies the queue. Topics and tags participate in routing, and consumption uses manual ACK. The Group here cannot be understood according to the semantics of Kafka consumer groups.

## First Integration

1. Prepare the required permissions for test accounts, switches and queues, and check the connected virtual hosts.
2. Use the RabbitMQ provider to create named queues and verify topic routing and message arrival.
3. Confirm after the business is submitted, and then check for processing failures, disconnections, reconnections and duplicate messages.

Asynchronous business processing uses a public handler contract, and the asynchronous lambda cannot be handed over to the synchronous delegate subscription overload. The queue obtained by the provider may be shared, and the consumer only releases the subscription resources it owns; see [Message Queues](../../messaging.md) for a complete example.

## Reliability Boundary

{% hint style="warning" %}
🚨 Manual ACK only covers the consumer confirmation process. Durable queues, message persistence, publication acknowledgment, and Broker high availability need to be verified separately; one option cannot be generalized as a complete reliability guarantee.
{% endhint %}

The fault experiment needs to distinguish between publishing results, Broker acceptance, business commit and message confirmation, see [Reliable Delivery and Message Storage](../reliability.md).

## Project Resources

[Message item index](README.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/messaging/rabbit)
