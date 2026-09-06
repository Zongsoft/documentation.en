---
description: "BitVector32 and BitVector64 bit tag structures."
icon: code
---

# BitVector

`BitVector32` and `BitVector64` are used to compress multiple Boolean states into an integer value, suitable for saving low-cost tags such as flag bits, status bits and permission bits.

## Type Relationship

| Type | Description |
| --- | --- |
| `BitVector32` | Saves a set of bit tags using a 32-bit integer. |
| `BitVector64` | Saves a set of bit tags using a 64-bit integer. |

## BitVector32 and BitVector64

A bit vector can compress multiple Boolean states into a single integer. The `bit` parameter in the indexer represents a bit mask, not a zero-based sequence number.

Source: [framework/Zongsoft.Core/test/Common/ConvertTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/ConvertTest.cs#L94) (excerpt; see source for context).

{% code title="ConvertTest.cs" %}
```csharp
public void TestBitVector32()
{
	Zongsoft.Common.BitVector32 vector = 1;

	Assert.Equal(1, vector.Data);
	Assert.True(vector[1]);
	Assert.False(vector[2]);
	Assert.False(vector[3]);
	Assert.False(vector[4]);
	Assert.False(vector[5]);

	vector[5] = true;
	Assert.Equal(5, vector.Data);
	Assert.True(vector[1]);
	Assert.False(vector[2]);
	Assert.False(vector[3]);
	Assert.True(vector[4]);
	Assert.True(vector[5]);
}
```
{% endcode %}

`BitVector64` can be implicitly converted from `BitVector32`, which is suitable for scenarios that require more flag bits.

{% hint style="info" %}
`BitVector32` and `BitVector64` focus more on compact storage; if you want to express a fixed-level encoding, use [HierarchyVector32](hierarchy-vector32.md).
{% endhint %}

The above is the original bit vector test of the framework ConvertTest; the numerical value represents the bit mask, and the index parameter cannot be directly understood as a bit sequence number starting from zero. Discussions does not call this structure directly.

## Related Resources

* [BitVector32.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/BitVector32.cs)
* [BitVector64.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/BitVector64.cs)
