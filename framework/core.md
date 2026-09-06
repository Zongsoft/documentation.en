---
description: "Responsibilities and main namespace of Zongsoft.Core core class library."
icon: cube
---

# Core Library

`Zongsoft.Core` is the core class library of Zongsoft framework and does not rely on third-party class libraries. It provides basic abstractions, utility classes, service models, and runtime capabilities that are shared by other modules in the framework.

## Main Abilities

- Collections, transformations, randomization, enumerations, and common extensions.
- Communication, execution pipeline and component model.
- Configuration, options, INI Profile and resource handling.
- Caching, serialization, transactions and lightweight state machines.
- Security, credentials, and authorization basic abstractions.
- service container, command model, background worker.
- Terminal application support.

## Code Location

```text
framework/Zongsoft.Core
```

## Namespace

<table data-view="cards">
	<thead>
		<tr>
			<th></th>
			<th></th>
			<th data-hidden data-card-target data-type="content-ref">Page</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><strong>common base</strong></td>
			<td>Collections, common tools, reflection, resources, and text processing.</td>
			<td><a href="core/common.md">common.md</a></td>
		</tr>
		<tr>
			<td><strong>Components and Services</strong></td>
			<td>Component model, commands, application context, service access and distributed collaboration.</td>
			<td><a href="core/components.md">components.md</a></td>
		</tr>
		<tr>
			<td><strong>version expression</strong></td>
			<td>Semantic versioning, four-segment numerical version numbers, and version comparison.</td>
			<td><a href="core/versioning/version.md">version.md</a></td>
		</tr>
		<tr>
			<td><strong>Configuration and data abstraction</strong></td>
			<td>Configure bindings, options files, data access abstractions, and data metadata.</td>
			<td><a href="core/configuration.md">configuration.md</a></td>
		</tr>
		<tr>
			<td><strong>runtime capabilities</strong></td>
			<td>Caching, communication, messaging, scheduling, serialization, transactions and state flow.</td>
			<td><a href="core/caching.md">caching.md</a></td>
		</tr>
		<tr>
			<td><strong>Application support</strong></td>
			<td>diagnostics, security, terminal, IO and expression handling.</td>
			<td><a href="core/diagnostics.md">diagnostics.md</a></td>
		</tr>
	</tbody>
</table>

## When to Read This Section

When you need to understand common types, service abstractions, command models, option configurations, or terminal capabilities in other modules, you should first return to the core class library. Modules such as plugin framework, data engine, and Web basic library will reuse the basic abstraction here.

## Related Resources

* [Zongsoft.Core source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core)
* [Zongsoft.Core README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/README.md)
* [Zongsoft.Core NuGet package](https://www.nuget.org/packages/Zongsoft.Core)
