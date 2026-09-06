---
description: "Enumerable, CollectionUtility, DictionaryUtility, and collection extension methods."
icon: wrench
---

# Collection Extensions

Collection extension types provide a set of low-level utility methods for handling synchronous/asynchronous enumeration, reflective collection operations, dictionary conversions, key-value pair construction, and synchronization packaging.

## Type Relationship

| Type | Description |
| --- | --- |
| `Enumerable` | Synchronous/asynchronous enumeration adaptation, empty enumeration, single value enumeration and typed enumeration. |
| `CollectionUtility` | Perform reflective `Add` and `Remove` on unknown collection objects. |
| `DictionaryUtility` | Perform reflective `Add`, `Remove` and key extraction on unknown dictionary objects. |
| `KeyValuePairExtension` | Create an array of key-value pairs and convert between an array of key-value pairs and an array of objects. |
| `DictionaryExtension` | Dictionary reading, conversion, synchronization packaging and enumeration to dictionary items. |
| `HashSetExtension` | Copy [`HashSet<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1) _[Source](https://source.dot.net/#System.Private.CoreLib/HashSet.cs)_ as an array. |
| `ListUtility` | Wrap [`IList<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ilist-1) _[Source](https://source.dot.net/#System.Private.CoreLib/IList.cs)_ as `SynchronizedList<T>`. |

## Enumerable

`Enumerable` mainly handles the scenario where "the object may be a value, a synchronous enumeration, or an asynchronous enumeration".

Source: [framework/Zongsoft.Core/test/Collections/EnumerableTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/EnumerableTest.cs#L241) (excerpt; see source for context).

{% code title="EnumerableTest.cs" %}
```csharp
public async Task EnumerateAdapters_PrioritizeCollectionsAndConvertScalarValues()
{
	var items = new List<int>([1, 2]);
	var syncObjects = Enumerable.Enumerate<object>(items);
	var asyncObjects = Enumerable.EnumerateAsync<object>(items);
	var syncDecimals = Enumerable.Enumerate<decimal>(10);
	var asyncDecimals = Enumerable.EnumerateAsync<decimal>(10);

	Assert.Collection(syncObjects,
		item => Assert.Equal(1, item),
		item => Assert.Equal(2, item));
	Assert.Collection(await CollectAsync(asyncObjects),
		item => Assert.Equal(1, item),
		item => Assert.Equal(2, item));
	Assert.Equal([10m], syncDecimals);
	Assert.Equal([10m], await CollectAsync(asyncDecimals));
}
```
{% endcode %}

Common capabilities include:

* `Empty(Type)`: Creates an empty enumeration by runtime type.
* `Empty<T>()`: Creates an empty asynchronous enumeration.
* `First<T>()`, `FirstOrDefault<T>()`: Get the first element asynchronously, and pass the cancellation and release enumerators; the former throws an exception when the sequence is empty, and the latter returns the default value of the element type. For actual usage of Discussions, see [Asynchronous query](../../data/querying.md).
* `Asynchronize<T>()`: Wrap synchronous enumeration into asynchronous enumeration.
* `Synchronize<T>()`: Convert asynchronous enumeration to blocking synchronous enumeration.
* `Enumerate<T>(object)`: Convert the object to [`IEnumerable<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1) _[Source](https://source.dot.net/#System.Private.CoreLib/IEnumerable.cs)_.
* `EnumerateAsync<T>(object)`: Convert the object to [`IAsyncEnumerable<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iasyncenumerable-1) _[Source](https://source.dot.net/#System.Private.CoreLib/IAsyncEnumerable.cs)_.

These methods preferentially identify synchronous or asynchronous collections as sequences, and only non-collection objects are type-converted according to single values; single-value enumerators only borrow the object, and releasing the enumerator does not release the element itself.

`Asynchronize<T>` and `EnumerateAsync<T>` only perform enumeration form adaptation and will not arrange synchronized enumeration to background threads. After the asynchronous enumerator receives the cancellation mark, it will throw [`OperationCanceledException`](https://learn.microsoft.com/en-us/dotnet/api/system.operationcanceledexception) _[Source](https://source.dot.net/#System.Private.CoreLib/OperationCanceledException.cs)_; when `EnumerateAsync<T>` receives both method-level and enumerator-level cancellation marks, either cancellation will terminate the enumeration. `Synchronize<T>` will block the current thread waiting for asynchronous enumeration. `await foreach` should be used directly in the asynchronous call chain.

When the source sequence implements `IPageable`, the adaptation results of `Asynchronize<T>`, `EnumerateAsync<T>` and `Synchronize<T>` continue to implement `IPageable`, and forward the dynamic `Suppressed` state and `Paginated` event; the event sender is the outer result object held by the caller.

## CollectionUtility

`CollectionUtility` is used in scenarios where the target object type is undefined, but may implement [`ICollection<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.icollection-1) _[Source](https://source.dot.net/#System.Private.CoreLib/ICollection.cs)_ or the non-generic [`IList`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.ilist) _[Source](https://source.dot.net/#System.Private.CoreLib/IList.cs)_.

Source: [framework/Zongsoft.Core/test/Collections/CollectionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/CollectionTest.cs#L12) (excerpt; see source for context).

{% code title="CollectionTest.cs" %}
```csharp
public void TestGenericCollection()
{
	object list = new List<int>();

	Assert.True(CollectionUtility.TryAdd(list, 1));
	Assert.NotEmpty((ICollection<int>)list);
	Assert.Equal(1, ((IList<int>)list)[0]);

	Assert.True(CollectionUtility.TryAdd(list, 2));
	Assert.Equal(2, ((IList<int>)list).Count);
	Assert.Equal(1, ((IList<int>)list)[0]);
	Assert.Equal(2, ((IList<int>)list)[1]);

	Assert.True(CollectionUtility.TryRemove(list, 1));
	Assert.Single((IList<int>)list);
	Assert.True(CollectionUtility.TryRemove(list, 2));
	Assert.Empty((IList<int>)list);
}
```
{% endcode %}

It will try to convert the incoming value to the collection element type, and then call the corresponding `Add` or `Remove`.

## DictionaryUtility

`DictionaryUtility` is used in scenarios where the target object type is undefined, but may implement [`IDictionary<TKey, TValue>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.idictionary-2) _[Source](https://source.dot.net/#System.Private.CoreLib/IDictionary.cs)_ or the non-generic [`IDictionary`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.idictionary) _[Source](https://source.dot.net/#System.Private.CoreLib/IDictionary.cs)_.

Source: [framework/Zongsoft.Core/test/Collections/DictionaryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/DictionaryTest.cs#L12) (excerpt; see source for context).

{% code title="DictionaryTest.cs" %}
```csharp
public void TestGenericDictionary()
{
	object dictionary = new Dictionary<string, int>();

	Assert.True(DictionaryUtility.TryAdd(dictionary, "K1", 1));
	Assert.NotEmpty((IDictionary<string, int>)dictionary);
	Assert.Equal(1, ((IDictionary<string, int>)dictionary)["K1"]);

	Assert.True(DictionaryUtility.TryAdd(dictionary, "K2", 2));
	Assert.Equal(2, ((IDictionary<string, int>)dictionary).Count);
	Assert.Equal(1, ((IDictionary<string, int>)dictionary)["K1"]);
	Assert.Equal(2, ((IDictionary<string, int>)dictionary)["K2"]);

	Assert.True(DictionaryUtility.TryRemove(dictionary, "K1"));
	Assert.Single((IDictionary<string, int>)dictionary);
	Assert.True(DictionaryUtility.TryRemove(dictionary, "K2"));
	Assert.Empty((IDictionary<string, int>)dictionary);
}
```
{% endcode %}

`TryGetEntry` can extract keys and values from [`DictionaryEntry`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.dictionaryentry) _[Source](https://source.dot.net/#System.Private.CoreLib/DictionaryEntry.cs)_ or [`KeyValuePair<TKey, TValue>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.keyvaluepair-2) _[Source](https://source.dot.net/#System.Private.CoreLib/KeyValuePair.cs)_.

## Dictionary Expansion

`DictionaryExtension` provides `TryGetValue`, dictionary type conversion and synchronization wrappers for non-generic dictionaries.

These extensions can only be used by reflection calls such as data service and configuration binding. See [DictionaryExtension source code](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/DictionaryExtension.cs) for specific conversion rules.

A generic dictionary can be wrapped by `Synchronize` into `SynchronizedDictionary<TKey, TValue>`.

## Key-value Pairs and Hash Set Extensions

`KeyValuePairExtension.CreatePairs` can create a key-value pair array based on a key name array and a value array.

When a key-value pair needs to be constructed, the key array and value array must be organized in the same order; see [KeyValuePairExtension source code](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/KeyValuePairExtension.cs) for their length check and combination method.

`HashSetExtension.ToArray` will create a new array and call `HashSet<T>.CopyTo`, which is suitable for scenarios that require array snapshots.

## Related Resources

* [Enumerable.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/Enumerable.cs)
* [CollectionUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/CollectionUtility.cs)
* [DictionaryUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/DictionaryUtility.cs)
* [KeyValuePairExtension.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/KeyValuePairExtension.cs)
* [DictionaryExtension.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/DictionaryExtension.cs)
* [HashSetExtension.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/HashSetExtension.cs)
* [ListUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/ListUtility.cs)
* [EnumerableTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/EnumerableTest.cs)
* [DictionaryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/DictionaryTest.cs)
