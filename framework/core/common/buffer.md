---
description: "Buffer memory leasing, encoding, decoding and binary reading tools."
icon: code
---

# Buffer

`Buffer` provides tool methods around [`IMemoryOwner<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.buffers.imemoryowner-1) _[Source](https://source.dot.net/#System.Private.CoreLib/IMemoryOwner.cs)_, array leasing, text encoding decoding, and [`ReadOnlySequence<byte>`](https://learn.microsoft.com/en-us/dotnet/api/system.buffers.readonlysequence-1) _[Source](https://source.dot.net/#System.Memory/ReadOnlySequence.cs)_ numeric reading.

## Memory Leasing

Source: [framework/Zongsoft.Core/test/Common/BufferTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/BufferTest.cs#L10) (excerpt; see source for context).

{% code title="BufferTest.cs" %}
```csharp
public void Lease_Array_CopiesValidPrefixAndDoesNotObserveCallerMutation()
{
	var source = new[] { 1, 2, 3, 4 };
	using var owner = source.Lease(3);

	source[0] = 10;
	source[3] = 40;

	Assert.Equal(3, owner.Memory.Length);
	Assert.Equal([1, 2, 3], owner.Memory.ToArray());
}
```
{% endcode %}

The above framework regression use case verifies that Lease copies a valid prefix that will not change as the original array is subsequently modified; it cannot be regarded as a shared view of the original array. The internal array pool is returned when freed.

## Release and Ownership

Source: [framework/Zongsoft.Core/test/Common/BufferTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/BufferTest.cs#L35) (excerpt; see source for context).

{% code title="BufferTest.cs" %}
```csharp
public void Lease_Array_DisposeIsIdempotentAndMakesOwnerInaccessible()
{
	var source = new byte[] { 1, 2, 3 };
	var owner = source.Lease();

	owner.Dispose();
	owner.Dispose();

	Assert.Equal([1, 2, 3], source);
	Assert.Throws<ObjectDisposedException>(() => _ = owner.Memory);
}
```
{% endcode %}

The test verifies that the release can be called repeatedly, and the leased memory cannot be read after it is released. The encoding and decoding methods also return the owner to be released; their default text encoding is UTF-8.

## Binary Read

`Buffer` provides big-endian and little-endian read methods, such as `TryGetInt32BigEndian`, `TryGetUInt64LittleEndian`, `TryGetDoubleBigEndian`.

These methods are suitable for network protocols, binaries, and cross-platform packet parsing.

## Related Resources

* [Buffer.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/Buffer.cs)
