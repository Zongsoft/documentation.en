---
description: "Understand batch buffering, trigger conditions, concurrent refresh and data consumption from the framework log implementation and existing tests."
icon: layer-group
---

# Spooler

Spooler temporarily stores continuously arriving entries and then hands them over to the batch callback for processing. The framework's file logger uses this to reduce the overhead of writing files item by item. It is an in-process buffer. When the entries have not been handed over to the persistence system, the process may still be lost due to exit.

## Actual Usage Location: File Log

Source: [framework/Zongsoft.Core/src/Diagnostics/FileLogger.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Diagnostics/FileLogger.cs#L57) (excerpt; see source for context).

{% code title="FileLogger.cs" %}
```csharp
protected FileLogger(TimeSpan period, int capacity, string filePath, int fileLimit = FILE_LIMIT)
{
	this.FilePath = filePath?.Trim();
	this.FileLimit = Math.Max(fileLimit, 0);
	this.Logging = period > TimeSpan.Zero || capacity > 1 ? new(this.OnFlushAsync, period, capacity) : null;
}
```
{% endcode %}

The constructor determines whether to enable logging buffering based on period and capacity. The actual writing and file size management is implemented by the logger's OnFlushAsync; the Spooler itself only organizes the staging and delivery of entries. For log business examples, see [diagnostics log](../diagnostics.md).

## From Put to Explicit Refresh

Discussions does not use Spooler directly, so the framework is used for testing below. Flusher is a test receiver in the same file that consumes received entries and accumulates the count; TestContext provides the test cancellation token.

Source: [framework/Zongsoft.Core/test/Caching/SpoolerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Caching/SpoolerTest.cs#L30) (excerpt; see source for context).

{% code title="SpoolerTest.cs" %}
```csharp
public async Task TestFlushAsync()
{
	const int COUNT = 1000;

	var flusher = new Flusher<string>();
	using var spooler = new Spooler<string>(flusher.OnFlushAsync, TimeSpan.FromHours(1));
	Assert.True(spooler.IsEmpty);
	Assert.Equal(0, flusher.Count);

	await spooler.FlushAsync(TestContext.Current.CancellationToken);
	Assert.True(spooler.IsEmpty);
	Assert.Equal(0, flusher.Count);

	#if NET8_0_OR_GREATER
	await Parallel.ForAsync(0, COUNT, TestContext.Current.CancellationToken, async (index, cancellation) => await spooler.PutAsync($"Value#{index}", cancellation));
	#else
	for(int i = 0; i < COUNT; i++)
		await spooler.PutAsync($"Value#${i}", TestContext.Current.CancellationToken);
	#endif

	Assert.Equal(COUNT, spooler.Count);

	await spooler.FlushAsync(TestContext.Current.CancellationToken);
	Assert.True(spooler.IsEmpty);
	Assert.Equal(COUNT, flusher.Count);
}
```
{% endcode %}

PutAsync completion indicates that the entry has been received or that the write capacity triggered a flush; it does not uniformly indicate external persistence completion. FlushAsync waits for the end of this refresh callback. The callback must actually consume the received sequence before it can remove the entry from the buffer.

## How to Trigger Capacity and Cycle Respectively?

Source: [framework/Zongsoft.Core/test/Caching/SpoolerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Caching/SpoolerTest.cs#L58) (excerpt; see source for context).

{% code title="SpoolerTest.cs" %}
```csharp
public async Task TestLimitAsync()
{
	var flusher = new Flusher<string>();
	using var spooler = new Spooler<string>(flusher.OnFlushAsync, TimeSpan.FromHours(1), 3);
	Assert.True(spooler.IsEmpty);
	Assert.Equal(0, flusher.Count);

	await spooler.PutAsync("A", TestContext.Current.CancellationToken);
	await spooler.PutAsync("B", TestContext.Current.CancellationToken);
	await spooler.PutAsync("C", TestContext.Current.CancellationToken);
	Assert.Equal(3, spooler.Count);
	Assert.Equal(0, flusher.Count);

	//触发数量限制
	await spooler.PutAsync("D", TestContext.Current.CancellationToken);

	Assert.False(spooler.IsEmpty);
	Assert.Equal(1, spooler.Count);
	Assert.Equal(3, flusher.Count);
}
```
{% endcode %}

This test sets the capacity to 3. The first three items enter the buffer, and when the fourth item is put in, the refresh of the first three items is triggered, and then the fourth item remains in the buffer. Don’t understand capacity as “all orders are placed immediately after item 3 is added”. Unbounded buffering is used when the capacity is zero, and memory growth should be observed in conjunction with consumption speed.

Periodic triggering is done by an internal timer calling FlushAsync. On target frameworks that support modifying Period, the existing [TestPeriodAsync](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Caching/SpoolerTest.cs) adjusts the one-hour period to 1 millisecond and waits for the callback to complete. The times here are for testing and are not recommended deployment parameters.

## Concurrent Refresh and Callback Responsibilities

Source: [framework/Zongsoft.Core/test/Caching/SpoolerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Caching/SpoolerTest.cs#L105) (excerpt; see source for context).

{% code title="SpoolerTest.cs" %}
```csharp
public async Task TestConcurrentFlushAsync()
{
	const int COUNT = 256;
	const int CONCURRENCY = 16;

	var flusher = new RecordingFlusher<int>(TimeSpan.FromMilliseconds(10));
	using var spooler = new Spooler<int>(flusher.OnFlushAsync, TimeSpan.FromHours(1));

	for(int i = 0; i < COUNT; i++)
		await spooler.PutAsync(i, TestContext.Current.CancellationToken);

	var tasks = Enumerable.Range(0, CONCURRENCY).Select(_ => spooler.FlushAsync(TestContext.Current.CancellationToken).AsTask()).ToArray();
	await Task.WhenAll(tasks);

	Assert.True(spooler.IsEmpty);
	Assert.Equal(1, flusher.Calls);
	Assert.Equal(1, flusher.MaximumConcurrency);
	Assert.Equal(COUNT, flusher.Count);
	Assert.Equal(Enumerable.Range(0, COUNT), flusher.Values.OrderBy(value => value));
}
```
{% endcode %}

RecordingFlusher records the number of concurrencies, the number of calls, and the set of entries in the same test file. The test verifies that multiple FlushAsync requests will not enter the callback concurrently, and this batch of entries will only be consumed once. This does not mean that the external system has exactly-once delivery semantics; timeout retries and idempotent are still the responsibility of the actual receiver.

{% hint style="warning" %}
🚨 Spooler's enumeration will consume the entries in the buffer, and the sequence obtained by the refresh callback will also take out the entries as read. Don't iterate over it for "view current content", and don't count and then write a second time in the same callback. When a batch of data needs to be used multiple times, it should be materialized once in the callback and then operate on this local collection.
{% endhint %}

Throwing an exception in the callback will not automatically put the items that have been read back into the buffer. Persistent messaging or transaction mechanisms should be used when reliable delivery is required, and read [message reliability](../../messaging/reliability.md).

## Empty, Shut Down and Release

Clear removes and discards the current entry without calling the refresh callback. Dispose stops the timer and ends the channel and does not replace the final refresh required by the business. When shutting down, first stop the producer, then wait for the required refresh to complete, and finally release the owned instance; data that cannot be accepted should be persisted first.

The period should be determined based on the allowed delay, and the limit should be determined based on the time consumption of a single batch and the memory usage. The framework [Spooler.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Caching/Spooler.cs) gives specific boundaries, and the command line interaction use case is in [samples/spooler](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/samples/spooler).
