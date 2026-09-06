---
description: "The capability scope, deployment products, integration entry points and usage boundaries of the OpenXml project."
icon: plug
---

# OpenXml

Provides explicit workbook and cell operations, suitable for programs that require direct control of the spreadsheet structure. The archiving process of model records can also refer to the ClosedXml project.

| Item | value |
| --- | --- |
| source code directory | `externals/openxml` |
| main bag | `Zongsoft.Externals.OpenXml` |
| Companion theme | [Spreadsheet and Template Extensions](../documents.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals openxml]
nuget:Zongsoft.Externals.OpenXml
```
{% endcode %}

Create or open a workbook directly using the SpreadsheetDocument wrapper in your project. The topic provides a short example of creating a workbook, writing to cells, and saving it.

## Integration Steps

1. Start by creating a worksheet and two cells, save and reopen to check the values.
2. Explicitly use the editable parameter when editing an existing file; the default open mode is read-only.
3. Verify the Chinese language, data type, required format and whether the file can be accessed again after release.

For specific configuration, calling examples and related basic concepts, see [Spreadsheet and Template Extensions](../documents.md).

## Project Boundaries

{% hint style="info" %}
💡 The wrapper does not launch Excel, nor does it calculate formulas. Capabilities such as formula caching and charts should be verified according to the underlying implementation; the workbook created by the caller must be released in time to close the underlying file package.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Spreadsheet and Template Extensions](../documents.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/openxml)
