---
description: "Configure Redis cache, etcd sequence number and lease lock, and understand Garnet server and reliable message storage."
icon: database
---

# Caching and Distributed Coordination

Caching, sequence numbers, and locks may all use key-value stores, but for different purposes. Cache allows invalidation or reconstruction according to business policies; sequence numbers need to be allocated atomically; locks limit concurrency through ownership. "Being able to read and write a key" cannot be taken to mean that these three types of capabilities have the same consistency guarantee.

## Component Selection

| Expand | Current responsibilities | Use the entrance |
| --- | --- | --- |
| Redis | Caching, sequence numbers, locks, message flow, configuration and message storage | Registered `Redis` provider and related plugin tree nodes |
| etcd | Basic KV, serial number, lease lock | Serial number/lock provider; not a public distributed cache implementation |
| Garnet | Redis protocol server started with the host | Worker and server settings; clients still need corresponding adapters |

## Redis Cache Closed Loop

Discussions has no business use case that directly calls the Redis cache; the framework's distributedcache is a runnable interactive client. It creates a separate instance named Redis and uses the DistributedCache key prefix. Before starting, follow the [sample instructions](https://github.com/Zongsoft/framework/blob/main/externals/redis/samples/distributedcache/README.zh-Hans.md) to prepare a test Redis instance and replace the placeholder password in the local connection.

The set command reads the key, value, expiration time and write conditions from user input, and finally calls:

Source: [framework/externals/redis/samples/distributedcache/Program.cs](https://github.com/Zongsoft/framework/blob/main/externals/redis/samples/distributedcache/Program.cs#L40) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
var succeeded = expiry.HasValue ?
	await cache.SetValueAsync(key, value, expiry.Value, requisite, cancellation) :
	await cache.SetValueAsync(key, value, requisite, cancellation);
```
{% endcode %}

The get command retrieves both the value and the remaining validity period:

Source: [framework/externals/redis/samples/distributedcache/Program.cs](https://github.com/Zongsoft/framework/blob/main/externals/redis/samples/distributedcache/Program.cs#L65) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
var (value, expiry) = await cache.GetValueExpiryAsync<string>(key, cancellation);
```
{% endcode %}

The complete program provides set, get, exists, expiry, remove and subscribe commands to facilitate observing cross-process reading, writing and notification under the same prefix. The independent program is responsible for releasing the RedisService created by itself; the shared cache obtained through the host provider is managed by the host, and the consumer should not release it one after another. For the resolution and fallback rules of named services, see [service resolution](../core/services/locating.md).

Ordinary Redis service may fallback the default connection when it cannot find the named connection. If the connection name is misspelled, it will not always fail immediately. The startup check should verify the actual selected configuration and isolate the data with business key prefixes.

## Scope and Notifications

Redis services can create immutable scopes through `WithDatabase()` and `WithNamespace()`. The old `Use()` or variable `Namespace` can only be set before first use and are not suitable for modifying the shared instance on every request.

Cache change notification requires the server to enable corresponding key space notification. Notifications adopt Pub/Sub semantics and will not be replayed when disconnected; local subscription queues also have capacity and overflow policies, so notifications cannot be used as business event logs that must be completely saved. The configuration provider is reloaded based on the local snapshot, also taking into account the state during the disconnection period.

## Etcd Serial Number and Lock

The etcd settings are located at `/Externals/Etcd/ConnectionSettings` and the driver is `etcd`. The serial number is obtained through the public `ISequence` provider; when multiple providers exist at the same time, the selected implementation is explicitly injected by the application composition layer.

{% hint style="info" %}
💡 The current etcd provider has not registered the `Etcd` alias, and you cannot obtain the etcd service by simply replacing the provider name in the named service expression. The connection name also does not automatically become the etcd key namespace.
{% endhint %}

The sequence number is incremented using a compare and swap transaction, which returns `seed + interval` for the first time, not `seed`. Expiration settings are used to create missing sequence numbers; increments preserve existing leases. Atomic allocation of sequence numbers does not mean uninterrupted business numbers, nor does it automatically form the same transaction as business data submission.

Both Redis and etcd locks need to understand **lease**and**fencing token**: the old process may still continue to execute after the lease expires, and the resource side should reject writes from expired holders. Automatic renewal needs to be configured explicitly. Work that relies on the lock should be stopped when the renewal fails or the result is uncertain. See [Distributed Locks](../core/services/distributed-lock.md) for a complete calling example.

## Garnet Server

`Zongsoft.Externals.Garnet` hosts Garnet servers via workers. The configuration path is `/Externals/Garnet`, and `value` named `server` is converted to a server option. Plugin worker startup may open listening ports, so the address, authentication, and persistence directories should be explicitly bound before enabling.

Relative directories usually resolve from the adapter assembly location, `~/` resolves from the application root directory. When AOF or checkpointing is enabled, stop, restart, and recovery should be tested; the in-process server shares resources and failure boundaries with the host. Redis protocol compatibility does not mean that all Redis commands and persistence behavior are the same.

## Reliable Message Storage and Troubleshooting

The message storage factory of Redis is mounted on `/Workspace/Messaging/Storages/Redis`, requiring the connection name to be strictly consistent with the Broker, and not using the default fallback of ordinary services. See [Reliable Delivery and Message Storage](../messaging/reliability.md) for stable storage identity and recovery requirements.

When an exception such as a missing method occurs during the first call, you should first check the third-party DLL version in the final deployment directory instead of just checking the project reference. The current Redis deployment chain has the risk of transitive dependency overwriting the version; when repairing, the actual project dependencies shall prevail, stop testing the host, redeploy the compatible version, and then verify the first connection.

Source references: [Redis](https://github.com/Zongsoft/framework/tree/main/externals/redis), [etcd](https://github.com/Zongsoft/framework/tree/main/externals/etcd), [Garnet](https://github.com/Zongsoft/framework/tree/main/externals/garnet).

## Continue Reading by Project

[Etcd](projects/etcd.md) · [Garnet](projects/garnet.md) · [Redis](projects/redis.md)
