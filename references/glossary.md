---
description: "Find framework, data, messaging, model and delivery terminology and jump to explanations and usage scenarios."
icon: book
---

# Glossary

Use this glossary to find the relevant guides. Each linked topic explains the background, examples, and limitations in detail.

## Applications and Plugins

| Term | Meaning | Further Reading |
| --- | --- | --- |
| Host | The process entry point that establishes the runtime environment and hosts plugins | [Host Overview](../hosting/hosting.md) |
| Content root | A base location used to resolve application configuration and resources; distinguish it from the working directory | [basic concepts](../overview/concepts.md#host) |
| Plugin | An extension unit made up of a manifest and runtime artifacts, not merely another name for a DLL | [Plugin Architecture](../overview/pluginization.md) |
| Plugin tree | The runtime structure that organizes builtins and extension points by path | [Plugin concept](../overview/concepts.md#plugin-tree) |
| Builtin | An object created, exposed, or referenced through plugin metadata | [Builtins and Services](../framework/plugins/builtins-and-services.md) |
| Module | A logical boundary organizing business services and their context | [Modules and providers](../overview/concepts.md#module-service-provider) |
| Service provider | An object that supplies service instances by contract and name | [service resolution](../framework/core/services/locating.md) |
| Site | A scenario division within a deployment configuration, often used for Web composition | [Web Host](../hosting/web.md) |

## Data

| Term | Meaning | Further Reading |
| --- | --- | --- |
| Entity mapping | Metadata describing the relationship between business entities and database tables, fields, and relationships | [Mapping Files](../framework/data/mapping.md) |
| Data schema (Schema) | A selection of members and navigation shapes to read or write | [Data Schemas](../framework/data/schema.md) |
| Navigation property | A member representing a related object or collection | [data concept](../framework/data/concepts.md) |
| Condition | A predicate structure that selects records | [Conditions and Operands](../framework/data/conditions-and-operands.md) |
| Data accessor | An object that performs data operations using names and mappings | [data access](../framework/data/data-access.md) |
| Data service | A layer that organizes authorization, business rules, and extensions around data operations | [Data Services](../framework/data/services.md) |
| Ambient transaction | A transaction scope that propagates with the current execution context | [Transactions and Consistency](../framework/data/transactions.md) |
| Archive format | File conventions for importing and exporting model data | [spreadsheets and templates](../framework/externals/documents.md) |

## Messaging and Distributed Collaboration

| Term | Meaning | Further Reading |
| --- | --- | --- |
| Broker | An intermediary service that accepts, routes, or stores messages | [Message Queues](../framework/messaging.md) |
| Acknowledgment (ACK) | A consumer’s report that a message has reached an agreed processing stage | [delivery concept](../framework/messaging/concepts.md) |
| Idempotency | Repeating the same business operation produces no additional side effects | [reliable delivery](../framework/messaging/reliability.md) |
| Backpressure | Limiting incoming work when downstream processing cannot keep up | [delivery concept](../framework/messaging/concepts.md) |
| Pending | The state of a message accepted by the broker but not yet fully acknowledged | [Message storage](../framework/messaging/reliability.md) |
| Lease | A mechanism that maintains a resource or lock state for a validity period | [Caching and collaboration](../framework/externals/caching.md) |
| Fencing token | A monotonically increasing token that lets a resource reject writes from expired lock holders | [Distributed Locks](../framework/core/services/distributed-lock.md) |

## AI Integration, Diagnostics and Delivery

| Term | Meaning | Further Reading |
| --- | --- | --- |
| Assistant | A named entry point for a set of model connections and service capabilities | [AI Integration](../framework/intelligences.md) |
| Session | An object that manages multi-turn chat state and history | [Sessions and Streaming Responses](../framework/intelligences/sessions.md) |
| Streaming response | Incremental delivery of content before the full result is available | [Sessions and Streaming Responses](../framework/intelligences/sessions.md) |
| Telemetry | Operational observation data such as logs, metrics, and traces | [Diagnostics](../framework/diagnostics.md) |
| OTLP | The OpenTelemetry protocol for transferring telemetry data | [protocol integration](../framework/diagnostics/otlp.md) |
| Deployment manifest | A description of how files are assembled into a runtime directory | [deployment format](deploy-files.md) |
| Release manifest | A description of an upgrade package’s identity, size, checksum, and executor | [Automatic Upgrades](../framework/upgrading.md) |
| Full/incremental release | A full copy after cleanup / an upgrade that overwrites files in the existing directory | [Upgrade type](../framework/upgrading.md) |
| `.deployment` | A handoff description passed from the in-process upgrader to the out-of-process deployer | [upgrade integration](../framework/upgrading/workflow.md) |
