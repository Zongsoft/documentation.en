---
description: "Deployment, connections, protocol mapping and reliability boundaries of Kafka messaging project."
icon: message
---

# Kafka

Integrate framework message contracts with Kafka via Confluent.Kafka. Ideal for applications that organize event streams around topics, partitions, and consumer groups. See [publish/subscribe and delivery](../concepts.md) for public concepts, and see [Message Queues](../../messaging.md) for cross-project comparison and complete calling examples.

| Item | value |
| --- | --- |
| source code directory | `messaging/kafka` |
| NuGet package | `Zongsoft.Messaging.Kafka` |
| provider and connection driver name | `Kafka` |

## Deployment and Connection

Append the implementation package to the existing host's [Deployment checklist](../../../references/deploy-files.md), retaining the manifest and running dependencies. Broker address, account and topic permissions are provided by the operating environment.

{% code title="Existing host deployment manifest (append fragment)" %}
```ini
[plugins zongsoft messaging kafka]
nuget:Zongsoft.Messaging.Kafka
```
{% endcode %}

Source: [framework/messaging/kafka/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/kafka/samples/Program.cs#L17) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
using var queue = new KafkaQueue("Kafka",
	Configuration.KafkaConnectionSettingsDriver.Instance.GetSettings("Kafka", $"server=127.0.0.1:9092;client=Zongsoft.Messaging.Kafka.Sample-{Guid.NewGuid():N};"));
```
{% endcode %}

Discussions currently does not have a business process that integrates this queue, so the project's existing interactive client is used. The queue is constructed directly here, and the topic is passed in by the subscribe / produce command. Before running, you need to adjust the address, client ID, and permission configuration according to the isolation environment.

## Subscription Entry in the Example

Source: [framework/messaging/kafka/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/kafka/samples/Program.cs#L38) (excerpt; see source for context).

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

Topic corresponds to the Kafka topic, Group corresponds to the consumer group, and Client is the client identifier. Submit the consumer offset of the relevant partition when confirming the message; the order and consumption concurrency need to be understood in conjunction with the partition design.

## First Integration

1. Prepare independent test topics and consumer groups, and confirm the Broker address and client permissions.
2. Obtain the Kafka queue according to the public message example and use an asynchronous handler to subscribe.
3. Send messages with business identifiers, observe partitions, consumer offsets, and processing results, and then verify repetition and restart.

Asynchronous business processing uses a public handler contract, and the asynchronous lambda cannot be handed over to the synchronous delegate subscription overload. The queue obtained by the provider may be shared, and the consumer only releases the subscription resources it owns; see [Message Queues](../../messaging.md) for a complete example.

## Reliability Boundary

{% hint style="warning" %}
🚨 The current ConsumerConfig does not turn off the SDK’s automatic commit and automatic site recording. Explicit confirmation mapping to Commit does not mean that all unconfirmed messages must be redelivered; public confirmation methods cannot be used to claim end-to-end at least once.
{% endhint %}

The fault experiment needs to distinguish between publishing results, Broker acceptance, business commit and message confirmation, see [Reliable Delivery and Message Storage](../reliability.md).

## Project Resources

[Message item index](README.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/messaging/kafka)
