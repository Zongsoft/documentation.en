---
description: "The Hangfire project's capability scope, deployment products, integration entrance and usage boundaries."
icon: plug
---

# Hangfire

Connect persistent background jobs with framework scheduling contracts, suitable for delayed execution and periodic tasks. Job storage, execution process, and management interface respectively assume different responsibilities.

| Item | value |
| --- | --- |
| source code directory | `externals/hangfire` |
| main bag | `Zongsoft.Externals.Hangfire` |
| Companion theme | [Task Scheduling and Resilient Execution](../execution.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals hangfire]
nuget:Zongsoft.Externals.Hangfire
```
{% endcode %}

The business handler is mounted to `/Workbench/Scheduler/Handlers`. The delay or periodic scheduler is provided by the container, and the daemon plugin variant is responsible for the background server.

Optional products include `Zongsoft.Externals.Hangfire.Storages.Redis` and `Zongsoft.Externals.Hangfire.Web`. Redis job storage uses Hangfire connection; Web Dashboard is not responsible for performing tasks on behalf of the background server.

## Integration Steps

1. Prepare the job store and verify the daemon variants required to deploy the site.
2. Register a stable handler name, schedule a short job according to the topic example, and observe the actual execution.
3. Re-validation cycle time zone, repeated execution, failure retry and shutdown recovery.

For specific configuration, calling examples and related basic concepts, see [Task Scheduling and Resilient Execution](../execution.md).

## Project Boundaries

{% hint style="info" %}
💡 Scheduling returns task identifiers, not business results. Persistent task parameters should use serializable business data; when changing handler names or parameter structures, old tasks in storage need to be taken into account.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Task Scheduling and Resilient Execution](../execution.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/hangfire)
