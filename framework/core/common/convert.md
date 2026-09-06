---
description: "Convert type conversion and hexadecimal conversion tool."
icon: code
---

# Convert

`Zongsoft.Common.Convert` is an enhanced conversion tool within the framework covering regular type conversions, nullable types, enum aliases, [`TimeSpan`](https://learn.microsoft.com/en-us/dotnet/api/system.timespan) _[Source](https://source.dot.net/#System.Private.CoreLib/TimeSpan.cs)_ abbreviation format, hex literals and custom converters.

## Conversion Value

`ConvertValue` supports regular type conversion, enumeration alias conversion, [`TimeSpan`](https://learn.microsoft.com/en-us/dotnet/api/system.timespan) _[Source](https://source.dot.net/#System.Private.CoreLib/TimeSpan.cs)_ abbreviation format conversion, and custom [`TypeConverter`](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.typeconverter) _[Source](https://source.dot.net/#System.ComponentModel.TypeConverter/TypeConverter.cs)_.

Source: [framework/Zongsoft.Core/test/Common/ConvertTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/ConvertTest.cs#L15) (excerpt; see source for context).

{% code title="ConvertTest.cs" %}
```csharp
Assert.Null(Zongsoft.Common.Convert.ConvertValue<int?>("", default(int?)));
Assert.Null(Zongsoft.Common.Convert.ConvertValue<int?>("x", () => default(int?)));
Assert.NotNull(Zongsoft.Common.Convert.ConvertValue<int?>("123", () => default(int?)));
Assert.Equal(123, Zongsoft.Common.Convert.ConvertValue<int?>("123", () => default(int?)));

object value = 123;
Assert.Equal(123.0f, Zongsoft.Common.Convert.ConvertValue<float>(value));
Assert.Equal(123.0d, Zongsoft.Common.Convert.ConvertValue<double>(value));
Assert.Equal(123.0m, Zongsoft.Common.Convert.ConvertValue<decimal>(value));

value = "123";
Assert.Equal(123, Zongsoft.Common.Convert.ConvertValue<int>(value));
```
{% endcode %}

`TryConvertValue` is suitable for scenarios where you do not want an exception to be thrown if the conversion fails. The following framework test also demonstrates the cooperation of the time span abbreviation format and type conversion.

Source: [framework/Zongsoft.Core/test/Common/ConvertTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/ConvertTest.cs#L33) (excerpt; see source for context).

{% code title="ConvertTest.cs" %}
```csharp
var duration = TimeSpan.Parse("1:20:30");
Assert.Equal(duration, Zongsoft.Common.Convert.ConvertValue<TimeSpan>("1:20:30"));
Assert.True(TimeSpanUtility.TryParse("15S", out duration));
Assert.Equal(duration, Zongsoft.Common.Convert.ConvertValue<TimeSpan>("15s"));
Assert.True(TimeSpanUtility.TryParse("20m", out duration));
Assert.Equal(duration, Zongsoft.Common.Convert.ConvertValue<TimeSpan>("20M"));
Assert.True(TimeSpanUtility.TryParse("30h", out duration));
Assert.Equal(duration, Zongsoft.Common.Convert.ConvertValue<TimeSpan>("30H"));
Assert.True(TimeSpanUtility.TryParse("40D", out duration));
Assert.Equal(duration, Zongsoft.Common.Convert.ConvertValue<TimeSpan>("40d"));
```
{% endcode %}

Binary and hexadecimal text can be converted to and from each other.

Source: [framework/Zongsoft.Core/test/Common/ConvertTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/ConvertTest.cs#L70) (excerpt; see source for context).

{% code title="ConvertTest.cs" %}
```csharp
public void TestToHexString()
{
	var source = new byte[16];

	for(int i = 0; i < source.Length; i++)
		source[i] = (byte)i;

	var hexString1 = Zongsoft.Common.Convert.ToHexString(source);
	var hexString2 = Zongsoft.Common.Convert.ToHexString(source, '-');

	Assert.Equal("000102030405060708090A0B0C0D0E0F", hexString1);
	Assert.Equal("00-01-02-03-04-05-06-07-08-09-0A-0B-0C-0D-0E-0F", hexString2);

	var bytes1 = Zongsoft.Common.Convert.FromHexString(hexString1);
	var bytes2 = Zongsoft.Common.Convert.FromHexString(hexString2, '-');

	Assert.Equal(source.Length, bytes1.Length);
	Assert.Equal(source.Length, bytes2.Length);

	Assert.True(BinaryCompare(source, bytes1));
	Assert.True(BinaryCompare(source, bytes2));
}
```
{% endcode %}

{% content-ref url="enum-utility.md" %}
[enum-utility.md](enum-utility.md)
{% endcontent-ref %}

These snippets come from the framework ConvertTest. BinaryCompare in the hex test is a helper method of the test class; the first two snippets are excerpts from inside the method. Discussions The actual implementation of custom tag conversion can be found at [TagsConverter](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Models/TagsConverter.cs).

## Related Resources

* [Convert.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/Convert.cs)
* [ConvertTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/ConvertTest.cs)
