---
description: "An extension point is composed of a composition path and behavioral agreements; give filters, validators, and events maintainable collaboration contracts."
icon: diagram-project
---

# Designing Extension Points as Collaboration Contracts

That a plugin loads only means it has entered the runtime. For another team or plugin to extend it, you must also state where objects attach, which contract they implement, when they are invoked, and who is responsible when something fails. An extension point meant to last cannot be just “a list you can add things to”; it has to be a **contract** both sides can work against independently.

Zongsoft's plugin tree makes composition locations explicit, its service container provides capability lookup, and the data engine, Web, and security modules each define the business semantics of their extension points. This article uses the forum's filters to show which agreements a designer needs to add around these mechanisms. For syntax, see [Plugin Manifests and Loading](../../framework/plugins/plugin-file.md) and [Builtins and Services](../../framework/plugins/builtins-and-services.md).

## Read Two Responsibilities from an Existing Extension Point

The forum's [Zongsoft.Discussions.plugin](https://github.com/Zongsoft/discussions/blob/main/src/Zongsoft.Discussions.plugin) references the existing module instance at `/Workbench/Modules` and exposes the module accessor's filter collection as a mountable node. It then attaches `PostFilter` and `ThreadFilter` under `/Workbench/Modules/Discussions/Accessor/Filters`.

This composition carries two responsibilities:

- The module side provides an **extensible collection**: `expose` brings the accessor's `Filters` collection into the plugin tree as a stable mount target.
- The filter side provides **objects that meet its requirements**: `type` on an `object` describes the builtin type to construct, and the builtin name is a tree node name.

The path is only part of the contract. Existing filters also match models: `PostFilter` handles posts, while `ThreadFilter` handles threads and their content. A successful mount does not mean every query runs that filter. An external plugin contributing to this collection must understand matching conditions, processing stages, and object lifetimes — not merely copy the path string.

![Filter composition and the behavioral agreements around it](../../.gitbook/assets/zongsoft-plugin-extension-contracts.png)

_The left side shows where the filters attach in the plugin tree; the right side lists the behavioral agreements extension authors must spell out. A successful mount does not make these agreements true._

## Choose the Collaboration Mechanism by Completion Requirements

Modules do not have to collaborate through one mechanism. Ask first what the collaboration must accomplish, then choose:

| Collaboration need | Mechanism to use | Agreement to write down |
| --- | --- | --- |
| The caller must get this operation's result | Call a service through an explicit contract | Input, result, permissions, timeout, and failure semantics |
| The owner lets others join a processing step | Contribute a filter, handler, or driver at an extension point | Applicability, order, shared state, and exception handling |
| Several interested parties learn that a fact happened | Define an event and its handling | Timing, payload, and the impact of a failing subscriber |
| Work must cross processes or survive delivery failures | Use messaging with a persistence strategy | Acknowledgment, retry, duplicate handling, and recovery ownership |

The mechanisms can be combined; for example, a service completes an operation and schedules an external index update after commit. But a step that must succeed before a transaction can be confirmed should not become an unacknowledged notification just to feel “decoupled.” Events reduce references to concrete subscribers, but they also move part of the dependency into event semantics and operational diagnostics.

{% hint style="info" %}
The forum's current [Module](https://github.com/Zongsoft/discussions/blob/main/src/Module.cs) only exposes an event registry through `Events`; it does not mean posting or approval already publishes concrete business events, let alone reliably delivers them to a message queue. Such a capability still needs a full implementation; see [Events](../../framework/core/components/events.md).
{% endhint %}

## Write a Usage Agreement for the Extension Point

Suppose the product wants third parties to add post-approval handlers. At review time you should at least produce the following statement. This is an extension design suggestion, not an API the forum already offers:

- **Owner and entry**: who maintains the extension point, which plugin contributors must depend on, and which contract the target collection accepts.
- **Input and applicability**: whether input is committed business identifiers or mutable objects, how a site is identified, and which operations trigger processing.
- **Order and concurrency**: whether multiple handlers have an order, run concurrently, or may be invoked repeatedly. File order is not a business-order guarantee.
- **Failure and lifecycle**: whether an exception affects the main operation, who records and retries, and whether shutdown waits for in-flight work.
- **Compatibility and validation**: how new fields are handled, whether old plugins keep working, and whether the business still works with no contributed handler.

The more mutable objects an extension point exposes, the more easily external plugins depend on internals. For an extension meant to be compatible over time, provide input and output proportional to the operation's purpose. Do not turn the whole service container, data accessor, and internal object graph into a public protocol.

## Review Plugin Dependencies and Project References Together

The forum's [Web manifest](https://github.com/Zongsoft/discussions/blob/main/src/api/Zongsoft.Discussions.Web.plugin) depends on the domain plugin. Assembly references make types compilable, manifest dependencies describe runtime composition order, and deployment carries the files to the target environment. All three should match the actual call relationships; declaring a name in a manifest does not download a NuGet package.

If two business plugins must reference each other's internal services and wait for each other's objects at startup, first ask whether the collaboration responsibility can be reassigned: extract a stable public contract, or let one higher-level component own the complete flow. Dependencies that need an immediate result should stay explicit; replacing them with string service names does not make the dependency disappear. For module service fallback, see the [Plugin Application Model](../../framework/plugins/application-model.md).

## Make Names Locatable Without Mixing Name Domains

A failure record containing only `Discussions` may leave the reader unsure whether it means a module, a plugin, or a configuration section. The forum shows the distinction clearly:

- Plugin name: `Zongsoft.Discussions.Web` (used for manifest dependencies and composition order).
- Module name: `Discussions` (the resolution domain for module services).
- Builtin name: `PostFilter` (a node under the accessor's filter collection).
- Configuration name: option sections starting with `/Discussions`.

Each participates in plugin dependencies, module lookup, tree-node location, or configuration reading respectively, and they are not interchangeable. When adding a public path, service alias, or configuration key, state its owner and scope, and keep the names existing consumers already use.

## Verify Composition, Then Verify Business Effect

Tests of an extension point should cover “the plugin loaded, but the extension did not behave as expected”: model mismatch, duplicate registration, missing target node, handler exception, ordering change, and concurrent calls on a shared instance. Checking that a type can be constructed misses all of these collaboration failures.

For extensions that need reliable notification, also examine the failure window between the database commit and the message send. Depending on consistency requirements, you may need persisted pending messages, compensation, or reconciliation; these are application designs and are not implied by “the framework supports events.” See [Message Delivery Concepts](../../framework/messaging/concepts.md) for delivery boundaries.

Once the contract is clear, maintainers can judge whether a new plugin is compatible, whether startup may proceed without an optional plugin, and who takes over when something fails. The next article carries these agreements into [verified plugin delivery](evolutionary-delivery.md).

## Further Reading

- Elux's [micro-module design](https://github.com/hiisea/elux/blob/main/docs/designed/micro-module.md) discusses module autonomy and inter-module collaboration (including its evented Action mechanism); this article is about Zongsoft's plugin tree, service resolution, and failure handling, whose runtime behavior follows the linked Zongsoft implementation.
- Extension paths and service syntax: [Plugin Manifests and Loading](../../framework/plugins/plugin-file.md).
- Module services and builtin collaboration: [Builtins and Services](../../framework/plugins/builtins-and-services.md).
