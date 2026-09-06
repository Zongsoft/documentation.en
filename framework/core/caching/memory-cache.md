---
description: "MemoryCache in-process cache, expiration policy, dependency tokens, and eviction events."
icon: database
---

# MemoryCache

`MemoryCache` is an in-process cache implementation in the `Zongsoft.Caching` namespace. It wraps [`Microsoft.Extensions.Caching.Memory.MemoryCache`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.memorycache) _[Source](https://source.dot.net/#Microsoft.Extensions.Caching.Memory/MemoryCache.cs)_ and adds Zongsoft-style expiration descriptions, dependency tokens, priorities, obsolescence events, and quantity limit reminders.

{% hint style="info" %}
`MemoryCache` is suitable for caching in-process reconstructable data, such as metadata, descriptors, parsing results, or lightweight objects. Cross-process sharing, distributed consistency, and inter-service cache synchronization should use the concrete implementation of `IDistributedCache`.
{% endhint %}

## Type Relationship

| Type | Description |
| --- | --- |
| `MemoryCache` | In-process cache container providing read, write, delete, get or create, cleanup and event notification. |
| `MemoryCacheOptions` | Configure scan frequency and quantity limits; `MemoryCache.Shared` uses immutable options. |
| `MemoryCacheScanner` | Periodically call `Compact(0)` to trigger expired item scanning. |
| `MemoryCache.Expiration` | Describes sliding expiration, absolute expiration, or a combination of both. |
| `CachePriority` | Maps to underlying cache item priorities: `Low`, `Normal`, `High`, `NeverRemove`. |
| `CacheEvictedEventArgs` | `Evicted` event parameters, including key, value, elimination reason and status object. |
| `CacheLimitedEventArgs` | `Limited` event parameters, including the exceeded quantity and current quantity. |

## Create Cache

Discussions Indirectly use the framework's cache reuse mechanism through data accessors. The independent demonstration of the cache itself comes from Core's memorycache interactive program: frequency of one second, sliding expiration of thirty seconds, and quantity reminder threshold. You can create independent cache instances directly or use shared instances.

Source: [framework/Zongsoft.Core/samples/memorycache/Program.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/samples/memorycache/Program.cs#L13) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
const int FREQUENCY  = 1;
const int EXPIRATION = 30;
const int LIMIT      = 5;

using var cache = new MemoryCache(TimeSpan.FromSeconds(FREQUENCY), LIMIT);
using var scanner = new MemoryCacheScanner(cache);

cache.Limited += Cache_Limited;
cache.Evicted += Cache_Evicted;
```
{% endcode %}

`MemoryCache.Shared` is a global shared instance, and its `Options` is immutable and cannot be modified at runtime.

## Read and Write

`SetValue` is used to write cache entries directly, `GetValue` and `TryGetValue` are used to read cache entries, and `Remove` is used to delete cache entries.

Source: [framework/Zongsoft.Core/test/Caching/MemoryCacheTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Caching/MemoryCacheTest.cs#L14) (excerpt; see source for context).

{% code title="MemoryCacheTest.cs" %}
```csharp
public void Test()
{
	object value;

	var cache = new MemoryCache();
	Assert.Equal(0, cache.Count);
	Assert.False(cache.Contains("KEY"));
	Assert.False(cache.Remove("KEY", out _));
	Assert.False(cache.TryGetValue("KEY", out _));

	value = cache.GetOrCreate("K1", () => "V1");
	Assert.NotNull(value);
	Assert.True(cache.Contains("K1"));
	Assert.Equal("V1", value);
	Assert.True(cache.Remove("K1", out value));
	Assert.Equal("V1", value);
	Assert.Equal(0, cache.Count);

	const int COUNT = 10000;
	Parallel.For(0, COUNT, index => cache.SetValue($"KEY#{index}", $"Value#{index}@{Environment.CurrentManagedThreadId}"));
	Assert.Equal(COUNT, cache.Count);
}
```
{% endcode %}

Use `GetOrCreate` or `GetOrCreateAsync` when "create without" semantics are required. These methods call the factory when a read misses; a concurrent miss does not mean that the factory must be executed only once, and it should not be relied upon to complete the only write in the business. Here is the real purpose of the data accessor provider: cache the created accessor and use its Disposed notification as a cache invalidation dependency.

Source: [framework/Zongsoft.Core/src/Data/DataAccessProviderBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Data/DataAccessProviderBase.cs#L52) (excerpt; see source for context).

{% code title="DataAccessProviderBase.cs" %}
```csharp
public TDataAccess GetAccessor(string name, IDataAccessOptions options = null)
{
	if(string.IsNullOrEmpty(name) || options == null || options.Settings == null || !options.Settings.Any())
		name = GetName(name);

	return _accesses.GetOrCreate(name, key =>
	{
		var accessor = this.CreateAccessor(name, options);
		return (accessor, accessor.Disposed);
	});
}
```
{% endcode %}

## Expiration Policy

`MemoryCache.Expiration` can express sliding expiration, absolute expiration, or both at the same time. The following table is a parameter form reference; the following actual example writes the terminal input text into the Key# sequence number, and uses the EXPIRATION constant and the Now attribute of the same type to record the status.

| Writing method | Description |
| --- | --- |
| `TimeSpan.FromMinutes(10)` | Sliding expiration means that cache items will expire only if they are not accessed within a specified period of time. |
| `DateTimeOffset.UtcNow.AddHours(1)` | Absolute expiration means that the cached item expires after the specified time point. |
| `(TimeSpan.FromMinutes(10), DateTimeOffset.UtcNow.AddHours(1))` | Set sliding expiration and absolute expiration at the same time. |

Source: [framework/Zongsoft.Core/samples/memorycache/Program.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/samples/memorycache/Program.cs#L72) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
cache.SetValue($"Key#{++count}", text, TimeSpan.FromSeconds(EXPIRATION), Now);
```
{% endcode %}

## Dependency Token

Cache entries can depend on [`IChangeToken`](https://source.dot.net/#Microsoft.Extensions.Primitives/IChangeToken.cs). When the token changes, the cache item is marked as invalid and the eviction callback is triggered.

Source: [framework/Zongsoft.Core/test/Caching/MemoryCacheTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Caching/MemoryCacheTest.cs#L38) (excerpt; see source for context).

{% code title="MemoryCacheTest.cs" %}
```csharp
public void TestDependency()
{
	var cache = new MemoryCache();
	cache.Evicted += this.Cache_Evicted;

	var cancellation = new CancellationTokenSource();
	var value = cache.GetOrCreate("KEY", key =>
	{
		return ("Value1", new CancellationChangeToken(cancellation.Token));
	});

	Assert.NotNull(value);
	Assert.Equal("Value1", value);
	Assert.Equal(1, cache.Count);

	//通知缓存项过期
	cancellation.Cancel();
	Assert.False(cache.Contains("KEY"));
	Assert.Equal(0, cache.Count);

	//清理缓存
	cache.Compact();

	//等待缓存项过期事件的触发
	Assert.True(SpinWait.SpinUntil(() => Volatile.Read(ref _reason) >= 0, 10_000), "等待缓存项过期事件回调超时。");

	//确认缓存过期的原因
	Assert.Equal(CacheEvictedReason.Depended, (CacheEvictedReason)Volatile.Read(ref _reason));

	//创建一个已经过期的缓存项
	value = cache.GetOrCreate("KEY", key =>
	{
		return ("Value2", new CancellationChangeToken(cancellation.Token));
	});

	Assert.NotNull(value);
	Assert.Equal("Value2", value);
	Assert.False(cache.Contains("KEY"));
	Assert.Equal(0, cache.Count);

	//创建一个依赖失效的缓存项
	value = cache.GetOrCreate("KEY", key =>
	{
		return ("Value3", Common.Notification.GetToken());
	});

	Assert.NotNull(value);
	Assert.Equal("Value3", value);
	Assert.False(cache.Contains("KEY"));
	Assert.Equal(0, cache.Count);
}
```
{% endcode %}

[`CancellationChangeToken`](https://source.dot.net/#Microsoft.Extensions.Primitives/CancellationChangeToken.cs) in the example will convert the cancellation token into a cache dependency token.

The elimination reason corresponding to dependency failure is `CacheEvictedReason.Depended`.

## Events

`MemoryCache` provides two events. The TestDependency above relies on the Cache_Evicted callback of the same test class to log the cause and wait for asynchronous notification; the entire fixture should be retained when reusing tests. Interactive examples actively clear in the Limited callback: this is the cleanup strategy selected by the example and is not automatically executed within the cache.

The meaning of the event is as follows:

| Events | Trigger time |
| --- | --- |
| `Evicted` | Cache items are evicted due to expiration, dependency failure, deletion, replacement, or capacity pressure. |
| `Limited` | After setting or creating a cache entry, the current number exceeds `MemoryCacheOptions.CountLimit`. |

Source: [framework/Zongsoft.Core/samples/memorycache/Program.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/samples/memorycache/Program.cs#L96) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
private static void Cache_Limited(object sender, CacheLimitedEventArgs e)
{
	var content = CommandOutletContent.Create(CommandOutletColor.Magenta, "** Limited **\t")
		.Append(CommandOutletColor.DarkYellow, e.Limit.ToString())
		.Append(CommandOutletColor.DarkGray, "/")
		.Append(CommandOutletColor.DarkYellow, e.Count.ToString());

	Terminal.WriteLine(content);

	//清空缓存
	((MemoryCache)sender).Clear();
}
```
{% endcode %}

{% hint style="warning" %}
`CountLimit` is the alert threshold at the Zongsoft level. When the threshold is exceeded, the `Limited` event will be triggered, but the cache items will not be automatically deleted; the application needs to call `Compact`, `Clear` or adjust the cache write strategy according to the scenario.
{% endhint %}

## Clean and Scan

`Compact` is used to trigger underlying cache cleanup. Because the cache item is not necessarily removed from the underlying container immediately after it expires or the dependency becomes invalid, actively calling `Compact(0)` can prompt the cache to perform a scan.

`MemoryCacheScanner` encapsulates a timer and calls `Compact(0)` according to the `MemoryCacheOptions.ScanFrequency` cycle.

Source: [framework/Zongsoft.Core/src/Caching/MemoryCacheScanner.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Caching/MemoryCacheScanner.cs#L52) (excerpt; see source for context).

{% code title="MemoryCacheScanner.cs" %}
```csharp
public void Start() => _timer.Change(TimeSpan.Zero, _cache.Options.ScanFrequency);
public void Stop() => _timer.Change(Timeout.Infinite, Timeout.Infinite);
```
{% endcode %}

## Related Resources

* [MemoryCache.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Caching/MemoryCache.cs)
* [MemoryCacheOptions.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Caching/MemoryCacheOptions.cs)
* [MemoryCacheScanner.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Caching/MemoryCacheScanner.cs)
* [MemoryCacheTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Caching/MemoryCacheTest.cs)
