---
description: "Select third-party adapters according to application scenarios, and understand the relationship between public contracts, plugin deployment, and external operating environments."
icon: plug-circle-bolt
---

# Extension Plugins

![Different external capabilities are connected to the framework through adaptation interfaces](../.gitbook/assets/zongsoft-externals-cover.png)

External extensions integrate third-party runtimes, infrastructure or cloud services with Zongsoft. The business module relies on the framework contract as much as possible, and the application composition layer is responsible for selecting the provider; the actual protocol, resources and fault behavior are still determined by the specific implementation.

## Read on Demand

| demand | Expand | Topics |
| --- | --- | --- |
| Cache, serial number, distributed lock, in-process cache server | Redis、etcd、Garnet | [Caching and Distributed Coordination](externals/caching.md) |
| Rule calculations, language scripts, text expressions | Lua、Python、Scriban | [Scripts and Expressions](externals/scripting.md) |
| Background jobs, retries, timeouts and circuit breaking | Hangfire、Polly | [Task Scheduling and Resilient Execution](externals/execution.md) |
| Workbook exchange, template and cell operations | ClosedXml、OpenXml | [Spreadsheet and Template Extensions](externals/documents.md) |
| Object storage, SMS push, WeChat and callback | Amazon、Aliyun、Wechat | [Cloud Services and Callbacks](externals/cloud.md) |
| Industrial communication and equipment data integration | Opc | [OPC UA Device Protocol](externals/integration.md) |

## Browse by Project

If the project name has been determined, you can enter the independent pages of 14 projects including Aliyun, Amazon, Redis, etcd, and Hangfire from [Project index](externals/projects/README.md) to view packages, configuration entries, and implementation restrictions.

## Common Path of Integration

First determine the required public capabilities, and then deploy the implementation package and its accompanying files. After the plugin is loaded, check the configuration, service registration or plugin tree node, and finally perform a controlled call. Just seeing the name in the plugin list does not prove that the lazy link, native library, or third-party dependency version is correct.

{% code title=".deploy (taking Scriban as an example)" %}
```ini
[plugins zongsoft externals scriban]
nuget:Zongsoft.Externals.Scriban
```
{% endcode %}

This content should be merged into the existing host's deployment manifest. See [Deployment Tool](../tools/deployer.md) for deployment parameters and runtime directory; see [The first business plugin](../get-started/first-business-plugin.md) for how to enable business plugins through configuration selection.

## Three Common Integration Forms

**service provider**supplies cache, lock and other instances by connection name;**Matchable services**finds evaluators and archivers by name or format;**plugin tree builtin** mounts the file system, worker and handler collections through extended paths. They are both plugin extensions, but they are obtained in different ways.

Don't see a type in the package and infer that it must be registered in the container; don't apply Redis's alias rules to other providers. See [Service Resolution and Ownership](core/services/locating.md) for detailed differences.

## Run and Version Boundaries

External service addresses, credentials, databases, buckets, certificates, and licenses are provided by the deployment environment. Package versions should be checked against a combination of host and third-party dependencies, especially if multiple deployment manifests copy different versions of DLLs to the same directory.

The framework adapter does not automatically create all cloud resources or override their permission configuration. The script plugin does not automatically become a security sandbox, and the Dashboard does not automatically become a complete operation and maintenance authority system. Each topic describes the capabilities that have been achieved, the responsibilities that need to be applied and the verification path.

Source references: [external extension directory](https://github.com/Zongsoft/framework/tree/main/externals).
