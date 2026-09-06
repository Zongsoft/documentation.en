---
description: "Understand the application context, service registration, service resolution and plugin service discovery of Zongsoft.Services."
icon: server
---

# Zongsoft.Services

`Zongsoft.Services` is the service model of Zongsoft runtime. It connects standard .NET dependency injection, application context, [application modules](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationModule.cs), and plugin trees, allowing hosts, plugin assemblies, and declarative builtins to register capabilities into the same runtime and resolve them by application, module, or plugin tree location.

It is not intended to replace .NET DI, but rather complements these capabilities on top of [`IServiceCollection`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection) _[Source](https://source.dot.net/#Microsoft.Extensions.DependencyInjection.Abstractions/IServiceCollection.cs)_, `System.IServiceProvider`, and [`IServiceProviderFactory<TContainerBuilder>`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iserviceproviderfactory-1) _[Source](https://source.dot.net/#Microsoft.Extensions.DependencyInjection.Abstractions/IServiceProviderFactory.cs)_:

* application context: Use [`IApplicationContext`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationContext.cs) to represent the current application instance, uniformly exposing configuration, environment, modules, services, events, [workers](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) and lifecycle.
* Application module: Use [`IApplicationModule`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationModule.cs) to represent a subsystem or plugin module, and provide the module with its own service resolution domain.
* Service discovery: Find services by service name, tags, matching parameters, module names, and plugin tree expressions, rather than just parsing by type.
* Service building: String declarative registration, code registration and property injection through [`ServiceAttribute`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ServiceAttribute.cs), [`IServiceRegistration`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IServiceRegistration.cs), [`ServiceDependencyAttribute`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ServiceDependencyAttribute.cs) and [`ServiceProviderFactory`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ServiceProviderFactory.cs).
* Distributed collaboration: Basic abstractions such as distributed lock and lock tokens are provided in `Zongsoft.Services.Distributing`.

## Runtime Structure

In plugin-based applications, the service model usually forms three layers:

| Hierarchy | entrance | function |
| --- | --- | --- |
| Application service container | `ApplicationContext.Current.Services` | The global default container hosts services registered by hosts, plugins, and frameworks. |
| module service container | `ApplicationContext.Current.Modules["模块名"].Services` | The service is resolved based on the module name boundary; if it is not found within the module, it will fallback to the application service container. |
| plugin tree builtin | `/Workspace/Environment/Services`、`/Workbench/...` | Declarative objects and extension points; where service nodes are registered with the application service container, other nodes are usually discovered through paths or builtin resolvers. |

The design intention of this structure is to separate "process-level infrastructure" and "business capabilities provided by plugins": the host is responsible for establishing the Host and service container, the plugin is responsible for declaring assembly, builtin and extension point, and the application context is responsible for organizing the two into a queryable runtime.

{% hint style="info" %}
Not all objects in the plugin tree are DI services. Only objects that are scanned and registered by assembly, explicitly registered by code, or are hung under `/Workspace/Environment/Services` and added to the service collection by the host build process will enter the service container.
{% endhint %}

## Registration Source

Services typically enter an application from four sources.

### Code Registration

Hosts or plugins can add services directly to the services collection during the Host build phase. Below is the actual register for the gRPC extension in the framework, configuring protocol services and reflection support. Normal .NET service lifecycle still takes effect according to DI rules, such as singleton, scope and transient.

Source: [framework/Zongsoft.Web/grpc/GrpcInitializer.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Web/grpc/GrpcInitializer.cs#L86) (excerpt; see source for context).

{% code title="GrpcInitializer.cs" %}
```csharp
public void Register(IServiceCollection services, IConfiguration configuration)
{
	services.AddGrpc();
	services.AddGrpcReflection();
}
```
{% endcode %}

Code registration is suitable for hosting private services, starting portal-related services, or infrastructure services that require a clear lifecycle.

### Assembly Scan

When the plugin host builds the application, it will call [`ServiceCollectionExtension.Register`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ServiceCollectionExtension.cs) to scan the public types in the assembly:

1. First scan the assembly referenced by the entry assembly.
2. Then scan the entry assembly itself.
3. Then recursively scan the assembly declared in the loaded plugin `manifest`; the same assembly is only registered once.

There are two registration modes when scanning:

| mode | Applicable scenarios | behavior |
| --- | --- | --- |
| Implement `IServiceRegistration` | An assembly needs to centrally register multiple services, complex lifecycles, or conditional registrations. | The framework creates a registrar instance and calls `Register(services, configuration)`; the registrar takes over service registration of this type. |
| Mark `ServiceAttribute` | The type itself is a service, suitable for simple and conventional registration. | The framework registers the implementation type as a singleton and additionally registers it by contract, name, label, and static member rules. |

Source: [src/Services/ForumService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ForumService.cs#L41) (excerpt; see source for context).

{% code title="ForumService.cs" %}
```csharp
[Service(nameof(ForumService))]
[DataService(typeof(ForumCriteria))]
public class ForumService : DataServiceBase<Forum>
{
	#region 构造函数
	public ForumService(IServiceProvider serviceProvider) : base(serviceProvider) { }
```
{% endcode %}

When `ServiceAttribute` specifies a name, the framework will register the name. When the name ends with `Service`, the short name without the suffix will also be registered. Therefore, the ForumService of Discussions will register the ForumService name and the Forum short name.

### Static Member Registration

If the service object is inherently a static property or field, the member value can be exposed through `ServiceAttribute.Members`. The framework reads the specified public static members and registers them as singleton instances by member type and explicit contract.

Source: [framework/Zongsoft.Diagnostics/protocols/server/src/Listener.Metrics.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Diagnostics/protocols/server/src/Listener.Metrics.cs#L51) (excerpt; see source for context).

{% code title="Listener.Metrics.cs" %}
```csharp
[Service(Tags = "gRPC", Members = nameof(Metrics))]
partial class Listener
{
	#region 单例字段
	public static readonly MetricsProcessor Metrics = new();
```
{% endcode %}

This method is suitable for objects that are stateless, globally unique, and already provided by the framework or third-party library. For services that require dependency injection of construction parameters, ordinary type registration is preferred.

### Plugin Tree Service Node

The plugin host will also look for the `/Workspace/Environment/Services` node and register each child builtin under the node as a singleton service into the application service collection. The service type is `ValueType` from the builtin, and the service instance is created or unpacked by the builtin at parse time.

The Discussions plugin declaration hangs the module in /Workbench/Modules, and then exposes data accessor, events and properties. It does not declare the /Workspace/Environment/Services node; see [plugin manifest](../plugins/plugin-file.md) for a complete list. Do not treat any module node as this dedicated service registration node.

This method is suitable for services that require slave plugin file declaration and are assembled with `{option:...}`, `{path:...}` or `{service:...}`. Its lifecycle is registered as a singleton in the current host implementation, so don't put request-level state, session state, or objects that must be frequently rebuilt here.

## Service Analysis

Service resolution is divided into four categories: by type, by name, by label, and by matching parameters.

| way | API | Description |
| --- | --- | --- |
| type resolution | `Resolve<T>()`、`ResolveRequired<T>()`、`ResolveAll<T>()` | Semantic encapsulation of standards `GetService`, `GetRequiredService`, `GetServices`. |
| name resolution | `Resolve("name")`、`ResolveRequired("name")` | Name resolution service registered under `ServiceAttribute.Name`. |
| Match parsing | `Find<T>(argument)`、`FindAll<T>(argument)` | Find matches among multiple implementations of the same contract. The service can implement [`IMatchable`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IMatchable.cs) or [`IMatcher<T>`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IMatcher.cs), and can also use the `Name` attribute and parameters to ignore case matching. |
| Tag parsing | `Resolves(tag)`、`GetTags(tag)` | Organizing service collections according to `ServiceAttribute.Tags` is suitable for grouping a group of similar extensions into the same purpose. |

{% hint style="warning" %}
Resolution by name relies on the name mapping recorded during the registration phase, mainly from `ServiceAttribute.Name`. If you only manually register `services.AddSingleton<T>()` without additional registration names, you cannot find it directly through `Resolve("name")`.
{% endhint %}

### Parse by Type

Parsing by type is suitable for ordinary dependencies: the caller knows which contract is required and does not care who implements it. `Resolve<T>()` is suitable for optional dependencies, `ResolveRequired<T>()` is suitable for strong dependencies that fail if missing, and `ResolveAll<T>()` is suitable for multi-implementation collections such as [initializers](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationInitializer.cs), handlers, and filters.

There are two typical usages in [`ApplicationContext`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationContext.cs):

* `Exit(...)` finds the current [`IHost`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.ihost) _[Source](https://source.dot.net/#Microsoft.Extensions.Hosting.Abstractions/IHost.cs)_ via `Resolve<IHost>()`. Only when the host and lifecycle services exist and the host has not been stopped, the process will be stopped; then System.Environment.Exit will still be called to exit the process. Therefore this method cannot be treated as a "do nothing without a host" query.
* `Initialize()` collects all application [initializers](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationInitializer.cs) through `ResolveAll<IApplicationInitializer>()` and then executes them one by one.

Source: [framework/Zongsoft.Core/src/Services/ApplicationContext.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationContext.cs#L150) (excerpt; see source for context).

{% code title="ApplicationContext.cs" %}
```csharp
public void Exit(int exitCode, TimeSpan timeout = default)
{
	if(_disposed != 0)
		return;

	var host = _services.Resolve<IHost>();
	var lifetime = _services.GetService<IHostApplicationLifetime>();

	if(host != null && lifetime != null)
	{
		//如果应用程序正在停止或已经停止，则不再执行停止操作，否则会导致死锁
		var exiting = lifetime.ApplicationStopping.IsCancellationRequested || lifetime.ApplicationStopped.IsCancellationRequested;

		if(!exiting)
		{
			if(timeout > TimeSpan.Zero)
				host.StopAsync(timeout).GetAwaiter().GetResult();
			else
				host.StopAsync().GetAwaiter().GetResult();

			host.WaitForShutdown();
		}
	}

	System.Environment.Exit(exitCode);
}
```
{% endcode %}

Source: [framework/Zongsoft.Core/src/Services/ApplicationContext.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationContext.cs#L188) (excerpt; see source for context).

{% code title="ApplicationContext.cs" %}
```csharp
public virtual bool Initialize()
{
	ObjectDisposedException.ThrowIf(_disposed != 0 || _initializers == null, this);

	var initialized = Interlocked.Exchange(ref _initialized, 1);
	if(initialized != 0)
		return false;

	var services = this.Services;

	if(services != null)
		_initializers.AddRange(services.ResolveAll<IApplicationInitializer>());

	foreach(var initializer in _initializers)
		initializer?.Initialize(this);

	return true;
}
```
{% endcode %}

These two examples illustrate the boundaries of type resolution: `Resolve<T>()` for a single optional object, and `ResolveAll<T>()` for a collection of multiple extension points. If the caller must get the service before continuing, use `ResolveRequired<T>()` instead, which exposes the missing registration as an explicit exception.

### Parse by Name

Resolution by name is suitable for scenarios where the text configuration ultimately points to a service instance. [`MessageQueueConverter`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Messaging/MessageQueueConverter.cs) in the core library is an example of this: it converts a string to `IMessageQueue`.

Its parsing order is:

1. If the text is in the shape of `queue@provider`, first use `Find<IMessageQueueProvider>(provider)` to find the corresponding queue provider, and then get the queue from the provider.
2. If no provider is specified, `ResolveAll<IMessageQueueProvider>()` is traversed to find the first provider containing the queue name.
3. If all providers are not found, the queue instance is finally resolved by service name using `Resolve(text)`.

Source: [framework/Zongsoft.Core/src/Messaging/MessageQueueConverter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Messaging/MessageQueueConverter.cs#L47) (excerpt; see source for context).

{% code title="MessageQueueConverter.cs" %}
```csharp
public static IMessageQueue Resolve(IServiceProvider services, string text)
{
	if(services == null || string.IsNullOrEmpty(text))
		return null;

	var index = text.IndexOf('@');

	if(index > 0 && index < text.Length - 1)
	{
		var provider = services.Find<IMessageQueueProvider>(text[(index + 1)..]);
		if(provider == null)
			return null;

		var name = text[..index];
		return provider.Exists(name) ? provider.Queue(name) : null;
	}

	foreach(var provider in services.ResolveAll<IMessageQueueProvider>())
	{
		if(provider.Exists(text))
			return provider.Queue(text);
	}

	return services.Resolve(text) as IMessageQueue;
}
```
{% endcode %}

`Resolve(text)` here is a fallback solution, suitable for situations where a queue object itself has been registered as a naming service through `ServiceAttribute.Name`. If the configuration must hit the naming service, you can use `ResolveRequired(name)` to cause configuration errors to fail directly in the parsing phase.

### Search by Parameters

`Find<T>(argument)` and `FindAll<T>(argument)` are suitable for scenarios where there are multiple implementations of the same contract and the caller only knows one "selection parameter". A large number of basic types in the core library already implement `IMatchable` or `IMatchable<string>` for this mode.

For example, [`ExpressionEvaluatorBase`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/ExpressionEvaluatorBase.cs) represents the expression evaluator name with `Name`, and `IMatchable` supports ignoring case matching. The caller only needs to pass in the name to find the target evaluator among multiple `IExpressionEvaluator` implementations.

Source: [framework/Zongsoft.Core/src/Expressions/ExpressionEvaluatorBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/ExpressionEvaluatorBase.cs#L71) (excerpt; see source for context).

{% code title="ExpressionEvaluatorBase.cs" %}
```csharp
bool Services.IMatchable.Match(object argument) => argument is string name && string.Equals(name, this.Name, StringComparison.OrdinalIgnoreCase);
bool Services.IMatchable<string>.Match(string name) => string.Equals(name, this.Name, StringComparison.OrdinalIgnoreCase);
```
{% endcode %}

The actual call to match by name and get the queue is shown in the MessageQueueConverter above. See [Scripts and Expressions](../externals/scripting.md) for an execution example of the expression evaluator. Whether the name is available depends on whether the corresponding extension package is registered.

[`TextRegular`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Text/TextRegular.cs) treats the matching parameters as text to be verified. In other words, `Find<ITextRegular>(text)` does not find regular rules by name, but finds rules that can match the text in a set of text rule services.

Source: [framework/Zongsoft.Core/src/Text/TextRegular.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Text/TextRegular.cs#L108) (excerpt; see source for context).

{% code title="TextRegular.cs" %}
```csharp
bool Services.IMatchable.Match(object parameter) => parameter != null && this.Match(parameter.ToString());
```
{% endcode %}

Message queue providers are also typical scenarios. [`MessageQueueFactoryBase`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Messaging/MessageQueueFactoryBase.cs) matches the queue factory by name, and `Find<IMessageQueueProvider>(provider)` is called when `MessageQueueConverter` parses `queue@provider`.

Source: [framework/Zongsoft.Core/src/Messaging/MessageQueueFactoryBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Messaging/MessageQueueFactoryBase.cs#L48) (excerpt; see source for context).

{% code title="MessageQueueFactoryBase.cs" %}
```csharp
protected virtual bool OnMatch(string name) => string.Equals(this.Name, name, StringComparison.OrdinalIgnoreCase);
bool IMatchable.Match(object argument) => this.OnMatch(argument as string);
bool IMatchable<string>.Match(string argument) => this.OnMatch(argument);
```
{% endcode %}

Therefore, the key to `Find` is not to "get the first service by type", but to let each candidate service determine for itself "whether I fit this parameter." When `IMatchable` or `IMatcher<T>` is not implemented, the framework will try to use the `Name` attribute of the service for default matching.

### Get Label Set

Tags are good for organizing services by "purpose" rather than selecting individual implementations by type or name. `Zongsoft.Diagnostics.Protocols.Server` and `Zongsoft.Web.Grpc` use labels to decouple gRPC service registration from endpoint mapping.

In [`Listener.Metrics.cs`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Diagnostics/protocols/server/src/Listener.Metrics.cs), the `Metrics` static member is marked as a service and labeled `gRPC`:

For static members and their Service annotations, see the real source code snippet of "Static Member Registration" above.

When `ServiceCollectionExtension` scans this annotation, it will register the `Metrics` member value as a service and classify the service type into the `gRPC` tag. By [`GrpcInitializer`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Web/grpc/GrpcInitializer.cs), the [initializer](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationInitializer.cs) does not need to know which diagnostics or business gRPC services there are, it only needs to read the service type under the tag:

Source: [framework/Zongsoft.Web/grpc/GrpcInitializer.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Web/grpc/GrpcInitializer.cs#L57) (excerpt; see source for context).

{% code title="GrpcInitializer.cs" %}
```csharp
public void Initialize(IApplicationBuilder builder)
{
	if(builder is IEndpointRouteBuilder app)
	{
		foreach(var service in app.ServiceProvider.GetTags("gRPC"))
		{
			MapGrpcService(app, service);
		}

		app.MapGrpcReflectionService();
	}
}
```
{% endcode %}

The applicable scenario of this example is clear: the plugin or module is responsible for declaring "I am a gRPC service", and the Web gRPC [initializer](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationInitializer.cs) is responsible for uniformly mapping all services with the `gRPC` label. Both parties do not need to reference each other's specific implementations.

### Parse by Tag

`GetTags(tag)` returns the service type under the label, which is suitable for scenarios such as `GrpcInitializer` that "only require types, no instances". `Resolves(tag)` and `Resolves(Type, tag)` will further parse the instance from the service container, which is suitable for scenarios where the service object under the label needs to be directly called.

The framework TaggedServiceTest provides real instance parsing testing. ITaggedContractA, ITaggedContractB, TaggedService and ProviderScope are all defined in the same test file; the two contracts should resolve to the same object.

Source: [framework/Zongsoft.Core/test/Services/TaggedServiceTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Services/TaggedServiceTest.cs#L15) (excerpt; see source for context).

{% code title="TaggedServiceTest.cs" %}
```csharp
public void Register_SameServiceAndTagAcrossAttributes_MergesAllContracts()
{
	using var provider = CreateProvider();

	var first = Assert.Single(provider.Provider.Resolves<ITaggedContractA>(TaggedService.Tag.ToLowerInvariant()));
	var second = Assert.Single(provider.Provider.Resolves<ITaggedContractB>(TaggedService.Tag.ToUpperInvariant()));

	Assert.Same(first, second);
}
```
{% endcode %}

The test scans the test assembly through the following factory and then creates the framework service provider. Registration must be completed before parsing; tags do not rely on the caller to be temporarily attached to the object.

Source: [framework/Zongsoft.Core/test/Services/TaggedServiceTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Services/TaggedServiceTest.cs#L57) (excerpt; see source for context).

{% code title="TaggedServiceTest.cs" %}
```csharp
private static ProviderScope CreateProvider()
{
	var services = new ServiceCollection();
	services.Register(typeof(TaggedServiceTest).Assembly, null);
	return new ProviderScope(new ServiceProviderFactory().CreateServiceProvider(services));
}
```
{% endcode %}

It is more appropriate to choose `GetTags("gRPC")` when actually mapping the gRPC endpoint, because `MapGrpcService<TService>()` requires the service type; when you need to perform the service object behavior, use `Resolves(...)` to resolve the instance.

## Module Service Domain

The module service domain is used to solve the problem of "the same contract may have different implementations in different modules". The `Services` attribute of the module will create a service provider with the module name based on the application service container; when parsing, it will first try the modular service and then fallback to the application service container.

The source of modular services is usually a service type with [`ApplicationModuleAttribute`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationModuleAttribute.cs). When the framework scans the service contract, it will additionally register the packaging service with the module name, so that the module container can obtain the implementation of this module first.

Source: [src/Module.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Module.cs#L31) (excerpt; see source for context).

{% code title="Module.cs" %}
```csharp
[assembly: ApplicationModule(Zongsoft.Discussions.Module.NAME)]
```
{% endcode %}

Discussions Declare the [ApplicationModule](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationModule.cs) on the assembly and define Module.NAME as Discussions. Property injection for MessageSendCommand explicitly selects the same module service domain.

When an object is located under a module or plugin tree node, the framework will try to select a service container based on the module to which the object belongs. In this way, business plugins can declare their own module services while still reusing application-level public services.

## Property Injection

Constructor injection is still preferred. Property or field injection is mainly used for plugin builtin, runtime reflection construction objects, optional dependencies, or scenarios where services need to be selected by module name.

[`ServiceDependencyAttribute`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ServiceDependencyAttribute.cs) supports three types of choices:

| settings | meaning |
| --- | --- |
| Do not specify `Provider` | Use the service container of the module where the injection target is located; fallback to the application service container when it is not found. |
| `Provider = "/"` or `Provider = "*"` | Use the application service container directly. |
| `Provider = "模块名"` | Use the service container of the specified module; fallback to the application service container when it is not found. |

Source: [src/Services/Commands/MessageSendCommand.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/Commands/MessageSendCommand.cs#L58) (excerpt; see source for context).

{% code title="MessageSendCommand.cs" %}
```csharp
[ServiceDependency(Provider = Module.NAME)]
public MessageService Service { get; set; }
```
{% endcode %}

If `ServiceName` is set, the injector will instead use the `GetService(string)` lookup naming service for [`IServiceProvider<T>`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IServiceProvider%601.cs). `ServiceName = "~"` or `ServiceName = "."` indicates that the module name where the injection target is located is used as the service name.

## Service Discovery in Plugins

The `{service:...}` expression in the plugin manifest is processed by [`ServicesParser`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/Services/ServicesParser.cs) in [`Zongsoft.Plugins`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Plugins). Instead of simply fetching objects from the global container, it selects the service domain based on the current builtin location and explicit container name.

| expression | result |
| --- | --- |
| `{service:@}` | Return the application default service container. |
| `{service:@模块名}` | Returns the module service container named `模块名`; if the module does not exist, it returns empty. |
| `{service:服务名}` | Resolve a service named `服务名` from the module container or application container to which the current builtin belongs. |
| `{service:服务名@模块名}` | Resolve a service named `服务名` from the `模块名` module service container. |
| `{service:~}` | Resolve a service based on the current target member type. |
| `{service:*}` | Resolve all services by current target member type. |
| `{service:~@}`、`{service:*@}` | Forces resolution by target member type from the application's default service container. |
| `{service:~@模块名}`、`{service:*@模块名}` | Resolves by target member type from the specified module service container. |

The "module name" and "service name" in the above table are syntax placeholders, not registered services. Discussions.plugin actually uses static and path expressions to mount modules, accessors and filters. You can read it with reference to [Builtins and Services](../plugins/builtins-and-services.md).

All formats can also continue to access properties or fields after the service object, such as `{service:服务名.属性名@模块名}`. This is suitable for referencing some configuration object or sub-object of a registered service in the plugin builtin property.

{% hint style="info" %}
When `@模块名` is not explicitly written, the service resolver will try to use the current builtin parent node name to match the module name; only if the match fails, the application default service container will be used. If the plugin path naming can be consistent with the module name, the service expression will be more natural.
{% endhint %}

## Plugin Host Build Order

The application builder in `Zongsoft.Plugins.Hosting` orchestrates service registration and plugin loading. When using `Application.Daemon(...)` and `Application.Terminal(...)` as the entry points, the construction process is roughly as follows:

1. Create a .NET Host builder and set the container factory to `Zongsoft.Services.ServiceProviderFactory`.
2. Load host `.option` configuration by application name.
3. Create [`PluginOptions`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/PluginOptions.cs) and add the plugin configuration source to the application configuration.
4. Call [`PluginTree.Get(options).Load()`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/PluginTree.cs) to load the plugin tree.
5. Scan the host reference assembly, host assembly and plugin manifest assembly, and perform service registration.
6. Register the default `System.Net.Http.HttpClient` service.
7. Register the builtin under `/Workspace/Environment/Services` as a singleton service.
8. Build the Host and initialize [`PluginApplicationContext`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/PluginApplicationContext.cs) through `Initialize()`.
9. When the application starts, open the [workbench](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/IWorkbenchBase.cs) and load the [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) under `/Workbench/Startup`.

`Daemon` and `Terminal` will first register their respective application context implementations and then map to [`PluginApplicationContext`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/PluginApplicationContext.cs) and [`IApplicationContext`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationContext.cs). This means that application code typically only depends on `IApplicationContext`, while the plugin host still has access to the plugin tree and [workbench](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/IWorkbenchBase.cs) using a more specific plugin context.

## Usage Suggestions

* Services that are host-specific, lifecycle-sensitive, and require immediate configuration are registered with code.
* Common business services in plugin assembly are registered with `ServiceAttribute` or `IServiceRegistration`.
* Objects that need to be declared in the plugin manifest and can be assembled by configuration or path expressions are placed in `/Workspace/Environment/Services`.
* Objects that need to be discovered by other plugins according to the extension point are first hung in the agreed plugin tree path instead of being forcibly put into the DI container.
* When there are multiple implementations of the same contract, use `Find<T>(argument)`, `IMatchable`, `IMatcher<T>` or tag organization. Do not hard-code the implementation selection logic on the caller.
* The internal services of the module should be marked with [`ApplicationModuleAttribute`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/ApplicationModuleAttribute.cs) as much as possible so that the module container can prioritize the implementation of this module.

{% hint style="warning" %}
The common types `ServiceAttribute` scanned and registered are registered as singleton by default. Short lifecycle objects that contain mutable state, requested state, or need to be released should register an explicit lifecycle with code, or create it through a factory service.
{% endhint %}

## Sub Namespace

| namespace | Description |
| --- | --- |
| `Zongsoft.Services.Distributing` | distributed lock, lock token, and distributed collaboration base types. |

## Related Resources

* [Distributed Locks](services/distributed-lock.md)
* [Plugin Application Model](../plugins/application-model.md)
* [Host Integration](../plugins/hosting.md)
* [Builtins and Services](../plugins/builtins-and-services.md)
* [Services source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Services)
* [Plugins Hosting source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Plugins/src/Hosting)
* [Plugins Services source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Plugins/src/Services)


For information on parsing by contract, matching by name, provider positioning, and shared instance ownership, see [Service Resolution and Ownership](services/locating.md). For the language and concurrency differences of expression implementation, see [Scripts and Expressions](../externals/scripting.md).
