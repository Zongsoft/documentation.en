---
description: "Ability to organize report descriptions, template resources and data loading, and identify report contracts and specific engines."
icon: file-chart-column
---

# Reporting

`Zongsoft.Reporting` provides reports, description information, resources, data models and data loading contracts. Businesses can use this to manage template sources and data supplies separately, but the final rendering, export and design functions still rely on specific reporting engines.

## What Responsibilities Are Involved in Report Generation

| link | Problem solved | Corresponding abstraction |
| --- | --- | --- |
| Identify template | What is the report name, source and description? | `IReportDescriptor` |
| Open resource | Get template content from a file or other location | Describe objects, resources and resource resolvers |
| Create report instance | Load template with specific engine | `IReport` implemented by engine |
| Provide data | Mapping report models to business queries | `IReportDataLoader`、`IReportDataModel` |
| Rendering and exporting | Generate output that can be displayed or downloaded | Specific reporting engine |

The resource resolver returns report resources and cannot be directly used as a report instance factory. There are save, render or export methods in the interface, which does not mean that each adapter has implemented the corresponding methods.

## Integration Data Service

The default `ReportDataLoader` parses the object from the service container according to the data model's `Name` and requires that the object is a data service; then uses the model's `Schema` to initiate a select query. Therefore, the report model name must correspond to the service alias registered by the business, and cannot just fill in the database table name.

First complete the registration and separate query verification of [Data Services](data/services.md), and then connect the report model. The authorization, filter and query constraints of the service are still in effect, and the business data scope should not be bypassed through the report entry.

{% hint style="warning" %}
🚨 The default loader uses `Paging.Page(1)` and will not automatically traverse all pages. Before exporting a full report, you need to clarify the paging strategy, row limit, and resource consumption, and implement a data loader suitable for the report; otherwise, you may only get the first page of data.
{% endhint %}

## Templates and Resources Lifecycle

The file description object can open the template stream, but opening the file only proves that the resource is readable, but does not prove that the template format is supported by the target engine. Template formats, fonts, images, and external data sources should be checked along with the deployment.

Resource flows should be released by a clear layer. In particular, check whether the engine will close the stream passed in by the caller when opening the template to avoid subsequent processing from continuing to read the released resources. Before caching a report instance, you should also confirm that the engine instance can be shared and used concurrently across requests.

The `Key` file description object is based on the path and runtime hash and should not be treated as a report number that is stable across processes and versions. The persistent directory should use a stable identity defined by the business itself.

## Current Implementation Scope

The current default implementation of `ReportDataLocator` does not complete data positioning. Before use, you need to check the methods you plan to call one by one, instead of arranging a complete export pipeline based on the package name only.

Deploying `Zongsoft.Reporting` does not automatically provide the designer or a working report download interface. The app also needs to select an engine, meet its running and licensing conditions, load templates, connect to data, and handle output and errors.

## Recommended Acceptance Sequence

1. Use a fixed small template to verify resource lookup and engine loading.
2. Validate fields, groupings, sorting, and data permissions with a small amount of known data.
3. Check cross-page data, Chinese fonts, time zones and numerical formats.
4. Verify the download results against the output format supported by the actual implementation, and record the number of pages and total amount of data.
5. Then test concurrency, timeout, temporary file cleanup and resource release after failure.

Spreadsheet data exchange does not necessarily require a reporting engine. Just see [Spreadsheet and Template Extensions](externals/documents.md) when importing and exporting workbooks.

Source references: [Report module](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Reporting), [Default data loading](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Reporting/src/ReportDataLoader.cs).
