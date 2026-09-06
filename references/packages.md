---
description: "Zongsoft common package, module and source code directory index."
icon: boxes-stacked
---

# Package and Module Index

This page is used to quickly find common NuGet packages, source code directories, and uses.

## Core Framework

| package | source code directory | Purpose |
| --- | --- | --- |
| [`Zongsoft.Core`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) | `framework/Zongsoft.Core` | Core abstraction and basic class library |
| [`Zongsoft.Plugins`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Plugins) | `framework/Zongsoft.Plugins` | plugin framework |
| [`Zongsoft.Plugins.Web`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Plugins.Web) | `framework/Zongsoft.Plugins.Web` | Web plugin architecture support |
| [`Zongsoft.Data`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data) | `framework/Zongsoft.Data` | data engine |
| [`Zongsoft.Web`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Web) | `framework/Zongsoft.Web` | Web basic capabilities |
| `Zongsoft.Security` | `framework/Zongsoft.Security` | security capability |
| `Zongsoft.Diagnostics` | `framework/Zongsoft.Diagnostics` | diagnostic capabilities |

## Data Driven

| package | database |
| --- | --- |
| `Zongsoft.Data.MsSql` | SQL Server |
| `Zongsoft.Data.MySql` | MySQL / MariaDB |
| `Zongsoft.Data.SQLite` | SQLite |
| `Zongsoft.Data.DuckDB` | DuckDB |
| `Zongsoft.Data.PostgreSql` | PostgreSQL |
| `Zongsoft.Data.Influx` | InfluxDB |
| `Zongsoft.Data.TDengine` | TDengine |
| `Zongsoft.Data.ClickHouse` | ClickHouse |

## Message Queues

| package | middleware |
| --- | --- |
| `Zongsoft.Messaging.Kafka` | Kafka |
| `Zongsoft.Messaging.RabbitMQ` | RabbitMQ |
| `Zongsoft.Messaging.Mqtt` | MQTT |
| `Zongsoft.Messaging.ZeroMQ` | ZeroMQ |

## Tools

| package | Commands |
| --- | --- |
| `Zongsoft.Tools.Deployer` | `dotnet deploy` |
| `Zongsoft.Tools.Packager` | `dotnet-pack` |
| `Zongsoft.Tools.Upgrader` | `dotnet-upgrade` |

## Application Capabilities and Supporting Components

| package or component | Guide | Deployment concerns |
| --- | --- | --- |
| `Zongsoft.Commands` | [Commands](../framework/core/components/commands.md) | A collection of common commands, including scheduling commands |
| `Zongsoft.Web.OpenApi`、`Zongsoft.Web.Grpc` | [protocol integration](../framework/web/protocols.md) | Documentation/Debug Portal and gRPC Routing |
| `Zongsoft.Security.Web`、`Zongsoft.Security.Captcha` | [Security](../framework/security.md) | The identity interface and verification code are configured separately |
| `Zongsoft.Diagnostics.Protocols.Client`、`Zongsoft.Diagnostics.Protocols.Server` | [OTLP](../framework/diagnostics/otlp.md) | Protocol type/client and receiving handler |
| `Zongsoft.Intelligences`、`Zongsoft.Intelligences.Web` | [AI Integration](../framework/intelligences.md) | Assistant connection, model serving and session isolation |
| `Zongsoft.Learning` | [Machine Learning](../framework/learning.md) | Current training pipeline implementation limitations |
| `Zongsoft.Net` | [Network Communication](../framework/net.md) | Packagers, handlers and connection lifecycle |
| `Zongsoft.Hardwares` | [Hardware Information](../framework/hardwares.md) | Platform permissions and image stability |
| `Zongsoft.Reporting` | [Reporting](../framework/reporting.md) | Requires specific engines, templates and data loading |
| `Zongsoft.Messaging.Storages.Data` | [Message storage](../framework/messaging/reliability.md) | Database tables, mappings, SQL, and stable partitions |

## External Extension

| package | Guide |
| --- | --- |
| `Zongsoft.Externals.Redis`、`Zongsoft.Externals.Garnet`、`Zongsoft.Externals.Etcd` | [Caching and Distributed Coordination](../framework/externals/caching.md) |
| `Zongsoft.Externals.Lua`、`Zongsoft.Externals.Python`、`Zongsoft.Externals.Scriban` | [Scripts and Expressions](../framework/externals/scripting.md) |
| `Zongsoft.Externals.Hangfire`、`Zongsoft.Externals.Polly` | [Task Scheduling and Resilient Execution](../framework/externals/execution.md) |
| `Zongsoft.Externals.Hangfire.Storages.Redis`、`Zongsoft.Externals.Hangfire.Web` | [Hangfire’s deployment roles](../framework/externals/execution.md) |
| `Zongsoft.Externals.ClosedXml`、`Zongsoft.Externals.OpenXml` | [spreadsheets and templates](../framework/externals/documents.md) |
| `Zongsoft.Externals.Amazon`、`Zongsoft.Externals.Aliyun`、`Zongsoft.Externals.Wechat` | [cloud service](../framework/externals/cloud.md) |
| `Zongsoft.Externals.Aliyun.Gateway`、`Zongsoft.Externals.Wechat.Gateway` | [callback gateway](../framework/externals/cloud.md) |
| `Zongsoft.Externals.Opc` | [OPC UA Device Protocol](../framework/externals/integration.md) |

## Automatic Upgrades

| components | Deployment location | Guide |
| --- | --- | --- |
| `Zongsoft.Upgrading.Upgrader` | Directory of application plugins to be upgraded | [upgrade integration](../framework/upgrading/workflow.md) |
| `Zongsoft.Upgrading.Deployer` | Applied `.deployer` directory | [process handover](../framework/upgrading.md) |
| `Zongsoft.Upgrading.Web` | Release Management Web Host | [Release management](../framework/upgrading/workflow.md) |
| `Zongsoft.Tools.Upgrader` | Build/release tool environment | [Upgrade packager](../tools/upgrader.md) |

## How to Use Indexes

This table is organized by project and product naming in local source code, and is not a guarantee that all versions have been released. Check the package content and target framework of the target version when installing; new capabilities in the source code master branch may not have entered the package version you are using.

The same package may carry multiple plugin variants, and Web, daemon, and gateway may also be divided into independent projects. The dependency relationship is based on the target version of `.plugin`, `.deploy` and the project file. The loading order cannot be inferred just by the package name prefix.
