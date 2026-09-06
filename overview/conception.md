---
description: "Understand Zongsoft's design trade-offs for public contracts, plugin assembly, explicit metadata, and deployment composition."
icon: lightbulb
---

# Design Principles

Zongsoft organizes capabilities around business applications that can be plugged into a plugin architecture. The focus of the design is to enable the business intent, technical implementation, operating host, and delivery method to evolve independently while retaining a clear connection contract.

## Separation of Host and Business

The host is responsible for startup, configuration, service container and lifecycle. Capabilities such as order processing, message subscription, and report output are provided by plugins. The same business service that does not rely on web requests can be called by terminal commands, background [workers](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs), or web controllers.

This kind of reuse has a prerequisite: the business contract cannot be directly bound to the input and output methods of a certain host. Terminal prompts, HTTP status codes, and background task retry policies should stay in their respective adaptation layers, and business rules should be placed in shared services. See [Host Overview](../hosting/hosting.md).

## Contract-oriented Selection Implementation

Business code usually expresses "I want a cache" or "I want to publish a message", and the implementation selection is completed by service registration, provider and configuration. In this way, the deployment solution can select different capabilities for different environments without spreading the construction code of third-party SDKs in the business.

Abstractions also have boundaries. [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) retains common semantics across implementations; database dialects, message protocols, and cloud platform parameters are left to corresponding drivers or adapters. Just because an option appears on a public interface does not mean that every driver supports it. The corresponding implementation restrictions should be read before calling.

{% hint style="info" %}
💡 Replacement needs to meet both "interface compatibility" and "business semantics compatibility". For example, successful message publishing may only mean that it has entered the client queue, and it cannot be inferred that the consumer has completed business processing.
{% endhint %}

## Use Extension Point to Connect Both Sides

The plugin tree allows capability owners to expose stable extension paths for objects contributed by other plugins. Plugins that provide command executors do not need to reference all business commands. Business commands only need to comply with command contracts and extension path conventions.

The service container solves object dependencies, and the plugin tree solves contributions and assembly. Both have their own uses. Objects that need to be called by the business according to the contract are suitable for registration as services; objects that need to be added to a certain driver, command or filter collection are suitable for contribution to the extension point. See [Builtins and Services](../framework/plugins/builtins-and-services.md) for details.

## Explicit Metadata Makes Structures Auditable

`.plugin` describes assembly, `.option` describes running parameters, `.mapping` describes data relationships, and `.deploy` describes delivery files. Extracting these structures from business calls helps review configurations, reuse plans, and locate environmental differences.

The trade-off is that the metadata itself must also be maintained. Check the manifest when modifying the assembly name, check the mapping when modifying model fields, and check the path in the package when modifying the deployment product. The compiler overrides only part of the contract.

## Modules Express Business Boundaries

The module has application organization capabilities such as service resolution domains and events, and can provide its own services first while reusing application shared services. The module does not automatically isolate database, thread and process permissions; when tenant isolation, data permissions or process isolation is required, it must be designed separately.

Module names, plugin names, service aliases, and connection names should have clear naming conventions, but they do not have to be the same. The responsibilities they bear are shown in [basic concepts](concepts.md#module-service-provider).

## Deployment Is Part of the Application

What the application actually runs is "host release product + selected plugin + configuration and resources". Maintaining a reproducible deployment list reduces manual copying errors and allows development, testing, and release to use the same set of combination definitions.

Deployment, installation, packaging and automatic upgrade assume responsibilities at different stages, see [Deployment Model](deployment.md) for details. Dependency versions, native resources, and configuration coverage should be verified during the delivery phase; missing plugins found by the running environment will not be automatically filled in through NuGet.

## Trade-offs in Practice

Start with a minimum runnable host and a business plugin, and then introduce databases, messages and external services according to actual needs. Each time an implementation is added, the complete path of manifest loading, service resolution, configuration validation, and first call is verified. In this way, when a problem occurs, it can be located along clear boundaries without having to start all dependencies together and then guess the cause.
