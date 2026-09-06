---
description: ".storages Deployment products of the database message storage project, Broker connection matching and recovery conditions."
icon: database
---

#.storages: message storage

`messaging/.storages` provides a database storage implementation for reliable messages. It saves the record of waiting for delivery and confirmation after being accepted by the Broker; it is not responsible for establishing the network connection of Kafka, RabbitMQ, MQTT or ZeroMQ.

## Project Composition

| composition | Purpose |
| --- | --- |
| `Zongsoft.Messaging.Storages.Data` | Message storage plugin supported by data engine |
| `database` | SQLite, MySQL, PostgreSQL, SQL Server table building resources |
| Mapping and scripts | Access messages through named commands of the data engine |
| storage factory | For Broker to create storage by name, explicit assembly is required |

The Redis store is located at [externals/redis](../../externals/projects/redis.md) and is not part of the database implementation of this project. Their application integration and recovery rules are summarized in [Reliable Delivery and Message Storage](../reliability.md).

## Integration Order

1. Select the database and deploy the Data, [Corresponding data-driven](../../data/drivers.md) and storage plugins.
2. Use the corresponding table creation resources of the project's database directory in an independent library, and retain the mappings and scripts in the deployment product.
3. Configure a connection with the exact same name as the Broker; the default ZeroMQ guardian Broker is named QueueServer.
4. Set the stable storage identity before starting, and then inject the selected factory into the Broker. LeastOnce is explicitly enabled for both publishing and subscription.
5. Verify acceptance, handle failure, confirmation, restart and expiration messages before using them for business.

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft messaging storages]
nuget:Zongsoft.Messaging.Storages.Data
```
{% endcode %}

The above only adds the storage package. The complete solution also requires Data, database driver and Broker. Connection and injection examples are maintained in [Reliable Delivery and Message Storage](../reliability.md).

## Configuration and Identity

The factory searches for the connection with the same name as the Broker in /Data/ConnectionSettings and /Messaging/Storages/ConnectionSettings in turn, and does not fallback the default business connection. Factory nodes use Sqlite, MySql, PostgreSql, MsSql names, database connection driver keys cannot be used as node names.

Environment variable `ZONGSOFT_MESSAGING_STORAGE_IDENTIFIER` freezes on first use of factory. It and the connection name determine the storage partition; fallback machine name when not set. After the container is rebuilt or the machine is renamed, it is necessary to ensure that the same logical Broker still uses a stable identity.

## Restoration and Maintenance

{% hint style="warning" %}
🚨 Installing the message store does not mean automatically obtaining the transaction outbox, nor does it mean that the message will definitely enter the queue when there is no online matching consumer. Whether to accept, when to re-submit, and how to delete after confirmation are jointly decided by the Broker protocol and the storage implementation.
{% endhint %}

Expired records in the database will be filtered when reading, but will not be automatically cleaned in the background by this plugin. Maintenance should define partitions and check for legacy identity and legacy protocol data compatibility. The business must still design idempotent processing to avoid repeated side effects caused by failures before and after confirmation.

[ZeroMQ project](zero.md) · [Message item index](README.md) · [source code and database resources](https://github.com/Zongsoft/framework/tree/main/messaging/.storages)
