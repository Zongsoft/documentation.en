---
description: "Illustrating handler registration, job execution, and retry boundaries through the framework Hangfire sample."
icon: calendar-check
---

# Task Scheduling and Resilient Execution

Discussions currently does not have scheduled cleanup, message retry or daily report handlers. Therefore this page uses MyHandler in the framework externals/hangfire/samples.

## Existing Task Handler

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

The handler records parameters and execution times; it does not send emails, modify forum data, or persist counts. It is suitable to first confirm that the scheduler can find and call the handler.

## List Registration

Source: [framework/externals/hangfire/samples/Zongsoft.Externals.Hangfire.Samples.plugin](https://github.com/Zongsoft/framework/blob/main/externals/hangfire/samples/Zongsoft.Externals.Hangfire.Samples.plugin#L19) (excerpt; see source for context).

{% code title="Zongsoft.Externals.Hangfire.Samples.plugin" %}
```xml
<extension path="/Workbench/Scheduler/Handlers">
	<object name="MyHandler" type="Zongsoft.Externals.Hangfire.Samples.MyHandler, Zongsoft.Externals.Hangfire.Samples" />
</extension>
```
{% endcode %}

The stable handler name is MyHandler. The assembly and manifest also depend on the Hangfire master plugin. Full operation requires configuring storage and starting the server, only loading the sample DLL does not automatically generate a job.

## Scheduling and Execution Are Understood Separately

The scheduler decides when to execute and what data to pass in; the handler implements the execution content. Period, delay and retry have their own configurations, and one execution result cannot be regarded as a guarantee for all strategies. See [Hangfire project](projects/hangfire.md) for existing integration methods.

## Elastic Strategy

Polly provides integration of execution strategies such as retries and timeouts, which are at different levels from task persistence. Timeout of an operation may have had side effects; retrying will call it again. Therefore, business actions should clearly define the idempotent key, success criteria and compensation, and do not retry all exceptions.

If you add background tasks to Discussions in the future, you should first select the real business entrance and retain the SiteId and permission context, and then evaluate transactions, repeated executions, and storage failures. Related reading: [Scheduling](../core/scheduling.md), [Polly](projects/polly.md), [business matters](../data/transactions.md).
