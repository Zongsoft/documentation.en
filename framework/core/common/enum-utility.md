---
description: "EnumUtility and EnumEntry enumeration metadata tools."
icon: list-check
---

# EnumUtility

`EnumUtility` reads the name, alias, description, and value of an enumeration item and returns it as a `EnumEntry` structure.

The following code is from the framework test, Gender is the test enumeration in Zongsoft.Tests, not Discussions.Models.Gender. The former's alias and description must be read together with the test definition.

## EnumEntry

`EnumEntry` contains metadata for the enumeration items:

| Field | Description |
| --- | --- |
| [`Type`](https://learn.microsoft.com/en-us/dotnet/api/system.type) _[Source](https://source.dot.net/#System.Private.CoreLib/Type.cs)_ | Enumeration type. |
| `Name` | Enumeration item name. |
| `Value` | Enumeration item value; you can select an enumeration value or a basic value. |
| `Aliases` | Enumeration item alias. |
| `Description` | Enumeration item description. |

## Read Enumeration Items

Source: [framework/Zongsoft.Core/test/Common/EnumUtilityTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/EnumUtilityTest.cs#L12) (excerpt; see source for context).

{% code title="EnumUtilityTest.cs" %}
```csharp
public void TestGetEnumEntry()
{
	var entry = EnumUtility.GetEnumEntry(Gender.Female);

	Assert.Equal("Female", entry.Name);
	Assert.Equal(Gender.Female, entry.Value); //注意：entry.Value 为枚举类型
	Assert.True(entry.HasAlias("F"));
	Assert.Equal("女士", entry.Description);
	Assert.Equal("女士", EnumUtility.GetEnumDescription(Gender.Female));

	entry = EnumUtility.GetEnumEntry(Gender.Male, true);

	Assert.Equal("Male", entry.Name);
	Assert.Equal((byte)1, entry.Value); //注意：entry.Value 为枚举项的基元类型
	Assert.True(entry.HasAlias("M"));
	Assert.Equal("男士", entry.Description);
	Assert.Equal("男士", EnumUtility.GetEnumDescription(Gender.Male));
}
```
{% endcode %}

`GetEnumEntries` can read the entire enumeration type, and can also append null value items to nullable enumerations.

Source: [framework/Zongsoft.Core/test/Common/EnumUtilityTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/EnumUtilityTest.cs#L32) (excerpt; see source for context).

{% code title="EnumUtilityTest.cs" %}
```csharp
public void TestGetEnumEntries()
{
	var entries = EnumUtility.GetEnumEntries(typeof(Gender), true);

	Assert.Equal(2, entries.Length);
	Assert.Contains(entries, entry => entry.Name == "Male");
	Assert.Contains(entries, entry => entry.Name == "Female");

	entries = EnumUtility.GetEnumEntries(typeof(Nullable<Gender>), true, null, "<Unknown>");

	Assert.Equal(3, entries.Length);
	Assert.Equal("", entries[0].Name);
	Assert.Null(entries[0].Value);
	Assert.Equal("<Unknown>", entries[0].Description);

	Assert.Contains(entries, entry => entry.Name == "Male");
	Assert.Contains(entries, entry => entry.Name == "Female");
}
```
{% endcode %}

## Related Resources

* [EnumUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/EnumUtility.cs)
* [EnumEntry.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/EnumEntry.cs)
* [EnumUtilityTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/EnumUtilityTest.cs)
