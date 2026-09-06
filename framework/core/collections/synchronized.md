---
description: "SynchronizedList and SynchronizedDictionary synchronized collections."
icon: shield
---

# Synchronized Collections

A synchronized collection is used to wrap a regular list or dictionary and protect it from concurrent access via read-write locks. They are suitable for scenarios such as a small amount of shared state within the framework, cache indexes, registries, descriptor collections, etc.

## Type Relationship

| Type | Description |
| --- | --- |
| `SynchronizedList<T>` | Thread security list, implementing [`IList<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ilist-1) _[Source](https://source.dot.net/#System.Private.CoreLib/IList.cs)_, [`ICollection<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.icollection-1) _[Source](https://source.dot.net/#System.Private.CoreLib/ICollection.cs)_ and the non-generic [`ICollection`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.icollection) _[Source](https://source.dot.net/#System.Private.CoreLib/ICollection.cs)_. |
| `SynchronizedDictionary<TKey, TValue>` | Thread security dictionary, implements [`IDictionary<TKey, TValue>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.idictionary-2) _[Source](https://source.dot.net/#System.Private.CoreLib/IDictionary.cs)_, and provides a concurrent-style upsert method. |
| `ListUtility` | Provides `IList<T>.Synchronize()` packaging method. |
| `DictionaryExtension` | Provides `IDictionary<TKey, TValue>.Synchronize()` packaging method. |

## SynchronizedList

`SynchronizedList<T>` uses read-write locks to protect reads and writes. Read operations enter the read lock, write operations enter the write lock, and the read lock is also maintained during enumeration.

Source: [framework/Zongsoft.Core/test/Collections/SynchronizedListTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/SynchronizedListTest.cs#L13) (excerpt; see source for context).

{% code title="SynchronizedListTest.cs" %}
```csharp
public void Add()
{
	const int COUNT = 1_0000;

	var list = new SynchronizedList<int>(COUNT);

	Parallel.For(0, COUNT, index =>
	{
		list.Add(index);

		if(index % 100 == 0)
		{
			foreach(var item in list)
			{
				Assert.True(item >= 0);
			}

			for(int i = 0; i < list.Count; i++)
			{
				Assert.True(list[i] >= 0);
			}
		}
	});

	Assert.Equal(COUNT, list.Count);
	var hashset = new HashSet<int>(list);
	Assert.Equal(COUNT, hashset.Count);
}
```
{% endcode %}

You can also use extension methods to wrap existing lists.

The packaging entrance of the existing list is ListUtility.Synchronize; after packaging, it should be uniformly accessed through the returned object. If the shared code continues to directly modify the original list, the lock will be bypassed.

{% hint style="warning" %}
A read lock is held when enumerating `SynchronizedList<T>`. Do not perform operations inside an enumeration loop that may block for a long time, and do not write to the list during the same enumeration.
{% endhint %}

## SynchronizedDictionary

In addition to regular dictionary operations, `SynchronizedDictionary<TKey, TValue>` also provides `TryAdd`, `GetOrAdd`, `AddOrUpdate`, and `TryUpdate` methods similar to concurrent dictionaries.

Source: [framework/Zongsoft.Core/test/Collections/SynchronizedDictionaryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/SynchronizedDictionaryTest.cs#L139) (excerpt; see source for context).

{% code title="SynchronizedDictionaryTest.cs" %}
```csharp
public void GetOrAdd()
{
	const int COUNT = 1_0000;

	var keys = System.Linq.Enumerable.Range(0, COUNT).ToArray();
	Random.Shared.Shuffle(keys);

	var dictionary = new SynchronizedDictionary<int, string>(COUNT);
	for(int i = 0; i < COUNT; i++)
	{
		if(i % 2 == 1)
			dictionary[i] = $"Value#{i}";
	}

	Assert.Equal(COUNT / 2, dictionary.Count);

	Parallel.For(0, COUNT, index =>
	{
		var key = keys[index];
		var value = dictionary.GetOrAdd(key, key => $"Added#{key}");

		if(key % 2 == 0)
			Assert.Equal($"Added#{key}", value);
		else
			Assert.Equal($"Value#{key}", value);

		if(index % 100 == 0)
		{
			foreach(var item in dictionary)
			{
				Assert.True(item.Key >= 0);
				Assert.True(dictionary.ContainsKey(item.Key));
			}
		}
	});

	Assert.Equal(COUNT, dictionary.Count);
}
```
{% endcode %}

`GetOrAdd` and `AddOrUpdate` use upgradeable read locks to reduce write lock scope. `Keys` and `Values` return snapshot collections to prevent the caller from getting the internal collection view.

## Select Suggestions

| scene | Suggestions |
| --- | --- |
| Requires complete [`IList<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ilist-1) _[Source](https://source.dot.net/#System.Private.CoreLib/IList.cs)_ semantics and controls concurrent access | Use `SynchronizedList<T>`. |
| Requires complete [`IDictionary<TKey, TValue>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.idictionary-2) _[Source](https://source.dot.net/#System.Private.CoreLib/IDictionary.cs)_ semantics and conditional updates | Use `SynchronizedDictionary<TKey, TValue>`. |
| High-throughput lock-free or fine-grained concurrent dictionary scenarios | Evaluate the .NET official concurrency collection first. |

## Related Resources

* [SynchronizedList.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/SynchronizedList.cs)
* [SynchronizedDictionary.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/SynchronizedDictionary.cs)
* [SynchronizedListTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/SynchronizedListTest.cs)
* [SynchronizedDictionaryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/SynchronizedDictionaryTest.cs)
