---
description: "ArrayExtension, StringExtension, DateTimeExtension, TimeSpanUtility, TypeExtension, UriExtension common extensions."
icon: wrench
---

# Common Extensions

`Zongsoft.Common` provides a common set of extension methods for day-to-day processing of arrays, strings, times, types, and URIs.

## Type Relationship

| Type | Description |
| --- | --- |
| `ArrayExtension` | Array null value detection and creation of empty arrays by runtime type. |
| `StringExtension` | Character removal, string pruning, number detection, desensitization, fragmentation analysis. |
| `DateTimeExtension` | Time reset, time-consuming calculation. |
| `TimeSpanUtility` | Time intervals are parsed and limited to the specified range. |
| `TypeExtension` | Judgment of type compatibility, generic definition, collection type, scalar type, default value, etc. |
| `UriExtension` | Reads the specified key from a URL query string. |

## String Expansion

Source: [framework/Zongsoft.Core/test/Common/StringExtensionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/StringExtensionTest.cs#L44) (excerpt; see source for context).

{% code title="StringExtensionTest.cs" %}
```csharp
public void TestSlice()
{
	var parts = StringExtension.Slice("a - b --  c  ", '-').ToArray();

	Assert.NotEmpty(parts);
	Assert.Equal(3, parts.Length);
	Assert.Equal("a", parts[0]);
	Assert.Equal("b", parts[1]);
	Assert.Equal("c", parts[2]);

	var hasColon = false;
	parts = StringExtension.Slice("issue-100-park:10001-1", chr => hasColon ? false : !(hasColon = chr == ':') && chr == '-').ToArray();

	Assert.NotEmpty(parts);
	Assert.Equal(3, parts.Length);
	Assert.Equal("issue", parts[0]);
	Assert.Equal("100", parts[1]);
	Assert.Equal("park:10001-1", parts[2]);
}
```
{% endcode %}

The above directly quotes Core's sharding test, covering whitespace, continuous separators, and stateful separation judgments. The actual input and assertions of RemoveAny, Trim, and IsDigits are also in the same file. `Slice` ignores whitespace fragments and is suitable for parsing simple delimited text.

## Time Expansion

`TimeSpanUtility` supports `ms`, `s`, `m`, `h`, `d` and other abbreviated formats, and also supports standard time formats.

Source: [framework/Zongsoft.Core/test/Common/TimeSpanTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TimeSpanTest.cs#L10) (excerpt; see source for context).

{% code title="TimeSpanTest.cs" %}
```csharp
public void Clamp()
{
	var minimum = TimeSpan.FromHours(1);
	var maximum = TimeSpan.FromHours(12);

	var duration = TimeSpanUtility.Clamp(new TimeSpan(0, 2, 30, 59), minimum, maximum);
	Assert.Equal(new TimeSpan(0, 2, 30, 59), duration);

	duration = TimeSpanUtility.Clamp(TimeSpan.Zero, minimum, maximum);
	Assert.Equal(minimum, duration);
	duration = TimeSpanUtility.Clamp(TimeSpan.MinValue, minimum, maximum);
	Assert.Equal(minimum, duration);
	duration = TimeSpanUtility.Clamp(new TimeSpan(0, 0, 30, 40), minimum, maximum);
	Assert.Equal(minimum, duration);

	duration = TimeSpanUtility.Clamp(TimeSpan.MaxValue, minimum, maximum);
	Assert.Equal(maximum, duration);
	duration = TimeSpanUtility.Clamp(new TimeSpan(1, 0, 0, 1), minimum, maximum);
	Assert.Equal(maximum, duration);
}
```
{% endcode %}

`DateTimeExtension.GetElapsed` can calculate the elapsed time from a certain point in time to the present. Discussions' file upload controller generates a file name based on the number of days since the millennium era plus a random string; the following is a snippet of the UploadAsync callback parameter, and the complete method can be found in the source code link. Use a wall clock for this date difference, and a specialized timing tool should be used to measure code execution time.

Source: [src/api/Controllers/FileController.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/api/Controllers/FileController.cs#L81) (excerpt; see source for context).

{% code title="FileController.cs" %}
```csharp
  args => args.FileName = $"{Timestamp.Millennium.Epoch.GetElapsed().Days}-{Randomizer.GenerateString()}", cancellation);
```
{% endcode %}

## Type Extension

`TypeExtension` provides more flexible generic judgment capabilities than `Type.IsAssignableFrom`.

Source: [framework/Zongsoft.Core/test/Common/TypeExtensionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TypeExtensionTest.cs#L14) (excerpt; see source for context).

{% code title="TypeExtensionTest.cs" %}
```csharp
public void TestIsAssignableFrom()
{
	var baseType = typeof(ICollection<Person>);
	var instanceType = typeof(Collection<Person>);

	Assert.True(TypeExtension.IsAssignableFrom(baseType, instanceType));
	Assert.True(baseType.IsAssignableFrom(instanceType));

	baseType = typeof(ICollection<>);

	Assert.True(TypeExtension.IsAssignableFrom(baseType, instanceType));
	Assert.False(baseType.IsAssignableFrom(instanceType));

	Assert.True(TypeExtension.IsAssignableFrom(typeof(IService<>), typeof(EmployeeService)));
	Assert.True(TypeExtension.IsAssignableFrom(typeof(PersonServiceBase<>), typeof(EmployeeService)));

	Assert.True(TypeExtension.IsAssignableFrom(typeof(IService<>), typeof(EmployeeService), out var genericTypes));
	Assert.NotEmpty(genericTypes);
	Assert.Single(genericTypes);
	Assert.Same(genericTypes[0], typeof(IService<Employee>));

	Assert.True(TypeExtension.IsAssignableFrom(typeof(PersonServiceBase<>), typeof(EmployeeService), out genericTypes));
	Assert.NotEmpty(genericTypes);
	Assert.Single(genericTypes);
	Assert.Same(genericTypes[0], typeof(PersonServiceBase<Employee>));
}
```
{% endcode %}

The above is from TypeExtensionTest, and all collection types used are test inputs; the full test can be reused to check generic matching behavior. It can also determine sets, lists, dictionaries, hash sets, numeric types, nullable types, and scalar types.

## URI Extension

No call case for this extension has been found in Discussions and the framework project. [Implementation of UriExtension](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/UriExtension.cs) splits Query by & and =, and ignores the case of key names; it does not have a complete URI decoding process, nor does it actively remove the question mark from the Query header. Web requests should preferentially use the framework controller's request query set and avoid treating this tool as a full URL parameter resolver.

## Related Resources

* [ArrayExtension.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/ArrayExtension.cs)
* [StringExtension.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/StringExtension.cs)
* [DateTimeExtension.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/DateTimeExtension.cs)
* [TimeSpanUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/TimeSpanUtility.cs)
* [TypeExtension.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/TypeExtension.cs)
* [UriExtension.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/UriExtension.cs)
* [StringExtensionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/StringExtensionTest.cs)
* [TimeSpanTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TimeSpanTest.cs)
* [TypeExtensionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/TypeExtensionTest.cs)
