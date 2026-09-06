---
description: "The Lua project's capability scope, deployment products, integration entry points, and usage boundaries."
icon: plug
---

# Lua

Provides Lua expression evaluation through NLua and KeraLua. The business chooses Lua through the common evaluator contract, and script syntax and return values are still determined by Lua.

| Item | value |
| --- | --- |
| source code directory | `externals/lua` |
| main bag | `Zongsoft.Externals.Lua` |
| Companion theme | [Scripts and Expressions](../scripting.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals lua]
nuget:Zongsoft.Externals.Lua
```
{% endcode %}

Match expression evaluator by name `Lua` after application initialization. An example calculation would use `return x + y`, with the variable passed in its own dictionary for each call.

## Integration Steps

1. Deploy native runtime libraries that match the operating system and architecture.
2. Use the Lua name and syntax according to the public calling method of the script topic, and verify the input and return value types.
3. Cover multiple return values, null values, syntax errors, and consecutive calls before integrating business rules.

For specific configuration, calling examples and related basic concepts, see [Scripts and Expressions](../scripting.md).

## Project Boundaries

{% hint style="info" %}
💡 Each Evaluate creates and releases Lua state; do not assume that any script has time limits or security isolation based on this. Sharing Global is only suitable for configuring stable content during the startup phase.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Scripts and Expressions](../scripting.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/lua)
