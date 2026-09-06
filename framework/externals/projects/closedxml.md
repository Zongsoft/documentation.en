---
description: "ClosedXml project's capability scope, deployment products, integration entry and usage boundaries."
icon: plug
---

# ClosedXml

Business model integration Excel data archiving, extraction and template services are suitable for processes such as exporting business records, manually filling them in and then importing them.

| Item | value |
| --- | --- |
| source code directory | `externals/closedxml` |
| main bag | `Zongsoft.Externals.ClosedXml` |
| Companion theme | [Spreadsheet and Template Extensions](../documents.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals closedxml]
nuget:Zongsoft.Externals.ClosedXml
```
{% endcode %}

Match services in `Spreadsheet` format via [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core)'s archive or template contract. The actual data service first provides its own descriptor, allowing the primary key, length and semantics in the mapping to enter the archiving process.

## Integration Steps

1. Export a small number of model records along the topic's generator example, checking column names, display formats, and field validation.
2. After manually modifying or appending records, use the same model to describe the extraction and compare the actual field values.
3. Cover empty tables, null values, long integers, dates and field changes, and then integrate business verification and writing.

For specific configuration, calling examples and related basic concepts, see [Spreadsheet and Template Extensions](../documents.md).

## Project Boundaries

{% hint style="info" %}
💡 The import location is an Excel Table. The spreadsheet name is `__{model.QualifiedName}__`. The worksheet name or ordinary area cannot replace it. When there are no summary rows, additional records will be extracted downwards within the spreadsheet column range, and comments should be placed outside the column band.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Spreadsheet and Template Extensions](../documents.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/closedxml)
