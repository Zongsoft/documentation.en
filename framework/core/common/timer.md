---
description: "Understand periodic callback, cancellation, stop and release through the existing TimerTest in the framework."
icon: clock
---

# Timer

Timer is used to call asynchronous callback periodically. It is suitable for lightweight polling within the host; cross-process persistence scheduling and failure retries should read [Task scheduling](../scheduling.md). Discussions There is no business example that directly uses timers. The existing tests of the framework are used here.

## Create Callbacks and Completion Signals

TimerTest sets the period to 1 millisecond and waits for the 10th callback with a completion signal. LIMIT, _count, _timer and _completion are all members of this test fixture; the period is a test parameter, and the actual task needs to be configured based on the work time.

Source: [framework/Zongsoft.Core/test/Common/TimerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TimerTest.cs#L22) (excerpt; see source for context).

{% code title="TimerTest.cs" %}
```csharp
public TimerTest()
{
	_timer = new Timer(TimeSpan.FromMilliseconds(1), this.OnTick);
	_completion = new TaskCompletionSource(TaskCreationOptions.RunContinuationsAsynchronously);
}
```
{% endcode %}

## Start and Wait for Completion

Source: [framework/Zongsoft.Core/test/Common/TimerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TimerTest.cs#L31) (excerpt; see source for context).

{% code title="TimerTest.cs" %}
```csharp
public async Task Test()
{
	Assert.False(_timer.IsRunning);
	_timer.Start(TestContext.Current.CancellationToken);
	Assert.True(_timer.IsRunning);

	await _completion.Task.WaitAsync(TimeSpan.FromSeconds(30), TestContext.Current.CancellationToken);
	Assert.Equal(LIMIT, _count);
	Assert.False(_timer.IsRunning);
}
```
{% endcode %}

The test checks IsRunning before start, after start, and after end. The wait limit of 30 seconds is used to prevent failed tests from hanging indefinitely and does not represent timer accuracy or business timeout commitments.

## Stop in Callback

Source: [framework/Zongsoft.Core/test/Common/TimerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TimerTest.cs#L44) (excerpt; see source for context).

{% code title="TimerTest.cs" %}
```csharp
private ValueTask OnTick(object state, CancellationToken cancellation)
{
	if(Interlocked.Increment(ref _count) >= LIMIT)
	{
		_timer.Stop();
		_completion.TrySetResult();
	}

	return ValueTask.CompletedTask;
}
```
{% endcode %}

After the callback reaches LIMIT, Stop is called and the waiting party is notified. Stopping the cycle does not mean undoing the business side effects that have occurred; external operations in callbacks should still comply with their own cancellation and idempotent conventions.

{% hint style="info" %}
💡 The component that created the timer should be responsible for eventually releasing it. On target frameworks that support runtime setting of Period, subsequent periods can be adjusted; see [Timer.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/Timer.cs) for the actual conditional compilation and release logic.
{% endhint %}
