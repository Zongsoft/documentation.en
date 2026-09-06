---
description: "Start plugin-based applications in the .NET Host and load the plugin directory."
icon: power-off
---

# Host Integration

The goal of host integration is to connect the standard .NET Host with the Zongsoft plugin framework. The host still uses [`IHost`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.ihost) _[Source](https://source.dot.net/#Microsoft.Extensions.Hosting.Abstractions/IHost.cs)_, configuration, dependency injection and lifecycle events; the plugin framework is responsible for loading the plugin tree during the Host build process, registering the plugin assembly service, and creating the application context.

## Start the Portal

The business code of Discussions is placed in the plugin, and the process is hosted by the hosting project. The following is the actual entrance to the existing guard host. The platform branch is selected by the project compilation symbols.

The plugin framework provides two sets of entrances: `Application.Daemon(...)` and `Application.Terminal(...)`:

Source: [hosting/daemon/Program.cs](https://github.com/Zongsoft/hosting/blob/main/daemon/Program.cs#L9) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
static void Main(string[] args)
{
	#if WINDOWS
	Zongsoft.Plugins.Hosting.Application
		.Daemon("zongsoft.daemon", [.. args, "host=daemon", "site=daemon"], builder =>
		{
			builder.Services.AddWindowsService(options => options.ServiceName = builder.Environment.ApplicationName);
		}).Run();
	#elif LINUX
	Zongsoft.Plugins.Hosting.Application
		.Daemon("zongsoft.daemon", [.. args, "host=daemon", "site=daemon"], builder =>
		{
			builder.Services.AddSystemd();
		}).Run();
	#else
	Zongsoft.Plugins.Hosting.Application
		.Daemon("zongsoft.daemon", [.. args, "host=daemon", "site=daemon"])
		.Run();
	#endif
}
```
{% endcode %}

`Daemon` is suitable for background services and resident processes, and `Terminal` is suitable for terminal programs. Both will load the host configuration file, then build and initialize the Host.

## Configure Load Order

When the host starts, it will load the `.option` file according to the application name:

- `{ApplicationName}.option`
- `{ApplicationName}.{Environment}.option`
- `{ApplicationName}.{Host}.option`
- `{ApplicationName}.{Host}.{Environment}.option`
- `{ApplicationName}.{Site}.option`
- `{ApplicationName}.{Site}.{Environment}.option`

Among them, `Environment` comes from the Host environment name, and `host` and `site` come from the configuration section. The application name can be determined by Host settings, `appsettings.json`, or entry assembly name.

{% hint style="info" %}
`.option` is an XML configuration file used by Zongsoft configuration system. The plugin configuration in the plugin directory will also be added to the application configuration as a configuration source.
{% endhint %}

## Plugin Loading Process

When building a Host, the framework performs the following actions:

1. Create `PluginOptions`, determine the content root, environment name and plugin directory, and add the plugin configuration source to the application configuration.
2. Load plugin tree via `PluginTree.Get(options).Load()`.
3. Register service types in the host assembly, host reference assembly, and plugin manifest assembly.
4. Add default `System.Net.Http.HttpClient` service.
5. Register a declarative service under `/Workspace/Environment/Services`.
6. Build the Host and call `Initialize()` to initialize the application context.

If the `plugins` directory does not exist, the plugin loading will fail and a directory does not exist exception will be thrown. When deploying the host, ensure that the plugin directory matches the host application directory.

{% content-ref url="plugin-file.md" %}
[plugin-file.md](plugin-file.md)
{% endcontent-ref %}

## Service Registration

The plugin assembly service registration comes from two sources:

- Assembly scan: host assembly, host reference assembly, and assembly declared in plugin `manifest`.
- Plugin tree service node: The builtin under `/Workspace/Environment/Services` will be registered to the service collection as a singleton.

This means that ordinary services can be registered with code attributes and conventions; services that require declarative configuration can be placed in the plugin tree.

## Initialization and Lifecycle

After the Host is built, `ApplicationContext` will be initialized. The application context will parse all `IApplicationInitializer` and perform initialization; when the Host lifecycle enters Started, Stopping, or Stopped, the application context will start or stop the registered worker and trigger the corresponding event.

The recommended boundaries for plugin-based applications are:

- The host is responsible for the process, environment and a small amount of basic configuration.
- Plugins are responsible for modules, services, commands, drivers, controllers and business capabilities.
- The application context is responsible for uniformly exposing modules, services, events and lifecycle at runtime.

<details>

<summary>When should I change the host and when should I write a plugin?</summary>

If the change affects process startup, site selection, host environment, service hosting method, or deployment entry, it is usually the responsibility of the host. If the changes are to business capabilities, database drivers, commands, Web APIs, background workers, or module internal services, plugins are given priority.

</details>

{% content-ref url="../../get-started/deploy-first-plugin.md" %}
[deploy-first-plugin.md](../../get-started/deploy-first-plugin.md)
{% endcontent-ref %}

## Content Root Matches Configuration

The default plugin directory is composed of content root and `plugins`. The absolute path to the startup DLL does not automatically change the working directory, so you should launch from the deployed directory and verify that the base manifest is included. Host configuration and plugin configuration use different suffix matching rules, see [Option Configuration Files](../../references/option-files.md) for details.

The assembly reference scan only processes the entry reference that has been loaded at that time, and then processes the entry assembly and the assembly declared in the loaded plugin manifest. Don't assume that every DLL you put in the directory will automatically participate in the service scan.

The web application uses the matching `Zongsoft.Web.Application.Web(...)` entry and additionally completes the assembly of controllers, initializers and middleware, see [Web Fundamentals](../web.md).
