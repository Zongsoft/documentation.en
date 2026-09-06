---
description: "The Garnet project's capability scope, deployment products, integration portals and usage boundaries."
icon: plug
---

# Garnet

Run a Garnet server that supports the Redis protocol through a host [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs). The project assumes server hosting responsibilities, and business clients still need corresponding access adapters.

| Item | value |
| --- | --- |
| source code directory | `externals/garnet` |
| main bag | `Zongsoft.Externals.Garnet` |
| Companion theme | [Caching and Distributed Coordination](../caching.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals garnet]
nuget:Zongsoft.Externals.Garnet
```
{% endcode %}

The setting is at `/Externals/Garnet`, and the value of the named server is converted to a server option. Starting the [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) will use the corresponding port and storage directory.

## Integration Steps

1. Enable [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) after explicitly binding address, port, authentication and persistence directory.
2. Use the selected client to verify the commands and data types your app actually relies on.
3. If AOF or checkpointing is enabled, verify stops, restarts, and data recovery.

For specific configuration, calling examples and related basic concepts, see [Caching and Distributed Coordination](../caching.md).

## Project Boundaries

{% hint style="info" %}
💡 The server and host share process resources and fault boundaries. Relative directories are usually resolved relative to the adapter assembly, and `~/` is relative to the application root directory; Redis protocol compatibility does not mean that all commands and persistence behaviors are the same.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Caching and Distributed Coordination](../caching.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/garnet)
