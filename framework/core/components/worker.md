---
description: "Zongsoft.Components Worker worker model and worker command."
icon: person-running
---

# Worker

`Worker` is a runtime component model in Zongsoft that can be started, stopped, paused and resumed. It is suitable for expressing objects with lifecycle such as background services, event exchangers, scheduling servers, message listeners, device collectors, etc.

Workers unify lifecycle commands and running status. host, terminal commands, or plugin startup processes can control different runtime components with the same set of methods.

## Key Types

| Type | Description |
| --- | --- |
| `IWorker` | Worker interface defines `Start`, `Stop`, `Pause`, `Resume` and corresponding asynchronous methods. |
| `WorkerBase` | Worker base class, responsible for state switching, reentry control, enablement judgment and state change events. |
| `WorkerState` | Worker status enumeration, including stopped, running, paused, etc. |
| `WorkerCommandBase` | Worker command base class, used to expose worker lifecycle operations as commands. |
| `WorkerStartCommand`、`WorkerStopCommand` | Start and stop workers. |
| `WorkerPauseCommand`、`WorkerResumeCommand` | Suspend and resume workers. |
| `WorkerInfoCommand` | View worker status and basic information. |

## Lifecycle

{% stepper %}
{% step %}
## Start

`WorkerBase.StartAsync(...)` checks the enabled status and current status, enters `Starting`, then calls the startup logic of the derived class, and enters `Running` after success.
{% endstep %}

{% step %}
## Pause / Resume

If the worker supports pause and resume, calling the pause or resume command will switch the state and trigger the corresponding template method.
{% endstep %}

{% step %}
## Stop

The stop process enters `Stopping`, releases running resources, such as closing the event channel, stopping the scheduling server, unsubscribing or releasing external connections, and finally returns to `Stopped`.
{% endstep %}
{% endstepper %}

## Hangfire Example

`Server` in `Zongsoft.Externals.Hangfire` inherits from `WorkerBase`, creates `BackgroundJobServer` when started, and releases the server instance when stopped. It also exposes the `Handlers` collection, allowing plugins to mount the scheduling handler to `/Workbench/Scheduler/Handlers`.

Source: [framework/externals/hangfire/src/Zongsoft.Externals.Hangfire-daemon.plugin](https://github.com/Zongsoft/framework/blob/main/externals/hangfire/src/Zongsoft.Externals.Hangfire-daemon.plugin#L19) (excerpt; see source for context).

{% code title="Zongsoft.Externals.Hangfire-daemon.plugin" %}
```xml
	<extension path="/Workspace/Externals/Hangfire">
		<object name="Server" type="Zongsoft.Externals.Hangfire.Server, Zongsoft.Externals.Hangfire">
			<expose name="Handlers" value="{path:../@Handlers}" />
		</object>
	</extension>

	<extension path="/Workbench/Scheduler">
		<object name="Handlers" value="{path:/Workspace/Externals/Hangfire/Server/Handlers}" />
	</extension>

	<extension path="/Workbench/Startup">
		<object name="Hangfire" value="{path:/Workspace/Externals/Hangfire/Server}" />
	</extension>
```
{% endcode %}

This design allows the server itself to only care about lifecycle, while the handler and startup method are combined by the plugin manifest. The command system can also expose operations such as `start`, `stop`, `pause`, `resume` to the terminal or management program.

Workers are suitable for components with a clear lifecycle that may be started and stopped uniformly by the host. One-time tasks, common business services, or objects without persistent state generally do not need to implement `IWorker`. Suspension and resumption capabilities are governed by `CanPauseAndContinue`; workers that do not support suspension should not forcefully expose suspension semantics.

## Reference Implementation

* [WorkerBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/WorkerBase.cs)
* [Components/Commands Worker Commands](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Components/Commands)
* [Hangfire Server.cs](https://github.com/Zongsoft/framework/blob/main/externals/hangfire/src/Server.cs)
