---
description: "Understand the composition, positioning and applicable scenarios of the Zongsoft open source framework series."
icon: circle-info
---

# What Is Zongsoft?

Zongsoft is a set of open source frameworks, hosts and toolchains for .NET application development. Its core capabilities revolve around plugin-based applications: the host is only responsible for establishing the running environment, and business capabilities are deployed to the host through plugins, configurations, data mapping and ancillary resources.

## Components

### Framework

[`Zongsoft/framework`](https://github.com/Zongsoft/framework) is the framework code repository, including core class library, plugin framework, data engine, Web basic library, security, diagnostics, message queue, automatic upgrade and third-party service adaptation.

### Hosting

[`Zongsoft/hosting`](https://github.com/Zongsoft/hosting) provides three types of hosts: terminal, background service and Web. The host itself does not carry business logic. They load the plugin manifest in the `plugins/` directory through the plugin framework to form an application.

### Tools

[`Zongsoft/tools`](https://github.com/Zongsoft/tools) provides deployment and packaging tools, including `dotnet-deploy`, `dotnet-pack`, and regular expression GUI tools. The automatic upgrade packager `dotnet-upgrade` is located in the `upgrading/tool` directory of the framework repository.

## Applicable Scenarios

Zongsoft is more suitable for business systems that require long-term evolution, clear module boundaries, and diverse deployment forms. For example:

- Backend services combined with multiple business modules.
- Large applications whose capabilities need to be composed by site, environment, customer, or deployment configuration.
- A system that integrates common capabilities into plugins and reuse them across projects is needed.
- .NET applications that require deployment, installation packages, and automated upgrade processes.

## Relationship to Common Frameworks

Zongsoft, like [ABP](https://abp.io), is a system that favors complete application frameworks and toolchains, but its organizational focus is more toward "plugin-based applications runtime". When reading this document, you can first understand Zongsoft as:

1. A set of basic development abstractions.
2. A model for plugin-based applications.
3. A set of host templates.
4. A set of deployment, packaging and upgrade tools.
