---
description: "The capability scope, deployment products, integration entry points and usage boundaries of the Python project."
icon: plug
---

# Python

Python evaluation capabilities are provided through IronPython, suitable for using languages and libraries compatible with this runtime in controlled rules.

| Item | value |
| --- | --- |
| source code directory | `externals/python` |
| main bag | `Zongsoft.Externals.Python` |
| Companion theme | [Scripts and Expressions](../scripting.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals python]
nuget:Zongsoft.Externals.Python
```
{% endcode %}

Match expression evaluator by name `Python` after application initialization. The deployment must retain the required lib standard library and cannot rely on the system CPython package installation results.

## Integration Steps

1. Confirm that IronPython and the standard library are complete in the deployment directory.
2. Use the variable transfer method in the topic to verify simple expressions, and then verify the libraries actually imported by the application.
3. Establish clear execution boundaries for concurrent calls, input and output, and global state.

For specific configuration, calling examples and related basic concepts, see [Scripts and Expressions](../scripting.md).

## Project Boundaries

{% hint style="info" %}
💡 Currently implements a reuse engine and switches runtime IO during calls. The independent variable dictionary does not guarantee thread isolation; it is up to the application to serialize or use a separate process when needed, and the evaluator cannot be used as a security sandbox.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Scripts and Expressions](../scripting.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/python)
