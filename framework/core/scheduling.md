---
description: "Use the framework Hangfire example to concatenate the dispatch contract, handler and host lifecycle."
icon: calendar
---

# Zongsoft.Scheduling

Scheduling separates "when to execute" from "what to execute". [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) provides a public contract for scheduling, and the specific implementation determines delay, cycle, persistence, and failure handling methods. Discussions There is no ready-made scheduling business. This page follows the MyHandler example of the framework.

## Start with Handler

Source: [framework/externals/hangfire/samples/MyHandler.cs](https://github.com/Zongsoft/framework/blob/main/externals/hangfire/samples/MyHandler.cs#L11) (excerpt; see source for context).

{% code title="MyHandler.cs" %}
```csharp
public class MyHandler : HandlerBase<object>
{
	private long _count = 0;

	protected override ValueTask OnHandleAsync(object argument, Parameters parameters, CancellationToken cancellation) =>
		Logging.GetLogging(this).DebugAsync(
			"MyHandler handles the scheduling of the Hangfire.",
			new
			{
				Count = Interlocked.Increment(ref _count),
				Argument = argument,
				Parameters = parameters,
			},
			cancellation);
}
```
{% endcode %}

The handler receives business parameters, extended Parameters and cancellation tokens. The asynchronous interface does not mean that the current work must be time-consuming or require a background thread; the main thing here is log writing.

## Hang to a Location Discoverable by the Scheduler

Source: [framework/externals/hangfire/samples/Zongsoft.Externals.Hangfire.Samples.plugin](https://github.com/Zongsoft/framework/blob/main/externals/hangfire/samples/Zongsoft.Externals.Hangfire.Samples.plugin#L19) (excerpt; see source for context).

{% code title="Zongsoft.Externals.Hangfire.Samples.plugin" %}
```xml
<extension path="/Workbench/Scheduler/Handlers">
	<object name="MyHandler" type="Zongsoft.Externals.Hangfire.Samples.MyHandler, Zongsoft.Externals.Hangfire.Samples" />
</extension>
```
{% endcode %}

Scheduled jobs are associated with handlers by stable names. When renaming types, handler names, or moving plugins, consider references to jobs that already exist in the storage, otherwise the old jobs may still fail to execute after successful deployment.

## Scheduling Methods and Resources

| way | Things to check |
| --- | --- |
| Delayed execution | Whether the host is running when time base, cancellation, expiration |
| Periodic execution | cron syntax, time zone, strategy on missed triggers |
| persistent job | Storage availability, restart recovery, parameter compatibility |
| Multiple instance execution | Allocation mechanism, repetitive execution and idempotent |

Specific capabilities are subject to implementation. See [Project description](../externals/projects/hangfire.md) for Hangfire integration and storage plugins. The [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) contract cannot be understood as automatically having all persistence capabilities.

## Verify and Stop

First build and deploy a real sample, then register MyHandler with the selected scheduler and confirm that the log contains Count, Argument and Parameters. This count is reset to zero on restart and is not shared across multiple workers; it is not a persistent credential for job completion.

Stopping should stop new schedules, handle cancellations, and release service resources. Failed jobs may be retried, and business actions outside the sample require their own idempotent and compensation strategies. For background hosts, see [daemon](../../hosting/daemon.md).
