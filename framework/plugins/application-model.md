---
description: "Understand hosts, application contexts, modules, and services in plugin-based applications."
icon: sitemap
---

# Plugin Application Model

Plugin-based applications consist of host, plugin framework, plugin tree, application context and [application modules](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationModule.cs). The host is responsible for "how the process is started", and the plugin is responsible for "how the capabilities enter the application".

## Hosts

The host is responsible for starting the process and establishing the running environment. It usually only contains `Program.cs`, project files, deployment scripts and basic configuration. Business capabilities should be put into plugins as much as possible instead of being solidified in the host.

The framework provides two types of common startup entries:

- `Application.Daemon(...)`: used for background services, [workers](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) and resident processes.
- `Application.Terminal(...)`: Used for command lines, terminal tools, and interactive programs.

These entries will create the .NET Host, load the host `.option` configuration, register the plugin configuration source, load the plugin tree and initialize the application context.

## Application Context

application context represents the currently running application instance, corresponding to [`IApplicationContext`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationContext.cs) in the core library. It contains the application name, version, environment, configuration, module collection, service provider, event manager, [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) and runtime properties.

The `/Application` interface in the web host returns basic information about the current application context.

## Module

The module corresponds to [`IApplicationModule`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationModule.cs) in the core library. A module has a name, version, assembly, service container and property collection; modules inheriting `ApplicationModule<TEvents>` can also have their own event registry.

Module names are important: data access, service registration, log naming, and module isolation all use module names as boundaries. For example, by default, data service will search the module name according to [`ApplicationModuleAttribute`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationModuleAttribute.cs) of the assembly where the type is located, and then obtain the corresponding data accessor.

The `/Modules` interface in the web host can be used to view currently loaded module information.

## Service

Plugins can register services with applications. When the host starts, it will scan the host assembly and the assembly in the plugin list, and add types that comply with the service registration convention to the service container; the `/Workspace/Environment/Services` node in the plugin tree can also register declarative builtin as a singleton service.

Modules can also have their own service domains. [`ApplicationModule`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationModule.cs) will create a module-level service provider based on the application service container to isolate or name the internal services of the module.

## Workbench and Startup Node

The default plugin will mount commonly used runtime objects under `/Workbench`, for example:

- `/Workbench/Modules`: Current [application module](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationModule.cs) collection.
- `/Workbench/Services`: Current application service container.
- `/Workbench/Events`: Global event manager.
- `/Workbench/Configuration/ConnectionSettings/Drivers`: Connection settings driver collection.
- `/Workbench/Startup`: The set of [workers](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) that need to be loaded or run at startup.

This allows plugins to discover capabilities via a stable path between plugins, rather than directly referencing each other's implementation types.

## From Business Boundaries to Running Instances

There is no automatic one-to-one correspondence between plugins and modules. The module object is defined by the application and added to the application through `/Workbench/Modules`; [`ApplicationModuleAttribute`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationModuleAttribute.cs) on the assembly participates in the module ownership of the type. Both should use the same module name. See [The first business plugin](../../get-started/first-business-plugin.md) for the complete declaration and call.

The module container first parses module services, and then falls back to apply shared services, which is suitable for local implementation of business and collaboration with public infrastructure. It does not automatically create independent processes, databases or request scopes, see [basic concepts](../../overview/concepts.md#module-service-provider) for details.

## Starting and Stopping Responsibilities

During host building, the plugin tree is loaded first and the service is registered, and then the application context is initialized. When the [workbench](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/IWorkbenchBase.cs) is first built, it organizes its child nodes and puts the Startup subtree behind it; the workbench is opened and closed when the host enters the startup and stop phases.

Object construction, application initialization, and ongoing work are distinct phases. Components that need to maintain message subscriptions or periodic tasks should use [Workers](../core/components/worker.md) to manage startup, cancellation, and release; do not rely on the constructor to start background tasks that cannot be stopped.

{% hint style="warning" %}
🚨 Failure to build the plugin node does not mean that all external operations that have occurred can be rolled back. Components should control construction side effects and implement clear stopping and cleanup paths.
{% endhint %}

Source code positioning: [application context](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/PluginApplicationContext.cs), [module container](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationModule.cs).
