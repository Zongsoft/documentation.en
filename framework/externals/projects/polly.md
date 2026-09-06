---
description: "Polly project's capability scope, deployment products, integration entry points and usage boundaries."
icon: plug
---

# Polly

Use retries, timeouts, circuit breaking, rate limiting and fallback integration Core execution pipeline to handle temporary failures encountered in an operation.

| Item | value |
| --- | --- |
| source code directory | `externals/polly` |
| main bag | `Zongsoft.Externals.Polly` |
| Companion theme | [Task Scheduling and Resilient Execution](../execution.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals polly]
nuget:Zongsoft.Externals.Polly
```
{% endcode %}

The pipeline builder is bound to the executor through the plugin tree, and the strategy is expressed using the corresponding features of Core. Strategies and their sequence of combinations should be decided for specific operations.

## Integration Steps

1. Confirm that the actuator actually passes through the configuration pipeline with a controlled short operation.
2. Verify the number of retries, timeout cancellation, circuit breaking recovery or rate limiting rejection item by item, and then combine the strategies.
3. Compare the final error and total time consumption with business expectations before integrating external calls.

For specific configuration, calling examples and related basic concepts, see [Task Scheduling and Resilient Execution](../execution.md).

## Project Boundaries

{% hint style="info" %}
💡 Deployment of the plugin will not make all HTTP or database calls automatically obtain the policy. Retries require operations to be replayable; when combined with Hangfire job retries, the total number of attempts and total time budget should be calculated.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Task Scheduling and Resilient Execution](../execution.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/polly)
