---
description: "The Scriban project's capability scope, deployment products, integration portals and usage boundaries."
icon: plug
---

# Scriban

Provides Scriban pure script expression evaluation, suitable for integrating small-scale rules by name and configuring business plugins. A complete getting started example demonstrates service matching using this project.

| Item | value |
| --- | --- |
| source code directory | `externals/scriban` |
| main bag | `Zongsoft.Externals.Scriban` |
| Companion theme | [Scripts and Expressions](../scripting.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals scriban]
nuget:Zongsoft.Externals.Scriban
```
{% endcode %}

Match expression evaluator by name `Scriban` after application initialization. The expression uses pure script, such as `x + y`, without directly adding template delimiters.

## Integration Steps

1. First complete the first business plugin tutorial and confirm that the manifest, service registration and commands are all effective.
2. Pass a dictionary of independent variables for each evaluation and verify that numeric or text results are expected.
3. Save rule versions and test inputs to cover missing fields, syntax errors, and illegal results.

For specific configuration, calling examples and related basic concepts, see [Scripts and Expressions](../scripting.md).

## Project Boundaries

{% hint style="info" %}
💡 The current implementation reads variables.Count directly; an empty dictionary should be passed in even if there are no variables. Establishing a context each time does not mean that any script can be security executed or is guaranteed to stop within a specified time.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Scripts and Expressions](../scripting.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/scriban)
