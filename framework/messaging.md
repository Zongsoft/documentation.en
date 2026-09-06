---
description: "Selection, configuration, implementation differences and usage examples of Zongsoft message queue plugin."
icon: message
---

# Message Queues

![Messages wait in the queue and are distributed to different receivers](../.gitbook/assets/zongsoft-messaging-cover.png)

Zongsoft's message queue system consists of the [Zongsoft.Messaging](core/messaging.md) core abstraction and four specific plugins. The core abstraction is responsible for unifying production, subscription, message confirmation and queue discovery; the plugin is responsible for implementing these abstractions into specific messaging systems.

Please read [Publish/Subscribe and Delivery Concepts](messaging/concepts.md) first when using it for the first time, and continue reading [Reliable Delivery and Message Storage](messaging/reliability.md) when you need to recover from a fault. This page explains the design differences, configuration methods, usage examples and precautions of the four implementations. Business code usually only needs to rely on the message abstraction of [`Zongsoft.Core`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core); application startup, deployment and connection parameters only need to care about specific plugins.

## Browse by Project

When a message implementation has been selected, [Kafka](messaging/projects/kafka.md), [RabbitMQ](messaging/projects/rabbit.md), [MQTT](messaging/projects/mqtt.md), [ZeroMQ](messaging/projects/zero.md) or [.storages message storage](messaging/projects/storages.md) can be entered directly. [Project index](messaging/projects/README.md) is organized by source code directory, and is read in conjunction with the common usage and protocols below.

## List of Plugins

| plugin | underlying library | Suitable for the scene | Main features |
| --- | --- | --- | --- |
| `Zongsoft.Messaging.Kafka` | `Confluent.Kafka` | High-throughput log streaming, event streaming, and consumer group processing. | Publish to Kafka topic, poll for consumption after subscription, and submit offset when confirming. |
| `Zongsoft.Messaging.RabbitMQ` | `RabbitMQ.Client` | Traditional message queues, routing, work queues and reliable delivery. | Using topic exchange, message expiration, priority, headers and manual ACK are supported. |
| `Zongsoft.Messaging.Mqtt` | `MQTTnet` | Device messages, lightweight publish/subscribe, and unstable network environments. | Publish directly through the connection manager, supporting reconnection, resubscription and QoS mapping. |
| `Zongsoft.Messaging.ZeroMQ` | `NetMQ` | Lightweight message forwarding and internal event channels between processes or within LAN. | Comes with a Broker that supports at-most-once broadcast, at-least-once persistent admission, and event/request response adaptation. |

{% hint style="info" %}
These plugins share the `IMessageQueue` interface, but the underlying protocols are not equivalent. Topics, tags, confirmations, retries, transactions, sequentiality, and persistence are all understood on an implementation-specific basis.
{% endhint %}

## Deployment Structure

Each message queue plugin contains three types of deployment files:

* `.plugin`: Register plugin assembly and connection settings driver.
* `.option`: Provides examples of default connection configurations.
* `.deploy`: Declare plugin manifest and third-party NuGet dependencies.

The plugin will mount the connection setting driver to `/Workbench/Configuration/ConnectionSettings/Drivers`, such as `Kafka`, `RabbitMQ`, `Mqtt`, and `ZeroMQ`. The application's message queue connection item is placed at `/Messaging/ConnectionSettings`.

## Real Client Connection Integration Port

Discussions does not use these queues directly, so the following uses the construction statements of the existing clients of each framework project. They call the connection settings driver directly, without going through the option file. RabbitMQ's xxxxxx is a sample placeholder value; Broker, account and permissions need to be provided by an independent testing environment. The plugin host configuration also uses these settings model, but should be placed in /Messaging/ConnectionSettings and the queue selected by the connection name.

