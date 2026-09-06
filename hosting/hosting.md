---
description: "Choose a terminal, backend, or web host, and understand application identity, configuration directories, and plugin lifecycle."
icon: server
---

# Host Overview

The host provides process entry, configuration, service container, logs and start-stop lifecycle, and the plugin provides business capabilities. Keeping the host entry simple allows the same business component to be reused in different deployment solutions; the prerequisite for reuse is that the target host has the interaction or protocol capabilities it needs.

## Choose a Host

| Type | main entrance | Suitable for the scene | read |
| --- | --- | --- | --- |
| Terminal | Interactive commands | Development and debugging, manual diagnostics, and reproduction of background behavior | [Terminal Host](terminal.md) |
| Daemon | long term [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) | Consume messages, execute jobs, and system services | [Background Service Host](daemon.md) |
| Web | HTTP pipeline | Controller, interface, gateway | [Web Host](web.md) |

The terminal can simulate background business, but the web controller and middleware still require a web host; plugins that use terminal input should not be started unconditionally in non-interactive services.

## How the Current Source Code Is Built

`Directory.Build.props` of the hosting root currently sets `net10.0`, and enables `ZongsoftFrameworkPathReferenced`, referencing the class library from the corresponding output of the adjacent framework. Before building the host, you need to prepare a framework product that matches the target framework and compilation configuration.

The framework library itself may support .NET 8, 9, 10, this does not mean that every host project and all dependencies use the same target. When switching versions, you must also check the project, central package version and release script, see [Environmental preparation](../get-started/prerequisites.md).

## Running Directory and Identity

Taking the actual deployment list of hosting/web/default as an example, Main.plugin is placed in plugins, and other configurations are selected by variables such as scheme, site, environment, etc.; see [Deploy Your First Plugin](../get-started/deploy-first-plugin.md) for the deployment location of Discussions.

Source: [hosting/web/default/.deploy](https://github.com/Zongsoft/hosting/blob/main/web/default/.deploy#L7) (excerpt; see source for context).

{% code title=".deploy" %}
```ini
#@import ../web.deploy

[plugins]
nuget:Zongsoft.Plugins/plugins/Main.plugin
```
{% endcode %}

The published entry point is Zongsoft.Hosting.Web.dll, and its dependency list and runtime configuration are generated with dotnet publish. Plugins and.option files are completed by the deployment manifest, and the deployment cannot be deemed complete simply by the existence of the assembly file.

The executable file name, runtime application name, `host` and `site` are not necessarily the same. The application name affects configuration and upgrade matching, and `host/site` participates in deployment and running configuration selection. The current application name of the terminal is `zongsoft.terminal`, but the entry assembly is `Zongsoft.Hosting.Terminal`.

The working directory, application directory and content root path should also be understood separately, see [basic concepts](../overview/concepts.md#host). When starting a service from a different directory, incorrect relative paths can cause configuration, database, or plugin resources to resolve to different locations.

## Verification of a Complete Boot

First confirm that the process is started correctly, then check plugin loading, builtin and service registration, and finally verify the business call. Some providers delay establishing a connection until first access, exposing endpoint, permissions, or DLL version errors.

When stopping, the host notifies the [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) to stop and release the resources it owns. Business workers should stop receiving new tasks, pass cancellations, and wait for necessary in-flight operations; they should not rely on force exit to replace normal lifecycle design.

Next step: [Deploy a Host](deployment.md), [Container Environments](containerization.md), [Plugin hosting integration](../framework/plugins/hosting.md).

Source references: [hosting repository](https://github.com/Zongsoft/hosting).
