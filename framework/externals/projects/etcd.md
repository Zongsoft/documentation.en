---
description: "The etcd project's capability scope, deployment products, integration entry points and usage boundaries."
icon: plug
---

# Etcd

Provides basic key-value operations, sequence numbers and lease lock, suitable for scenarios that require atomic sequence number allocation or coordinated access to shared resources.

| Item | value |
| --- | --- |
| source code directory | `externals/etcd` |
| main bag | `Zongsoft.Externals.Etcd` |
| Companion theme | [Caching and Distributed Coordination](../caching.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals etcd]
nuget:Zongsoft.Externals.Etcd
```
{% endcode %}

The connection settings are at `/Externals/Etcd/ConnectionSettings` and the drive key is `etcd`. Serial numbers and locks are processed through corresponding provider integration; when multiple implementations coexist, they must be explicitly selected at the application composition layer.

## Integration Steps

1. Configure the test cluster and independent key prefix, first confirm the connection and target key space.
2. Verify the first allocation, repeated increment and expiration strategies of serial numbers; the first result is seed + interval.
3. Verify that lock acquisition, competition, lease expiration and renewal failure have failed, and then let the resource end use fencing token to reject the old holder.

For specific configuration, calling examples and related basic concepts, see [Caching and Distributed Coordination](../caching.md).

## Project Boundaries

{% hint style="info" %}
💡 The current provider does not have an Etcd service alias and cannot copy the `name@Redis` call of Redis. It is also not an alternative implementation of the public distributed cache interface; the connection name does not automatically become the key namespace.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Caching and Distributed Coordination](../caching.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/etcd)
