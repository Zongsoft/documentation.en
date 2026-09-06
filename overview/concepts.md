---
description: "Distinguish between host, plugin, module, service, provider, builtin and four types of metadata files."
icon: shapes
---

# Core Concepts and Collaboration

This page takes "Discussions forum module provides topic query interface" as an example to illustrate several similar terms in the framework. Distinguish these concepts first, and then read the configuration and code, it will be easier to determine in which link a problem occurs.

## Host and Content Root <a id="host"></a>

**host**is the process entry for starting applications, such as terminals, background services and web applications.**content root** is the base directory for applications to find configurations, plugins, etc. Starting a DLL using an absolute path will not automatically switch the current working directory to the directory where the DLL is located.

Therefore, after deployment, you should enter the runtime directory and then start, or specify the content root through the host configuration. When `plugins/` cannot be found, check the content root first and do not change the business assembly immediately.

## Plugins and Assembly <a id="plugin"></a>

**assembly**contains executable .NET types.**plugin** is described by the `.plugin` manifest, which declares dependencies, assembly and extension contributions; a plugin can contain multiple assemblies, or it can only contribute assembly configurations.

Project references let the compiler see types; plugin manifests let the runtime know what to load; and deployment files let the necessary files appear in the correct location. These three links are not substitutes for each other. For example, the Discussions.Web project can be compiled, but the Zongsoft.Discussions.Web.plugin is not deployed, and its controller is still not loaded according to the manifest during runtime.

## Plugin Tree and Builtin <a id="plugin-tree"></a>

**plugin tree** is the path structure of the extension point at runtime. A plugin can append builtin to an existing path without modifying the path owner's source code. For example, Discussions adds the site validator to the data validator extension point, and the data driver adds the driver to `/Workbench/Data/Drivers`.

**builtin** is a buildable definition on the plugin tree. The builder determines how to create the object, and the resolver interprets expressions in properties. The following real-life listing creates the Discussions data validator and hooks it to the data extension point.

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L31) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<extension path="/Workbench/Data/Validators">
	<object name="Discussions" type="Zongsoft.Discussions.Data.DataValidator, Zongsoft.Discussions" />
</extension>
```
{% endcode %}

Builtin names are tree node names and do not automatically become service aliases in DI. The file directory is not the plugin tree path either. See [Builtins and Services](../framework/plugins/builtins-and-services.md) for detailed syntax.

## Modules, Services and Providers <a id="module-service-provider"></a>

**module** expresses the business boundaries in the application, such as Discussions. A module can be provided by several plugins; plugin loading does not automatically create business modules for each plugin. The application needs to define a module and mount the module object to `/Workbench/Modules`.

**service**is an object that provides certain operations through a contract.**service container** is responsible for parsing these objects by type, alias, or matching parameters. The module container first searches for module services, and then falls back to apply shared services; its lifecycle is not equal to the Web request lifecycle.

**provider** solves another type of problem: an implementation needs to provision multiple instances by name. For example, the Redis provider can return different caches by connection name. At this time, select provider first, and then get the named instance.

| Name | Example | decide what |
| --- | --- | --- |
| Plugin name | `Zongsoft.Externals.Redis` | Which plugin does the manifest dependency point to? |
| module name | `Discussions` | Module service resolution domain |
| provider name | `Redis` | Which implementation creates or finds instances |
| connection/instance name | Defined by actual environment options | Which set of connection parameters to use |
| builtin name | `Discussions` validator node | The location of the object in the plugin tree |

{% hint style="warning" %}
🚨 The provider in the service resolution expression is not the same concept as the module container specified by the plugin expression, and the names cannot be interchanged. See [Service Resolution and Ownership](../framework/core/services/locating.md) for complete rules.
{% endhint %}

## Configuration, Mapping and Deployment <a id="metadata"></a>

| File | Answered questions | reader |
| --- | --- | --- |
| `.plugin` | Which assemblies are loaded? Where to contribute objects? | plugin framework |
| `.option` | What connections, parameters and run options are used? | Configure provider |
| `.mapping` | How do business entities correspond to data sources? | data engine metadata loader |
| `.deploy` | Which files or packages are copied where? | Deployment tools |

The list of Discussions plugins can be kept stable with different connection configurations used for test and production environments. The deployment plan determines whether Redis and MySQL are included, and the run configuration determines which deployed implementation and connection to use. Names in the configuration must form consistent relationships with registrations, connection items, and business calls.

`.option` will not be loaded if you put it into the directory arbitrarily. The plugin configuration is associated with the file name of the loaded manifest, see [Option Configuration Files](../references/option-files.md) for details.

## Workers, Initializers and Ordinary Services <a id="lifetime"></a>

Initializers are used for configuration or assembly in the initialization phase of an application; workers are used for tasks that need to be started, run continuously, and stopped when shut down; ordinary services are used to perform an operation on a call basis. Writing "subscribe to messages and keep listening" in the constructor will make object assembly, external connections and failure handling entangled, which should usually be managed by workers.

Common types registered through service attribute scanning default to singleton. The creation and release of shared instances is determined by registration and provider contracts. When you need to call independent state each time, you should clarify the scope or use a factory, and do not deduce "new each time" from "named" and "modular".

## Applicable Boundaries of Plugin Architecture

Plugin architecture is suitable for splitting by business capabilities, combining infrastructure by environment, and allowing multiple hosts to reuse services. It also requires the team to maintain common contracts, manifests, configurations, and deployment compatibility; a very small standalone program may not necessarily need all the mechanisms.

{% hint style="warning" %}
🚨 Plugins are executed with host permissions in the host process. Module boundaries, plugin directories, and service names do not constitute a security sandbox; configuration change notifications do not mean that the assembly can be security hot-replaced. After replacing the DLL, you should verify it according to the application's stop, deploy, and start process.
{% endhint %}

Continue reading: [Design Principles](conception.md), [Plugin Application Model](../framework/plugins/application-model.md), [Glossary](../references/glossary.md).
