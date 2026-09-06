---
description: "Locker Synchronous and asynchronous mutex locks."
icon: shield
---

# Locker

`Locker` is a lightweight mutex wrapper used to serialize critical sections in synchronous code and asynchronous code.

Both examples are taken from LockerTest; COUNT is the number of cycles of 500 defined in the fixture. The test verifies the mutual exclusion effect by concurrently incrementing the final count, rather than guessing the execution order with delays.

## Synchronous Critical Section

The synchronization code uses `Lock()` to obtain a releasable lock handle, which is automatically released when leaving the scope of `using`.

Source: [framework/Zongsoft.Core/test/Common/LockerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/LockerTest.cs#L135) (excerpt; see source for context).

{% code title="LockerTest.cs" %}
```csharp
public void Lock()
{
	var count = 0;
	var locker = new Locker();

	Parallel.For(0, COUNT, i =>
	{
		using(locker.Lock())
		{
			count++;
		}
	});

	Assert.Equal(COUNT, count);
}
```
{% endcode %}

## Asynchronous Critical Section

Asynchronous code uses `LockAsync()` and releases the lock with `await using`.

Source: [framework/Zongsoft.Core/test/Common/LockerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/LockerTest.cs#L189) (excerpt; see source for context).

{% code title="LockerTest.cs" %}
```csharp
public async Task LockAsync2()
{
	const int TIMES = 50;

	var count = 0;
	var locker = new Locker();

	await Parallel.ForAsync(0, TIMES, async (_, cancellation) =>
	{
		for(int i = 0; i < COUNT; i++)
		{
			await using(await locker.LockAsync(cancellation))
			{
				count++;
			}
		}
	});

	Assert.Equal(COUNT * TIMES, count);
}
```
{% endcode %}

LockAsync supports cancellation tokens, which is suitable for background tasks that need to cancel waiting for locks.

{% hint style="info" %}
If you need to create a change token, see [Notification](notification.md); if you need to express why the operation failed, see [OperationException](operation-exception.md).
{% endhint %}

## Related Resources

* [Locker.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/Locker.cs)
* [LockerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/LockerTest.cs)
