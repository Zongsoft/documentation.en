---
description: "Check the boundaries of message acceptance and confirmation, configure database storage and design business idempotent for ZeroMQ."
icon: shield-check
---

# Reliable Delivery and Message Storage

Reliable delivery requires answering "Who is responsible for the message now?" A message may have entered the local client, reached the Broker, been written to persistent storage, been handed over to the consumer, or has completed business commit. Failures occur at different stages and are recovered in different ways.

## Define Business Completion Conditions First

Usually, complete business processing and record idempotency information before acknowledging a message. If the business transaction commits but the ACK is lost, the message can be delivered again, so store the deduplication record and business update in the same database transaction. For external payments, email, or other system calls, use the receiving system’s idempotency keys or a recoverable state workflow.

Sending a message and writing to the business database can also succeed independently: one may succeed while the other fails. To ensure eventual publication after a business commit, an application can implement a transactional outbox. Save pending events in the business transaction, then let a background worker send them and record the results. This is an application design, not a capability automatically provided by installing a message-storage plugin.

## Key Differences Between Implementations

| realize | Boundaries that currently require special checking |
| --- | --- |
| Kafka | Acknowledgment commits the consumer offset, but the current configuration leaves SDK auto-commit and automatic offset storage enabled. Withholding acknowledgment does not guarantee redelivery. |
| RabbitMQ | Consumers use manual ACKs. Durable queues, persistent messages, publisher confirms, and broker availability must each be verified separately. |
| MQTT | Publishing waits for the client’s publication result. QoS provides protocol-level reliability, not exactly-once business side effects. |
| ZeroMQ `MostOnce` | Immediate routing visibility and one local send, without persistent acceptance or redelivery. |
| ZeroMQ `LeastOnce` | Accepts a message only when an online matching subscriber exists. Persists Pending first, then delivers competitively and waits for an explicit ACK. |

{% hint style="warning" %}
🚨 Timeout or cancellation may occur after the remote end has accepted the message. At this time, "unknown result" should be recorded and restored through the business identifier. All exceptions cannot be interpreted as "the message must not have been sent."
{% endhint %}

## Prepare Database Storage for ZeroMQ

`Zongsoft.Messaging.Storages.Data` is an independent storage plugin that works through the named command of the data engine and supports SQLite, MySQL, PostgreSQL, and SQL Server. Redis has another storage implementation, see [Caching and Distributed Coordination](../externals/caching.md).

The preparation sequence is as follows:

1. Deploy the ZeroMQ Broker host, `Zongsoft.Data`, selected database driver, and `Zongsoft.Messaging.Storages.Data`.
2. Create `Messaging_Message` as [Table creation instructions for the corresponding database](https://github.com/Zongsoft/framework/tree/main/messaging/.storages/database); preserve the plugin's `.mapping` and `scripts` directories.
3. Configure a data connection with **exactly the same name** as the broker. The daemon plugin creates a broker named `QueueServer`.
4. Determine the stable storage identity before the process starts and inject the selected storage factory into the Broker.

Source: [framework/messaging/.storages/test/options/SQLite.option](https://github.com/Zongsoft/framework/blob/main/messaging/.storages/test/options/SQLite.option#L3) (excerpt; see source for context).

{% code title="SQLite.option" %}
```xml
<options>
	<option path="/Data">
		<connectionSettings>
			<connectionSetting connectionSetting.name="QueueServer" driver="SQLite"
			                   value="DataSource=provider.db" />
		</connectionSettings>
	</option>
</options>
```
{% endcode %}

The above is .storages testing existing SQLite options, using provider.db; Discussions has no Broker or message store configuration. The actual default contract test uses the temporary database created by yourself. See [Storage test instructions](https://github.com/Zongsoft/framework/blob/main/messaging/.storages/test/README.zh-Hans.md) for details. The application extension manifest should also declare dependencies on the actual messaging and storage plugins. The end section of the path can be selected as `Sqlite`, `MySql`, `PostgreSql`, or `MsSql`, matched according to the selected driver.

Existing storage plugins register their factories to /Workspace/Messaging/Storages, and the ZeroMQ daemon plugin's QueueServer builtin exposes the Storages dependency. The application extension manifest should assign the actual selected factory path to this property. For registration source codes, see [Data storage plugin](https://github.com/Zongsoft/framework/blob/main/messaging/.storages/src/Zongsoft.Messaging.Storages.Data.plugin) and [ZeroMQ daemon plugin](https://github.com/Zongsoft/framework/blob/main/messaging/zero/src/Zongsoft.Messaging.ZeroMQ-daemon.plugin) respectively.

{% code title="StartBroker.ps1" %}
```powershell
$env:ZONGSOFT_MESSAGING_STORAGE_IDENTIFIER = 'broker-storage-01'
# Then start the actual broker host in this environment.
```
{% endcode %}

The factory first accurately searches for the connection with the same name from `/Data/ConnectionSettings`, and then searches for `/Messaging/Storages/ConnectionSettings`, without falling back to the default business database. When the connection cannot be found, the name should be corrected. Do not change other databases to the default connection in order to bypass the error.

## Storage Identity and Recovery

A storage partition is identified by the connection name and a stable storage identity. The factory reads and freezes the identity environment variable on first use, falling back to the machine name when unset. Rebuilding a container or changing the machine or connection name can leave older messages in an original partition that is no longer visible.

When upgrading from an older `nodeId` configuration, the original values should be migrated to `ZONGSOFT_MESSAGING_STORAGE_IDENTIFIER` before first use. Protocol and data format compatibility need to be checked separately. Stable partition names cannot make old protocol payloads automatically compatible with new protocols.

Expired messages are filtered when reading, but database expired rows will not be automatically cleaned in the background by the plugin. A partition-defined maintenance strategy should be developed to avoid accumulating unbounded data or purging messages from other brokers.

## Enable At-Least-Once Delivery

Both publications and subscriptions must use `MessageReliability.LeastOnce` explicitly. The subscription option is `new MessageSubscribeOptions(MessageReliability.LeastOnce)` and the publishing option is `new MessageEnqueueOptions(MessageReliability.LeastOnce)`. The handler calls `AcknowledgeAsync` after completing the business operation.

When no matching subscription is online, the current broker returns `null` and does not save the new message. Existing Pending messages can be handled by matching consumers that reconnect or subscribe later. This model does not queue every newly published message while consumers are offline.

After acknowledgment, the broker stops delivery and deletes Pending asynchronously. Retries use the same message identifier. Applications must still handle failure windows between acknowledgment, storage deletion, and process exit.

## Acceptance Checklist

Verify business success, processing failure without confirmation, disconnection before and after ACK, Broker restart, storage unavailability, no online consumers, repeated consumption and expired data. Only by recording the business identification, message identification, Broker acceptance results and storage records in each experiment can we determine which layer of guarantee is in effect.

Source references: [ZeroMQ reliable protocol](https://github.com/Zongsoft/framework/blob/main/messaging/zero/PROTOCOL.zh-Hans.md), [Database message storage](https://github.com/Zongsoft/framework/tree/main/messaging/.storages).

## Continue Reading by Project

[ZeroMQ](projects/zero.md) · [.storages database storage](projects/storages.md) · [Redis storage](../externals/projects/redis.md) · [All Messaging Projects](projects/README.md)
