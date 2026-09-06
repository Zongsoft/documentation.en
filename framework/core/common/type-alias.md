---
description: "TypeAlias type aliases, generic types, nullable types, and array type resolution."
icon: book
---

# TypeAlias

`TypeAlias` is used to convert between types and readable strings. It supports built-in type aliases, nullable types, arrays, generic types, assembly shorthand, and nested types.

## Common Aliases

`TypeAlias` provides short names for common CLR types, such as:

| Type | Alias example |
| --- | --- |
| `System.Object` | `object` |
| `System.String` | `string` |
| `System.Int32` | `int32`、`int` |
| `System.Guid` | `guid` |
| `System.DateOnly` | `date`、`dateOnly` |
| `System.TimeOnly` | `time`、`timeOnly` |
| `System.TimeSpan` | `timeSpan` |

## Parsing Type

Source: [framework/Zongsoft.Core/test/Common/TypeAliasTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TypeAliasTest.cs#L15) (excerpt; see source for context).

{% code title="TypeAliasTest.cs" %}
```csharp
Assert.Same(typeof(void), TypeAlias.Parse("void"));
Assert.Same(typeof(object), TypeAlias.Parse(" object"));
Assert.Same(typeof(object[]), TypeAlias.Parse("object []"));
Assert.Same(typeof(object), TypeAlias.Parse("System.object"));
Assert.Same(typeof(object[]), TypeAlias.Parse("System.object []"));

Assert.Same(typeof(string), TypeAlias.Parse(" string "));
Assert.Same(typeof(string[]), TypeAlias.Parse("string[ ]"));
Assert.Same(typeof(string), TypeAlias.Parse("System.string"));
Assert.Same(typeof(string[]), TypeAlias.Parse("System.string [ ]"));

Assert.Same(typeof(int), TypeAlias.Parse(" int "));
Assert.Same(typeof(int), TypeAlias.Parse("int32"));
Assert.Same(typeof(int), TypeAlias.Parse("System.Int32"));
Assert.Same(typeof(int?), TypeAlias.Parse("int? "));
Assert.Same(typeof(int[]), TypeAlias.Parse("int [ ] "));
Assert.Same(typeof(int?[]), TypeAlias.Parse(" int?[]"));
```
{% endcode %}

Complex types support generics and assembly shorthand.

Source: [framework/Zongsoft.Core/test/Common/TypeAliasTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TypeAliasTest.cs#L107) (excerpt; see source for context).

{% code title="TypeAliasTest.cs" %}
```csharp
public void TestParseNested()
{
	Assert.Same(typeof(NestedClass), TypeAlias.Parse("Zongsoft.Common.Tests.TypeAliasTest+NestedClass@Zongsoft.Core.Tests"));
	Assert.Same(typeof(NestedStruct), TypeAlias.Parse("Zongsoft.Common.Tests.TypeAliasTest+NestedStruct@Zongsoft.Core.Tests"));

	Assert.Same(typeof(NestedClass.DeepenedClass), TypeAlias.Parse("Zongsoft.Common.Tests.TypeAliasTest+NestedClass+DeepenedClass@Zongsoft.Core.Tests"));
	Assert.Same(typeof(NestedClass.DeepenedStruct), TypeAlias.Parse("Zongsoft.Common.Tests.TypeAliasTest+NestedClass+DeepenedStruct@Zongsoft.Core.Tests"));

	Assert.Same(typeof(NestedStruct.DeepenedClass), TypeAlias.Parse("Zongsoft.Common.Tests.TypeAliasTest+NestedStruct+DeepenedClass@Zongsoft.Core.Tests"));
	Assert.Same(typeof(NestedStruct.DeepenedStruct), TypeAlias.Parse("Zongsoft.Common.Tests.TypeAliasTest+NestedStruct+DeepenedStruct@Zongsoft.Core.Tests"));
}
```
{% endcode %}

`@AssemblyName` is the abbreviation of the assembly name, which is equivalent to the common comma assembly qualified form.

## Generate Alias

Source: [framework/Zongsoft.Core/test/Common/TypeAliasTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TypeAliasTest.cs#L120) (excerpt; see source for context).

{% code title="TypeAliasTest.cs" %}
```csharp
public void TestGetAlias()
{
	Assert.Equal("object", TypeAlias.GetAlias(typeof(object)), true);
	Assert.Equal("object[]", TypeAlias.GetAlias(typeof(object[])), true);
	Assert.Equal("DBNull", TypeAlias.GetAlias(typeof(DBNull)), true);
	Assert.Equal("DBNull[]", TypeAlias.GetAlias(typeof(DBNull[])), true);

	Assert.Equal("void", TypeAlias.GetAlias(typeof(void)), true);
	Assert.Equal("string", TypeAlias.GetAlias(typeof(string)), true);
	Assert.Equal("string[]", TypeAlias.GetAlias(typeof(string[])), true);

	Assert.Equal("int32", TypeAlias.GetAlias(typeof(int)), true);
	Assert.Equal("int32?", TypeAlias.GetAlias(typeof(int?)), true);
	Assert.Equal("int32[]", TypeAlias.GetAlias(typeof(int[])), true);
	Assert.Equal("int32?[]", TypeAlias.GetAlias(typeof(int?[])), true);

	Assert.Equal("Single", TypeAlias.GetAlias(typeof(float)), true);
	Assert.Equal("Single?", TypeAlias.GetAlias(typeof(float?)), true);
	Assert.Equal("Single[]", TypeAlias.GetAlias(typeof(float[])), true);
	Assert.Equal("Single?[]", TypeAlias.GetAlias(typeof(float?[])), true);

	Assert.Equal("Date", TypeAlias.GetAlias(typeof(DateOnly)), true);
	Assert.Equal("Date?", TypeAlias.GetAlias(typeof(DateOnly?)), true);
	Assert.Equal("date[]", TypeAlias.GetAlias(typeof(DateOnly[])), true);
	Assert.Equal("date?[]", TypeAlias.GetAlias(typeof(DateOnly?[])), true);

	Assert.Equal("Range<Timestamp>", TypeAlias.GetAlias(typeof(Zongsoft.Data.Range<DateTimeOffset>)), true);
	Assert.Equal("Range<Timestamp>?", TypeAlias.GetAlias(typeof(Zongsoft.Data.Range<DateTimeOffset>?)), true);
	Assert.Equal("Range<Timestamp>[]", TypeAlias.GetAlias(typeof(Zongsoft.Data.Range<DateTimeOffset>[])), true);
	Assert.Equal("Range<Timestamp>?[]", TypeAlias.GetAlias(typeof(Zongsoft.Data.Range<DateTimeOffset>?[])), true);

	Assert.Equal("Zongsoft.Tests.Gender@Zongsoft.Core.Tests", TypeAlias.GetAlias(typeof(Gender)), true);
	Assert.Equal("Zongsoft.Tests.Gender?@Zongsoft.Core.Tests", TypeAlias.GetAlias(typeof(Gender?)), true);
	Assert.Equal("Zongsoft.Tests.Gender[]@Zongsoft.Core.Tests", TypeAlias.GetAlias(typeof(Gender[])), true);
	Assert.Equal("Zongsoft.Tests.Gender?[]@Zongsoft.Core.Tests", TypeAlias.GetAlias(typeof(Gender?[])), true);

	Assert.Equal("IEnumerable<Zongsoft.Tests.Gender@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(IEnumerable<Gender>)), true);
	Assert.Equal("IEnumerable<Zongsoft.Tests.Gender?@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(IEnumerable<Gender?>)), true);
	Assert.Equal("IEnumerable<Zongsoft.Tests.Gender[]@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(IEnumerable<Gender[]>)), true);
	Assert.Equal("IEnumerable<Zongsoft.Tests.Gender?[]@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(IEnumerable<Gender?[]>)), true);

	Assert.Equal("List<Zongsoft.Tests.Gender@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(List<Gender>)), true);
	Assert.Equal("List<Zongsoft.Tests.Gender?@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(List<Gender?>)), true);
	Assert.Equal("List<Zongsoft.Tests.Gender[]@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(List<Gender[]>)), true);
	Assert.Equal("List<Zongsoft.Tests.Gender?[]@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(List<Gender?[]>)), true);

	Assert.Equal("IDictionary<String, Zongsoft.Tests.Gender@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(IDictionary<string, Gender>)), true);
	Assert.Equal("IDictionary<String, Zongsoft.Tests.Gender?@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(IDictionary<string, Gender?>)), true);
	Assert.Equal("IDictionary<String, Zongsoft.Tests.Gender[]@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(IDictionary<string, Gender[]>)), true);
	Assert.Equal("IDictionary<String, Zongsoft.Tests.Gender?[]@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(IDictionary<string, Gender?[]>)), true);

	Assert.Equal("Dictionary<Range<DateTime>, Zongsoft.Tests.Gender@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(Dictionary<Zongsoft.Data.Range<DateTime>, Gender>)), true);
	Assert.Equal("Dictionary<Range<DateTime>?, Zongsoft.Tests.Gender?@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(Dictionary<Zongsoft.Data.Range<DateTime>?, Gender?>)), true);
	Assert.Equal("Dictionary<Range<DateTime>[], Zongsoft.Tests.Gender[]@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(Dictionary<Zongsoft.Data.Range<DateTime>[], Gender[]>)), true);
	Assert.Equal("Dictionary<Range<DateTime>?[], Zongsoft.Tests.Gender?[]@Zongsoft.Core.Tests>", TypeAlias.GetAlias(typeof(Dictionary<Zongsoft.Data.Range<DateTime>?[], Gender?[]>)), true);

	var tupleType = typeof(Tuple<ValueTuple<Zongsoft.Data.Range<int>>, ValueTuple<Zongsoft.Data.Range<DateTime>?>, ValueTuple<Zongsoft.Data.Range<DateTime>[]>, ValueTuple<Zongsoft.Data.Range<DateTime>?[]>>);
	var tupleAlias = "Tuple<ValueTuple<Range<Int32>>, ValueTuple<Range<DateTime>?>, ValueTuple<Range<DateTime>[]>, ValueTuple<Range<DateTime>?[]>>";
	Assert.Equal(tupleAlias, tupleType.GetAlias());

	tupleType = typeof(ValueTuple<string, DateOnly?, byte[], Guid?[], Zongsoft.Data.Range<DateTime>?[], Zongsoft.Data.ConditionOperator?[]>);
	tupleAlias = "ValueTuple<String, Date?, Byte[], Guid?[], Range<DateTime>?[], Zongsoft.Data.ConditionOperator?[]@Zongsoft.Core>";
	Assert.Equal(tupleAlias, tupleType.GetAlias());

	tupleType = typeof(Nullable<>).MakeGenericType(tupleType);
	tupleAlias += '?';
	Assert.Equal(tupleAlias, tupleType.GetAlias());

	tupleType = tupleType.MakeArrayType();
	tupleAlias += "[]";
	Assert.Equal(tupleAlias, tupleType.GetAlias());
}
```
{% endcode %}

`GetAlias(assemblyless: true)` can omit assembly information and is suitable for displaying only the type name in the current context.

## Custom Alias

Custom aliases can be registered via `TypeAlias.Aliases.Map`.

Custom mappings should be maintained through the registration contract of TypeAlias.Aliases. The current business case does not register a custom type alias; please check the mapping name and type resolution range with [TypeAlias source code](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/TypeAlias.cs).

Custom aliases are suitable for short type names in configuration files, scripts, parameter packages, and data model mappings.

This uses the actual assertions from the framework TypeAliasTest, the Zongsoft.Tests type from the test project. Do not copy the test assembly name into the forum's business type configuration.

## Related Resources

* [TypeAlias.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/TypeAlias.cs)
* [TypeAlias.Parser.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/TypeAlias.Parser.cs)
* [TypeAliasTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TypeAliasTest.cs)
