---
description: "Responsibilities of the Zongsoft.Components namespace and its subnamespaces."
icon: cubes
---

# Zongsoft.Components

`Zongsoft.Components` is the component model namespace in the core class library, covering basic builtins such as commands, execution pipelines, handlers, filters, converters, supervised objects, [workers](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs), identifiers, and event exchanges. Its goal is not to provide a specific business capability, but to provide a set of runtime abstractions for business objects in different fields that are composable, extensible, and reusable by the host.

## Main Responsibilities

* Define component identities, aliases, weights, named objects, and service descriptions so components can be discovered, described, and selected.
* Provides command mode, command tree, command expression parsing, command exit and parameter binding to trigger business actions across domains and with low coupling.
* Provides execution pipelines, handlers, and filters to organize context, execution logic, and cross-cutting features into scalable execution units.
* Provides runtime component modes such as [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs), monitor, weighted selector, and attempt tracker for background tasks, status observation, and failure control.
* Provide common converters, state machines and component feature extensions to reduce repeated implementation of infrastructure code.

## Command Mode

The command pattern is one of the most commonly reused component models across modules in `Zongsoft.Components`. It describes an operation as a command expression, and `CommandExecutor` locates `CommandNode` in the command tree, creates `CommandContext`, and calls the `ICommand` instance. Commands are not tied to a specific container or host: the same set of commands can be called directly by a background service or web program, or loaded as interactive commands by a terminal host.

This mode is particularly suitable for cross-domain calls. The caller only needs to know "what action to perform" and "which parameters to pass", and does not need to refer to the specific service interface. For example, SMS sending, queue subscription, file operation, configuration reading and other capabilities can all be entered into the unified execution pipeline through command expressions. The specific implementation of general commands is mainly located in the `Zongsoft.Commands` project, and the command interaction in the terminal program is implemented by `Zongsoft.Terminals` based on the same set of models.

{% content-ref url="components/commands.md" %}
[commands.md](components/commands.md)
{% endcontent-ref %}

## Sub Namespace

| namespace | Description |
| --- | --- |
| `Zongsoft.Components` | [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) types of command models, such as `ICommand`, `CommandBase`, `CommandContext`, `CommandNode`, `CommandLine`, and `CommandExecutor`. |
| `Zongsoft.Components.Commands` | Built-in component commands implemented based on the command model, such as [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) start, stop, pause, resume and status query commands. |
| `Zongsoft.Components.Converters` | Common converters for Boolean, enum, version, schema, etc. |
| `Zongsoft.Components.Features` | Retry, Fallback, Breaker, Throttle, Timeout and other execution feature models. |
| `Zongsoft.Components.States` | State machine, state diagram, state context, state handler and state flow. |

## Type

<table data-view="cards">
	<thead>
		<tr>
			<th></th>
			<th data-card-target data-type="content-ref"></th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Identity: Object identification abstraction and unified identification value.</td>
			<td><a href="components/identifier.md">identifier.md</a></td>
		</tr>
		<tr>
			<td>AliasAttribute: declare aliases for types, members, parameters, etc.</td>
			<td><a href="components/alias-attribute.md">alias-attribute.md</a></td>
		</tr>
		<tr>
			<td>Command: low-coupling cross-domain calling model, command line parsing and terminal interaction foundation.</td>
			<td><a href="components/commands.md">commands.md</a></td>
		</tr>
		<tr>
			<td>Events: event description, event location, event exchange, and plugin architecture handler mounting.</td>
			<td><a href="components/events.md">events.md</a></td>
		</tr>
		<tr>
			<td><a href="https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs">Worker</a>: A background worker model that can start, stop, pause, and resume.</td>
			<td><a href="components/worker.md">worker.md</a></td>
		</tr>
		<tr>
			<td>Handler: handler abstraction, selector, and plugin architecture handler collection.</td>
			<td><a href="components/handler.md">handler.md</a></td>
		</tr>
		<tr>
			<td>Filter: filter interface that performs pre- and post-processing around the context.</td>
			<td><a href="components/filter.md">filter.md</a></td>
		</tr>
		<tr>
			<td>Superviser: An observation, failure, and recovery model for supervisory objects.</td>
			<td><a href="components/superviser.md">superviser.md</a></td>
		</tr>
		<tr>
			<td>Weighter: Smooth weighted polling selector.</td>
			<td><a href="components/weighter.md">weighter.md</a></td>
		</tr>
		<tr>
			<td>Discriminator: Let the container identify subcollections or target types by content.</td>
			<td><a href="components/discriminator.md">discriminator.md</a></td>
		</tr>
		<tr>
			<td>Attempter: Failed attempt count statistics and lock window control.</td>
			<td><a href="components/attempter.md">attempter.md</a></td>
		</tr>
		<tr>
			<td>Converters: Commonly used type converters at the component layer.</td>
			<td><a href="components/converters.md">converters.md</a></td>
		</tr>
		<tr>
			<td>State machine: state diagram, state context, state handler and state flow.</td>
			<td><a href="components/states.md">states.md</a></td>
		</tr>
		<tr>
			<td>execution pipeline: combines Executor, context and Feature attributes into an execution unit.</td>
			<td><a href="components/executor.md">executor.md</a></td>
		</tr>
	</tbody>
</table>

## Related Resources

* [Components source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Components)
