---
description: "English documentation for the Zongsoft framework, hosts, and toolchain."
icon: book-open
---

# Zongsoft Framework

![Zongsoft documentation cover](.gitbook/assets/zongsoft-docs-cover.png)

Zongsoft is a collection of open-source frameworks, hosts, and development tools for .NET. It helps teams build business applications that are extensible through plugins, straightforward to deploy, and maintainable.

The documentation covers:

{% columns %}
{% column %}
### Framework

Core abstractions, the plugin framework, the data engine, Web fundamentals, security, diagnostics, messaging, automatic upgrades, and adapters for common third-party services.
{% endcolumn %}

{% column %}
### Hosts and Tools

Hosts run plugin-based applications. The toolchain deploys plugins, builds installation packages, publishes upgrade packages, and supports development and debugging.
{% endcolumn %}
{% endcolumns %}

## Quick Navigation

<table data-card-size="large" data-view="cards">
	<thead>
		<tr>
			<th></th>
			<th></th>
			<th data-hidden data-card-target data-type="content-ref">Page</th>
			<th data-hidden data-card-cover data-type="image">Cover</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><strong>Understand the overall design</strong></td>
			<td>Build the big picture starting from the boundaries of the framework, host, and toolchain.</td>
			<td><a href="overview/what-is-zongsoft.md">what-is-zongsoft.md</a></td>
			<td><a href=".gitbook/assets/zongsoft-docs-cover.png">zongsoft-docs-cover.png</a></td>
		</tr>
		<tr>
			<td><strong>Prepare Your Local Environment</strong></td>
			<td>Prepare the SDK, source repositories, directories, and optional containers.</td>
			<td><a href="get-started/prerequisites.md">prerequisites.md</a></td>
			<td><a href=".gitbook/assets/zongsoft-start-cover.png">zongsoft-start-cover.png</a></td>
		</tr>
		<tr>
			<td><strong>Understand Plugin-Based Applications</strong></td>
			<td>Explore the plugin tree, builtins, service registration, and host integration.</td>
			<td><a href="framework/plugins/README.md">plugins</a></td>
			<td><a href=".gitbook/assets/zongsoft-plugins-cover.png">zongsoft-plugins-cover.png</a></td>
		</tr>
		<tr>
			<td><strong>Learn data access</strong></td>
			<td>Read and write object graphs using data schemas, mapping files, and drivers.</td>
			<td><a href="framework/data/README.md">data</a></td>
			<td><a href=".gitbook/assets/zongsoft-data-cover.png">zongsoft-data-cover.png</a></td>
		</tr>
	</tbody>
</table>

## Reading Path

If you are new to Zongsoft, follow this reading path:

1. Start with [What Is Zongsoft?](overview/what-is-zongsoft.md) to understand the overall scope.
2. Read [Plugin Architecture](overview/pluginization.md) to understand why business capabilities are organized as plugins.
3. Follow [Prerequisites](get-started/prerequisites.md) and [Install Packages](get-started/install.md) to prepare your local environment.
4. [Choose a Host](get-started/hosting.md), complete a [minimal deployment](get-started/deploy-first-plugin.md), and write [your first business plugin](get-started/first-business-plugin.md).
5. Continue with the framework guides, choosing the core library, plugin framework, data engine, Web fundamentals, or other topics as needed.

{% hint style="info" %}
These guides follow the process of building a plugin-based application. For package names, source paths, and module relationships, see the [Package and Module Index](references/packages.md).
{% endhint %}

## Explore by Need

- To understand hosts, modules, services, and providers, read [Core Concepts](overview/concepts.md) and [Service Resolution](framework/core/services/locating.md).
- For business data and HTTP APIs, start with [your first query](framework/data/quickstart.md), continue to [Data Services](framework/data/services.md), and then add [Web Controllers](framework/web/controllers.md).
- For asynchronous work and external infrastructure, read [Message Delivery Concepts](framework/messaging/concepts.md), [Task Scheduling](framework/externals/execution.md), and the [Extension Index](framework/externals.md).
- For model calls, observability, or application delivery, explore [AI Integration](framework/intelligences.md), [Diagnostics](framework/diagnostics.md), and the [Upgrade Workflow](framework/upgrading/workflow.md).

{% hint style="info" %}
Verbatim source excerpts retain their original comments and literal values, including Chinese text, so they can be checked against the linked source. Page explanations and navigation are translated into English.
{% endhint %}
