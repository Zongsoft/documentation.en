---
description: "Opc project's capability scope, deployment products, integration entry and usage boundaries."
icon: plug
---

# Opc

Provides OPC UA client, server and read-write subscription adaptation for connecting to industrial data services. Node identification, numerical quality and sampling time are all business data.

| Item | value |
| --- | --- |
| source code directory | `externals/opc` |
| main bag | `Zongsoft.Externals.Opc` |
| Companion theme | [OPC UA Device Protocol](../integration.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals opc]
nuget:Zongsoft.Externals.Opc
```
{% endcode %}

Endpoints, application certificates, trust directories, user identities, and session lifecycles are organized by the application. The project includes supporting local server/client examples, which can be used to verify reading and subscription first.

## Integration Steps

1. Prepare the local server and client according to the project example, and establish certificate trust and session.
2. Browse and read known nodes first, then verify change subscriptions, status codes and source timestamps.
3. After verifying the disconnection, server restart and exit release, then design the write control of the real device.

For specific configuration, calling examples and related basic concepts, see [OPC UA Device Protocol](../integration.md).

## Project Boundaries

{% hint style="info" %}
💡 Apply certificates and user permissions to solve different problems. Plugin deployment does not automatically complete trust configuration, and reading a value does not mean that the quality status is normal; the trust method in the example cannot be directly used as a formal device deployment strategy.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [OPC UA Device Protocol](../integration.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/opc)
