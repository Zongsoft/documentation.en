---
description: "Use Hangfire handler and framework log test to explain log entry, context and placement."
icon: stethoscope
---

# Zongsoft.Diagnostics

diagnostics logs record what happened, where it happened, and the necessary context. Discussions does not have a standalone log handler example, so the existing handler of the framework Hangfire is used for testing with text logs. Telemetry metrics and tracking See also [Telemetry](diagnostics/telemetry.md).

## How Does the Business Handler Write Logs?

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

Logging.GetLogging(this) obtains the log entry based on the current object, and asynchronously records messages and Count, Argument, and Parameters. The counter is an in-process state, not the number of persistent executions; retrying the same task may also count again.

The context here is suitable for troubleshooting, but actual mission parameters may contain sensitive information. When extending this example, only the necessary fields are recorded, and user data should not be processed by following the "write the entire parameter object to the log" approach.

## Log Entry and Output Implementation

The log level expresses the severity, the source is used to locate the module, the exception object retains the call chain, and the additional data carries the structured context. Logger is responsible for output, Predication is responsible for filtering, and Logging organizes entry and distribution; successful registration does not mean that the target file, database, or remote receiver is available.

## Real Concurrency Testing with Text Logs

Source: [framework/Zongsoft.Core/test/Diagnostics/TextFileLoggerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Diagnostics/TextFileLoggerTest.cs#L13) (excerpt; see source for context).

{% code title="TextFileLoggerTest.cs" %}
```csharp
public async Task TestInfoAsync()
{
	const int COUNT = 512;

	using var context = new LoggerContext();
	using var logger = context.CreateLogger(16);

	await Parallel.ForAsync(0, COUNT, TestContext.Current.CancellationToken, async (index, cancellation) =>
	{
		await logger.LogAsync(new LogEntry(LogLevel.Info, "ConcurrentInfo", GetMessage(index)), cancellation);
	});

	await logger.FlushAsync(TestContext.Current.CancellationToken);

	var content = context.ReadAllText();
	AssertMessages(content, COUNT);
}
```
{% endcode %}

LoggerContext, GetMessage and AssertMessages all come from the same test file. Test for concurrent writes by explicitly flushing and then checking that each message appears exactly once. It verifies the local behavior of this text log implementation, not the distributed log delivery promise.

## Filter, Refresh and Exit

Log filtering should first control the source and level, and then consider the output cost. The buffer log may not be flushed before flushing. Process exit and abnormal paths need to check the Flush and release timing. Whether error logs and general information logs use the same buffering strategy should be subject to implementation and testing.

File paths, scrolling, resource text, and exception serialization are also checked with the actual outputter. Do not define a database log class out of thin air just to demonstrate the extended interface; the existing implementations can be read from [diagnostics source code](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Diagnostics) and [Log assertion](common/predication.md).

Related pages: [Spooler](caching/spooler.md), [diagnostics configuration](../diagnostics.md), [Operational troubleshooting](../../get-started/run-and-debug.md).