{% tabs %}
{% tab title="Kafka" %}
Source: [framework/messaging/kafka/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/kafka/samples/Program.cs#L17) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
using var queue = new KafkaQueue("Kafka",
	Configuration.KafkaConnectionSettingsDriver.Instance.GetSettings("Kafka", $"server=127.0.0.1:9092;client=Zongsoft.Messaging.Kafka.Sample-{Guid.NewGuid():N};"));
```
{% endcode %}
{% endtab %}

{% tab title="RabbitMQ" %}
Source: [framework/messaging/rabbit/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/rabbit/samples/Program.cs#L17) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
using var queue = new RabbitQueue("RabbitMQ",
	Configuration.RabbitConnectionSettingsDriver.Instance.GetSettings("RabbitMQ", $"server=127.0.0.1;port=5672;client=Zongsoft.Messaging.RabbitMQ.Sample-{Guid.NewGuid():N};username=program;password=xxxxxx;"));
```
{% endcode %}
{% endtab %}

{% tab title="MQTT" %}
Source: [framework/messaging/mqtt/samples/client/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/mqtt/samples/client/Program.cs#L17) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
using var queue = new MqttQueue("MQTT",
	Configuration.MqttConnectionSettingsDriver.Instance.GetSettings("Mqtt", $"server=127.0.0.1:1883;client=Zongsoft.Messaging.Mqtt.Sample-{Guid.NewGuid():N};"));
```
{% endcode %}
{% endtab %}

{% tab title="ZeroMQ" %}
Source: [framework/messaging/zero/samples/client/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/zero/samples/client/Program.cs#L17) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
using var queue = new ZeroQueue("ZeroMQ",
	Configuration.ZeroConnectionSettingsDriver.Instance.GetSettings("ZeroMQ", "server=127.0.0.1;client=Zongsoft.Messaging.ZeroMQ.Sample;Group=Demo;"));
```
{% endcode %}
{% endtab %}
{% endtabs %}

`server`, `client`, `group`, `topic`, `timeout` are common connection properties, but the meaning will change with the implementation: Kafka's `group` is the consumer group; RabbitMQ's `group` is exchange; ZeroMQ's `group` will become the topic prefix; MQTT is retained in the current connection settings `group`, but the queue implementation does not use it for filtering or grouping.

## Get Queue

When resolving the queue through the core provider, the framework looks for a connection entry with a matching name from `/Messaging/ConnectionSettings`:

Source: [framework/Zongsoft.Core/src/Messaging/MessageQueueUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Messaging/MessageQueueUtility.cs#L40) (excerpt; see source for context).

{% code title="MessageQueueUtility.cs" %}
```csharp
public static IMessageQueue Queue(IServiceProvider services, string name, IEnumerable<KeyValuePair<string, string>> settings = null)
{
	name ??= string.Empty;
	services ??= ApplicationContext.Current?.Services ?? throw new ArgumentNullException(nameof(services));

	foreach(var provider in services.ResolveAll<IMessageQueueProvider>())
	{
		if(provider.Exists(name))
			return provider.Queue(name, settings);
	}

	return null;
}
```
{% endcode %}

When you need to bypass the configuration and directly use a plugin to construct the queue, you can use the corresponding connection settings driver:

The above four tabs have given the construction statements of the actual clients of each project. The direct constructor is responsible for release; see [Message core abstraction](core/messaging.md) for the relationship between configuration driver and provider.

## Real Publish/subscribe Example

All four implementations follow the same core usage: create or parse a queue, subscribe to a topic, send a message, and confirm after successful processing. The following snippet is from the Kafka sample that keeps an interactive loop; executor is the terminal executor of the program and context.Arguments is the subject of user input. MQTT and ZeroMQ filter self-publish by default and use two clients when authenticating or self-receive as configured by the implementation.

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

{% hint style="warning" %}
`AcknowledgeAsync()` in the example is intentionally placed after the processing logic. The production environment should first complete business processing, drop-in or idempotent recording, and then confirm the message.
{% endhint %}

The following is the processing method of Handler inside the same Kafka Program.cs. _count and Handler.Instance have been defined in this nested class. Asynchronous processing cannot be passed directly to the synchronous `System.Action<Message>` overload, otherwise it will form `async void`, and the framework cannot wait for it to complete or correctly observe the exception.

Source: [framework/messaging/kafka/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/kafka/samples/Program.cs#L121) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
protected override async ValueTask OnHandleAsync(Message message, Parameters parameters, CancellationToken cancellation)
{
	if(message.IsEmpty)
		return;

	var count = Interlocked.Increment(ref _count);
	var content = CommandOutletContent.Create()
		.Append(CommandOutletColor.Cyan, "[Received]")
		.Append(CommandOutletColor.DarkYellow, $"#{count}")
		.Append(CommandOutletColor.DarkCyan, " Topic:")
		.AppendLine(CommandOutletColor.DarkGreen, message.Topic)
		.AppendLine(CommandOutletColor.Gray, Encoding.UTF8.GetString(message.Data));

	Terminal.Console.Executor.Output.Write(content);
	await message.AcknowledgeAsync(cancellation);
}
```
{% endcode %}

## Kafka Implementation

`KafkaQueue` uses `ProducerBuilder<Null, byte[]>` to publish messages and `ConsumerBuilder<string, byte[]>` to create consumers. After the subscription is successful, `KafkaSubscriber` starts a `MessagePollerBase` poller and calls `Consume(...)` of Kafka consumer to obtain the message.

Key behaviors:

* Publications must specify a non-empty topic; the default topic is `topic` from the connection settings.
* Publishing returns Kafka's `TopicPartition` string, not the business message ID.
* After receiving the message, `Message` is constructed, and the confirmation callback will execute `_consumer.Commit(result)`. The current connection configuration does not turn off the client's automatic commit/automatic offset storage site, so explicit confirmation does not guarantee "unconfirmed redelivery"; strict business confirmation requirements must verify the actual consumption configuration and restart recovery.
* `group` maps to Kafka `GroupId`; when not specified, a random consumer group is generated.
* `client` maps to Kafka `ClientId`; when not specified, a random client ID is generated.
* Connection properties such as `heartbeat`, `timeout`, `transactionId`, `transactionTimeout`, etc. are mapped to the Kafka configuration.
* The current implementation does not map `tags`, `Delay`, `Expiration`, `Priority` to Kafka messages.

Source: [framework/messaging/kafka/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/messaging/kafka/samples/Program.cs#L66) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
executor.Command("produce", async (context, cancellation) =>
{
	var round = context.Options.GetValue<int>("round", 1);
	var topic = context.Options.GetValue<string>("topic");

	if(string.IsNullOrEmpty(topic))
		throw new CommandOptionException("topic", "The topic is required.");

	var stopwatch = System.Diagnostics.Stopwatch.StartNew();

	for(int i = 0; i < round; i++)
	{
		for(int j = 0; j < context.Arguments.Count; j++)
		{
			var identifier = await queue.ProduceAsync(
				topic,
				Encoding.UTF8.GetBytes($"[{i + 1}]{context.Arguments[j]}"),
				null,
				cancellation);

			context.Output.WriteLine(CommandOutletColor.DarkGreen, $"[{i + 1}] {topic} Sent. (Identifier:{identifier ?? "N/A"})");
		}
	}

	stopwatch.Stop();
	context.Output.WriteLine(CommandOutletColor.Magenta, $"Elapsed: {stopwatch.Elapsed}");
});
```
{% endcode %}

## RabbitMQ Implementation

`RabbitQueue` uses RabbitMQ topic exchange. When `group` is empty, the exchange name is `/`, otherwise `group` is used; when `queue` is empty, a temporary queue is declared, otherwise a persistent queue is declared. When subscribing, `/` in the topic will be converted to `.`, and an empty topic will subscribe to `#`.

Key behaviors:

* Connections, channels, exchanges, and queues are initialized before publishing.
* The topic routing key will convert `/` to `.`, which is suitable for mapping application layer path topics to RabbitMQ topic routing keys.
* `MessageEnqueueOptions.Priority` Write message priority.
* `MessageEnqueueOptions.Expiration` writes the message expiration time.
* `MessageEnqueueOptions.Properties` is written to RabbitMQ headers.
* Each message generates `MessageId`, which is returned after publishing.
* If automatic confirmation is turned off when consuming, `Message.AcknowledgeAsync()` will execute `BasicAckAsync(...)`.
* `tags` is currently passed in as a consumer tag, not as a RabbitMQ binding filter.

See [RabbitMQ interactive client](https://github.com/Zongsoft/framework/blob/main/messaging/rabbit/samples/Program.cs) for the complete implementation of publishing, subscribing and terminal parameter processing, subject to the actual topic parameters and Group / Queue settings in the connection.

## MQTT Implementation

`MqttQueue` obtains the MQTTnet client through the connection manager, directly calls the publish and subscribe methods, and releases the connection usage rights after completion. The connection manager is responsible for connecting, reconnecting and resuming subscriptions; message processing uses bounded concurrency, so it cannot be assumed that the order of business processing completion is equal to the order of network reception.

Key behaviors:

* `server` can be written as `host:port` or a connection URI containing the protocol.
* A random client ID is generated when `client` is not specified.
* Publish and wait for `PublishAsync(...)` to be returned; the failure result is converted into an exception, and the available MQTT message ID is returned upon success. This identifier is not a globally unique ID for the business, nor does it represent the completion of consumer processing.
* `MessageReliability` will be mapped to MQTT QoS.
* Under MQTT 5, `Properties` maps to user properties and `Expiration` maps to message expiration intervals; MQTT 3 cannot be assumed to have these properties.
* Use `NoLocal = true` for subscription to avoid receiving messages on the same topic published by this client.
* Automatic confirmation is turned off when a message is received, and the handler calls `AcknowledgeAsync()` before confirming.
* `tags` currently does not participate in MQTT topic filter.

The client and server are from [MQTT samples](https://github.com/Zongsoft/framework/tree/main/messaging/mqtt/samples). Please use two clients with different Client identities to complete publishing and receiving, and handler explicitly confirms after terminal output; it does not implement the fictitious device temperature storage business.

## ZeroMQ Implementation

`ZeroQueueServer` now contains two delivery channels: `MostOnce` broadcasts through XPUB/XSUB, and `LeastOnce` registers consumers, persistently accepts messages, competes for delivery, and handles confirmation through the Control channel. The reliable channel is only started after the Broker configures message storage, and the publisher does not save messages to be delivered.

| mode | What does `ProduceAsync` completion mean? | When there is no online matching subscription |
| --- | --- | --- |
| `MostOnce` | The matching subscription is visible to the current publisher and a local send is completed. | Return `null` and will not wait for future subscriptions or reissues |
| `LeastOnce` | The Broker has written the Pending message to the storage | Return `null`, do not write to storage |
| `ExactlyOnce` | Not supported | Request denied before transfer status is established |

A non-null return value in both modes is not a handler completion notification. In at-least-once mode, the handler must explicitly call `AcknowledgeAsync()`; if it is not confirmed, the same message ID will be used for re-delivery, or it may be given to another online consumer.

### Ports and Deployment

The discovered port defaults to `7969`. The running port is obtained through the discovery protocol. When three values are configured, it is `Control,Incoming,Outgoing`; when two values are configured, it is `Incoming,Outgoing`. Control is randomly bound after storage is enabled. Running ports of unspecified or specified `*` can be randomly assigned.

Source: [framework/messaging/zero/src/Zongsoft.Messaging.ZeroMQ.option](https://github.com/Zongsoft/framework/blob/main/messaging/zero/src/Zongsoft.Messaging.ZeroMQ.option#L11) (excerpt; see source for context).

{% code title="Zongsoft.Messaging.ZeroMQ.option" %}
```xml
<option path="/Messaging/ZeroMQ">
	<servers port="32100,32101,32102">
		<server server.name="unnamed" port="*" />
	</servers>
</option>
```
{% endcode %}

The master plugin registers clients, event channels and request response adaptations; the guardian plugin is responsible for starting the Broker. When a reliable channel is required, continue to configure [Message storage](messaging/reliability.md). Just filling in the Control port will not enable persistence.

### Routing and Lifecycle

* `group` adds the `group:` prefix to the physical topic, and the handler still gets the logical topic. Subscriptions use prefix matching.
* By default, messages published by its own instance are filtered out; for same-process demonstrations, `filter=*` can be set. For formal isolation, appropriate instance and group configurations should still be used.
* Subscriptions are reused when the same topic, handler, label and normalization options are consistent; when there are incompatible subscriptions on the same topic, a conflict exception will be thrown and the original handler will not be replaced.
* The handlers of a single subscription are executed serially in the order they are received, and the bounded queue forms the backpressure of that subscription. The handler should handle cancellation promptly and cancel its own subscription when stopped.
* The provider may reuse the queue and do not release the shared queue after a business operation. A separate program that directly constructs the queue is responsible for the complete lifecycle.

`Compression` supports Brotli, GZip, ZLib, and Deflate. The threshold represents the number of payload bytes, such as `new MessageCompression("Brotli", 4096)`. Positive numbers `Delay` are not supported and will be rejected by the core capability check; the presence of the option does not mean that every driver implements it.

{% hint style="warning" %}
🚨 The current line protocol is `1.0` and cannot be mixed with old protocol clients, Brokers or old Pending envelopes. When upgrading, you should first stop publishing, process the original pending delivery messages, and formulate a storage migration or switching plan; do not clear the production storage directly. The Broker endpoint binds all network interfaces, and the adapter itself does not configure authentication and transmission encryption.
{% endhint %}

When independent verification is required, use the server and client interaction examples in the table below to keep the process alive and wait for the result to be explicitly received. Don't rely on fixed delays or repeated releases to prove that the first message was reliably delivered.

## Sample Project

There are samples for each implementation under the source code directory `framework/messaging`:

| realize | Example | Description |
| --- | --- | --- |
| Kafka | `messaging/kafka/samples/Program.cs` | Interactive commands receive topic, message content and round, explicit subscription and confirmation. |
| RabbitMQ | `messaging/rabbit/samples/Program.cs` | Interactive clients subscribe, publish, unsubscribe, and close by command parameters. |
| MQTT | `messaging/mqtt/samples/server/Program.cs`、`messaging/mqtt/samples/client/Program.cs` | Interactive broker and client that can verify publications, subscriptions, reconnections and confirmations. |
| ZeroMQ | `messaging/zero/samples/server/Program.cs`、`messaging/zero/samples/client/Program.cs` | The server starts the forwarder; the client subscribes, unsubscribes and publishes messages through terminal commands. |

## Usage Suggestions

* Kafka is preferred when high throughput, consumer groups, and event stream processing are required.
* RabbitMQ is preferred when traditional queues, routing keys, message expiration, priorities, and manual acknowledgments are required.
* MQTT is preferred when device messaging, lightweight connections, QoS, and automatic reconnection are required.
* ZeroMQ can be used when lightweight forwarding, event channels, or local/LAN communication within the framework are required.
* The business handler should be designed according to "possible duplicate delivery" and use business unique keys or message identifiers as idempotent.
* Don't rely on all plugins supporting tags, delays, expiration, priorities, or exact-once; confirm the mappings implemented before using them.

## Related Resources

* [Zongsoft.Messaging core abstraction](core/messaging.md)
* [Connection Configuration](data/connections.md)
* [Plugin Manifests and Loading](plugins/plugin-file.md)
* [Message Queuing source code directory](https://github.com/Zongsoft/framework/tree/main/messaging)
