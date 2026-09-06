---
description: "HierarchyVector32 four-level hierarchical encoding structure."
icon: code-branch
---

# HierarchyVector32

`HierarchyVector32` uses a 32-bit unsigned integer to express up to four levels of hierarchical encoding, each level occupying one byte.

Source: [framework/Zongsoft.Data/test/Models/AddressConditionConverter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/test/Models/AddressConditionConverter.cs#L10) (excerpt; see source for context).

{% code title="AddressConditionConverter.cs" %}
```csharp
public override ICondition Convert(ConditionConverterContext context)
{
	if(context.Value == null)
		return null;

	static ICondition GetCondition(string name, uint id)
	{
		if(id == 0)
			return Condition.Equal(name, 0u);

		return Zongsoft.Data.Range.Create((HierarchyVector32)id).ToCondition(name);
	}

	if(context.Names.Length == 1)
		return GetCondition(context.GetFullName(), Zongsoft.Common.Convert.ConvertValue<uint>(context.Value));

	return ConditionCollection.Or(context.Names.Select(name => GetCondition(context.GetFullName(name), Zongsoft.Common.Convert.ConvertValue<uint>(context.Value))));
}
```
{% endcode %}

## Common Members

| member | Description |
| --- | --- |
| `Depth` | Current encoding depth. |
| `Value` | Raw 32-bit value. |
| `Minimum` / `Maximum` | The range of values covered by the current level. |
| `Contains` | Determines whether one encoding contains another encoding. |
| `IsChild` | Determine whether it is a direct or indirect child. |
| `GetParent` | Get the parent encoding. |
| `GetAncestors` | Get a list of ancestor encodings. |

## Practical Use: Hierarchical Range Query

Discussions forums and folders use their own data model and do not encode numbers as HierarchyVector32. The AddressConditionConverter used in the frame data test above: interprets the address number as a hierarchical vector, and then generates range query conditions through Range.Create; multi-field conditions are combined with Or. Input zeros are individually converted to equivalent conditions to avoid mistaking the root node range for a specific address.

## Coding Constraints

The four bytes correspond to the four layers from high to low. The non-zero byte at the end determines the depth, and the remaining low-order range represents the descendants; the Minimum and Maximum of the fourth layer are the same. The zero vector has a depth of zero. The constructor itself does not verify whether the previous levels are continuous, and the caller should keep the coding rules consistent.

This is suitable for regional or institutional queries where the upper limit of the level is fixed and the coding rules are uniformly assigned by the system; it is not suitable for trees with arbitrary depth and frequently moved nodes. When you need a dynamic parent-child relationship, read [hierarchical model](../collections/hierarchical.md) first, and do not force-convert ordinary auto-increment IDs into valid hierarchical codes.

## Related Resources

* [HierarchyVector32.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/HierarchyVector32.cs)
