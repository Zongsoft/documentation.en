---
description: "Find messaging implementations by their Kafka, RabbitMQ, MQTT, ZeroMQ, and .storages source projects."
icon: folders
---

# Browse by Project

Use this page to find packages, configuration, and implementation boundaries for a specific broker or source project. [Message Queues](../../messaging.md) compares implementations and provides shared usage examples; [Delivery Concepts](../concepts.md) and [Reliability](../reliability.md) explain their common background.

| Source Project | Documentation | Responsibilities |
| --- | --- | --- |
| kafka | [Kafka](kafka.md) | Kafka client, topic partitioning and consumer offset |
| rabbit | [RabbitMQ](rabbit.md) | Exchange routing, queues, and consumer acknowledgments |
| mqtt | [MQTT](mqtt.md) | Device publish/subscribe, topic filter and QoS |
| zero | [ZeroMQ](zero.md) | Client, built-in Broker and two delivery paths |
| .storages | [Message storage](storages.md) | Database persistent records and storage factories for use by Brokers |

## Understand How the Roles Fit Together

The first four projects implement messaging, while .storages supplies optional persistent storage. With ZeroMQ, for example, the client, broker, and database storage may run in different processes or on different nodes. Connection and storage settings configure their respective roles.

{% hint style="info" %}
💡 Group, Queue, Topic and confirmation operations with similar names may have different meanings in different protocols. When migrating a project, first read the mapping and restrictions of the target project, and then reuse the public business handler.
{% endhint %}
