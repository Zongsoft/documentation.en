---
description: "Find extension packages, configuration entry points, and topic guides by their externals source projects."
icon: folders
---

# Browse by Project

If you know the project name or are ready to deploy a package, go directly to its project page. [Extension Plugins](../../externals.md) organizes the same capabilities by need: caching, scripting, scheduling, spreadsheets, cloud services, and device integration. Each project page collects an implementation’s deployment requirements and limitations and links to its full usage guide.

## Project Index

The following includes 14 external extension projects; Web, Gateway, and optional storage artifacts in the same directory are classified in the corresponding projects.

| Project | Main Capabilities | Related Topic |
| --- | --- | --- |
| [Aliyun (aliyun)](aliyun.md) | Integration of Alibaba Cloud OSS, messaging, SMS and voice, mobile push and other capabilities | [Cloud Services and Callbacks](../cloud.md) |
| [Amazon (amazon)](amazon.md) | Integrate Amazon S3 and compatible object storage through the framework file system | [Cloud Services and Callbacks](../cloud.md) |
| [ClosedXml (closedxml)](closedxml.md) | Connect business models to Excel archiving, extraction, and template services; useful for exporting records or importing completed forms | [Spreadsheet and Template Extensions](../documents.md) |
| [Etcd (etcd)](etcd.md) | Provides basic key-value operations, sequence numbers and lease lock, suitable for scenarios that require atomic sequence number allocation or coordinated access to shared resources. | [Caching and Distributed Coordination](../caching.md) |
| [Garnet (garnet)](garnet.md) | Run the Garnet server that supports the Redis protocol through the host worker | [Caching and Distributed Coordination](../caching.md) |
| [Hangfire (hangfire)](hangfire.md) | Connect persistent background jobs with framework scheduling contracts, suitable for delayed execution and periodic tasks | [Task Scheduling and Resilient Execution](../execution.md) |
| [Lua (lua)](lua.md) | Lua expression evaluation via NLua and KeraLua | [Scripts and Expressions](../scripting.md) |
| [Opc (opc)](opc.md) | Provides OPC UA client, server and read-write subscription adaptation for connecting industrial data services | [OPC UA Device Protocol](../integration.md) |
| [OpenXml (openxml)](openxml.md) | Provides explicit workbook and cell operations, suitable for programs that need to directly control the spreadsheet structure | [Spreadsheet and Template Extensions](../documents.md) |
| [Polly (polly)](polly.md) | Add retry, timeout, circuit breaking, rate limiting, and fallback to the Core execution pipeline to handle transient operation failures | [Task Scheduling and Resilient Execution](../execution.md) |
| [Python (python)](python.md) | Provides Python evaluation capabilities through IronPython, suitable for using languages and libraries compatible with this runtime in controlled rules | [Scripts and Expressions](../scripting.md) |
| [Redis (redis)](redis.md) | Integrate Redis with caching, sequences, distributed locks, message streams, configuration, and reliable message storage | [Caching and Distributed Coordination](../caching.md) |
| [Scriban (scriban)](scriban.md) | Evaluate Scriban pure-script expressions to integrate small rules into business plugins by name and configuration | [Scripts and Expressions](../scripting.md) |
| [Wechat (wechat)](wechat.md) | Provide WeChat accounts, public platforms, third-party platforms and payment adaptation | [Cloud Services and Callbacks](../cloud.md) |

## Combine the Topic and Project Guides

If the task is "Export business records", start by comparing ClosedXml with OpenXml from the spreadsheet topic; if ClosedXml has been selected, check the package, service matching and spreadsheet naming from the project page, and then jump to the export/import example.

{% hint style="info" %}
💡 Source directories, primary NuGet packages, and runtime capabilities do not have a one-to-one correspondence. A project may also have independent Web/Gateway packages, or may rely on native libraries or external services; select deployment artifacts according to the actual roles listed on the project page.
{% endhint %}
