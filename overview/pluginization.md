---
description: "Basic concepts and operation methods of Zongsoft plugin-based applications."
icon: puzzle-piece
---

# Plugin Architecture

Plugin architecture is a core feature of Zongsoft. A plugin-based application consists of a host and multiple plugins. The host is responsible for starting and hosting, and the plugins are responsible for providing business capabilities and extension points.

## What Is a Plugin

In Zongsoft, a plugin is not an alias of a single assembly, but a combination of a set of files and metadata. The most common plugin content includes:

- Plugin description file `*.plugin`.
- Plugin assembly `*.dll`.
- Option configuration `*.option`.
- Data map `*.mapping`.
- Localized resource directory, such as `zh-Hans`, `zh-CN`.
- Certificates, templates, static resources and other ancillary files.

## How to Load Plugins

After the host is started, the plugin framework will scan the `plugins/` directory under the host directory, read the plugin description file, load the assembly and create an application context. Plugins can register services, commands, event handlers, web controllers, or other module capabilities.

## Why Use Plugin Architecture

Plugin architecture solves the organizational problems of large business systems:

- Business modules can be released independently.
- Running the host can remain lightweight and stable.
- Different environments can combine different plugins through deployment files.
- Business plugins that do not depend on a particular interaction model or protocol can be reused across host types. Controllers and ASP.NET middleware, however, must run within a Web host.
- Configurations, mappings, and resources can be deployed with the plugin.

## Plugin Design Essays

These four essays examine design choices and their costs through the existing forum implementation. Read them in order or start with your current question. “Proposed” in the illustrations identifies a design scenario; the text explains which capabilities currently exist.

1. [Finding Module Boundaries Through Business Changes](pluginization/business-boundaries.md): Use changes to moderation, content, and statistics to establish ownership.
2. [Reusing Business Capabilities Across Hosts](pluginization/host-independent-business.md): Separate business operations, entry adapters, identity, and lifecycles.
3. [Designing Extension Points as Collaboration Contracts](pluginization/extension-contracts.md): Define extension paths, service calls, names, and failure semantics.
4. [From Incremental Change to Verified Plugin Delivery](pluginization/evolutionary-delivery.md): Validate release combinations and evaluate data compatibility and process separation.

## Document Reading Suggestions

Read [Plugin Framework](../framework/plugins/README.md) first to understand the application model, and then read [Deploy Your First Plugin](../get-started/deploy-first-plugin.md) to understand how the plugin enters the host. When you need to delve into data access, enter [Data Engine](../framework/data/README.md).

## Extension Point Is Also a Contract

One plugin provides a service or collection mount point, and another plugin declares dependencies and contributes implementations. Dependencies not only determine assembly visibility, but also describe "which components are required before they can work." Extension paths, service names, and configuration keys should be maintained like public APIs, with consumers synchronized when changed.

Plugin architecture does not equal arbitrary hot replacement, nor does it equal process-level isolation. Components loaded into the same host share process resources, and version conflicts or incorrect [workers](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) can affect the entire application. When independent fault boundaries are required, independent processes and communication protocols should be considered.

See [basic concepts](concepts.md) for background terminology and [The first business plugin](../get-started/first-business-plugin.md) for the complete combination path.
