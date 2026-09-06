---
description: "From messages, topics, consumers to acknowledgments and idempotency, understand the different delivery models behind the unified interface."
icon: envelopes-bulk
---

# Publish/Subscribe and Delivery Concepts

Messages separate "proposing work" and "doing work." The producer sends a fact or request, and the consumer processes it in its own execution environment. This can buffer traffic and separate lifecycles, but it also introduces delays, duplications and partial failures; successfully calling the interface no longer equals that the business has been completed.

## Identity in Message

| field or concept | function | Things that should not be confused with |
| --- | --- | --- |
| connection name | The application selects a configured queue connection | Driver name, physical queue name |
| Topic | Message routing or subscription scope | Wildcard characters are not universal for different Brokers |
| Group | Driver-specific grouping parameters | Kafka consumer group, RabbitMQ exchange, ZeroMQ prefixes have different meanings |
| Identifier | The identifier of a certain release or protocol message | It is not necessarily the key to stable business deduplication. |
| Identity | Identity information such as producer instances | Not directly equivalent to an authenticated user |
| Tags | Extend filtering or metadata | Some drivers do not use tags for routing |
| Data | business payload | Requires application of agreed encoding, structure and version |

It is recommended to include stable event identification, event type and necessary version information in the business payload. Preserve the business ID when retrying the same business event to avoid repeated side effects due to the adapter generating a new send ID each time.

## Broadcasting and Competitive Consumption

Broadcasting allows multiple subscribers to each see the same event, which is suitable for cache invalidation or status notifications. Competitive consumption lets a group of processors share work and is suitable for asynchronous tasks. Both may appear as "subscribing to the same topic", and the actual behavior must be judged based on the driver and group configuration.

For example, Kafka allocates partitions through consumer groups; ZeroMQ's at-most-once channel is broadcast, and at-least-once channels compete for delivery among online consumers. The business cannot just replace the connection driver and assume that the consumption quantity and order after expansion will be exactly the same.

## Reliability and Acknowledgment

"At most once" allows message loss but does not actively retry; "at least once" allows repetition to reduce the loss of unfinished delivery; "exactly once" is meaningful only within clearly defined protocols and status ranges. The protocol guarantees of MQTT QoS 2 cannot automatically cover the side effects of consumers writing to the database or calling third-party interfaces.

An acknowledgment tells the messaging system that a consumer has handled a message through the agreed stage. The framework expresses this action through `Message.AcknowledgeAsync`. Depending on the implementation, it may commit an offset, send an ACK, or do nothing when the protocol has no acknowledgment mechanism. A handler returning normally does not universally count as acknowledgment.

For the actual return value, confirmation conditions and background behavior of each driver, see [Message Queues](../messaging.md) and [reliable delivery](reliability.md).

## Sequence and Concurrency

Network reception sequence, processing start sequence, and business commit order are three different concepts. Concurrent processing, retries, and cross-partition consumption may change the completion order. When the same order needs to be processed in an orderly manner, the routing key, partition/consumer model and business version check should be clarified, and you cannot just rely on the sending end to call them one by one.

**backpressure** means that when the processing speed cannot keep up, the upstream is restricted from continuing to enter to prevent unlimited accumulation. This may manifest as a bounded memory queue, paused reads, or a backlog in the Broker. Backlog size and oldest message age should be monitored; simply increasing concurrency may shift pressure onto the database.

## Start with a Small End-to-End Test

First use two clients, independent topics and a small number of messages to verify sending, receiving and confirmation. Then have the handler fail intentionally, disconnect and reconnect from the client, and observe whether the message is re-delivered and the identity maintained. Real business side effects are added last.

Applications should independently define: how to identify duplicates, how many times to retry failures, how to isolate messages that cannot be processed, and how to stop receiving and waiting for in-transit tasks. The Unified Messaging interface provides call boundaries and does not automatically make these decisions for the business.
