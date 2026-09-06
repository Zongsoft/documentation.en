---
description: "Redis project's capability scope, deployment products, integration entrance and usage boundaries."
icon: plug
---

# Redis

Integrate Redis cache, sequence number, distributed lock, message flow, configuration and reliable message storage. Each capability shares the same infrastructure, but has different contract and consistency boundaries.

| Item | value |
| --- | --- |
| source code directory | `externals/redis` |
| main bag | `Zongsoft.Externals.Redis` |
| Companion theme | [Caching and Distributed Coordination](../caching.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals redis]
nuget:Zongsoft.Externals.Redis
```
{% endcode %}

The normal connection is located at `/Externals/Redis/ConnectionSettings`, and the driver and provider alias is `Redis`. For example, "Connection name@Redis" obtains an instance from the Redis provider based on the specified connection name.

## Integration Steps

1. Configure independent test connections per cache topic, using unique keys to verify reads, writes, and cleanups.
2. Verify the actual connection selection, database number and key prefix, and then verify the sequence number or lock according to the required capabilities.
3. When message storage is required, configure the connection and stable storage identity strictly with the same name as the Broker according to the reliability topic.

For specific configuration, calling examples and related basic concepts, see [Caching and Distributed Coordination](../caching.md).

## Project Boundaries

{% hint style="info" %}
💡 Ordinary providers may fallback to default connections, while message storage factories strictly match the Broker name. Shared instances should not release or modify the namespace on request; keyspace notifications are not replayed when disconnected and cannot replace reliable event logs.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Caching and Distributed Coordination](../caching.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/redis)
