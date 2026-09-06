---
description: "Zongsoft.Components.Converters Common type converters."
icon: repeat
---

# Converters

`Zongsoft.Components.Converters` provides common type converters at the component layer, mainly serving configuration binding, plugin builtin attribute parsing, command option binding and serialization collaboration.

The goal of these converters is not to replace business parsing logic, but to make external text configuration more similar to human writing habits, while keeping runtime objects still unambiguously typed.

## Converter List

| converter | Description |
| --- | --- |
| `ArchitectureConverter` | Convert between text and handler schema enumerations. |
| `BooleanConverter` | Boolean value conversion, suitable for compatibility with multiple text representations. |
| `CollectionConverter` | Collection converter, used to convert text or objects into target collections. |
| `CollectionConverter<TElementConverter>` | A collection converter that specifies a converter for collection elements. |
| `EncodingConverter` | Character encoding converter, for example from `utf-8` to `Encoding.UTF8`. |
| `EndpointConverter` | Network endpoint converter. |
| `EnumConverter` | Enumeration converter. |
| `GuidConverter` | GUID converter. |
| `TimeSpanConverter` | Time interval converter that supports time expressions in command options and configuration. |

## Use Location

* builtin attribute in plugin manifest.
* Complex values in options configuration files.
* Command line option bindings, such as `--timeout:5s`.
* Where JSON, type descriptors or configuration bindings require text to object conversion.

Source: [framework/Zongsoft.Core/src/Terminals/Commands/ShellCommand.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Terminals/Commands/ShellCommand.cs#L42) (excerpt; see source for context).

{% code title="ShellCommand.cs" %}
```csharp
[CommandOption(TIMEOUT_OPTION, 't', typeof(TimeSpan), "1s")]
public class ShellCommand : CommandBase<CommandContext>
{
	#region 常量定义
	private const string TIMEOUT_OPTION = "timeout";
```
{% endcode %}

`BooleanConverter` will be compatible with common writing methods such as `1`, `0`, `on`, `off`, `yes`, `no`, `enable`, `disable`; `TimeSpanConverter` delegates the universal time interval parsing tool to process text. The specific format should be subject to the corresponding converter source code and caller configuration convention.

## Collection Conversion

`CollectionConverter` is suitable for parsing a single text value in a connection string, plugin property, or command option into an array or generic collection. Default delimiters include `|`, `;`, `,`, and newline; parsing removes whitespace items and trims whitespace at both ends of each element. The elements are concatenated using the first delimiter when writing back the string.

If the target type is an array, the converter creates an array of the corresponding element type; if the target type is an abstract collection type, it creates `System.Collections.Generic.List<T>`; if the target type is a concrete collection type, it creates the collection and adds elements one by one.

Source: [framework/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs#L462) (excerpt; see source for context).

{% code title="ConnectionSettingsTest.cs" %}
```csharp
[TypeConverter(typeof(Components.Converters.CollectionConverter<MappingEntryConverter>))]
public MappingEntry[] Mapping
{
	get => this.GetValue<MappingEntry[]>();
	set => this.SetValue(value);
}
```
{% endcode %}

Source: [framework/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs#L485) (excerpt; see source for context).

{% code title="ConnectionSettingsTest.cs" %}
```csharp
public override object ConvertFrom(ITypeDescriptorContext context, CultureInfo culture, object value)
{
	if(value is string text)
	{
		if(string.IsNullOrEmpty(text))
			return default(MappingEntry);

		var index = text.IndexOfAny([':', '=']);

		return index < 0 ?
			new MappingEntry(text.Trim()) :
			new MappingEntry(text[..index].Trim(), text[(index + 1)..].Trim());
	}

	return base.ConvertFrom(context, culture, value);
}
```
{% endcode %}

Source: [framework/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/ConnectionSettingsTest.cs#L130) (excerpt; see source for context).

{% code title="ConnectionSettingsTest.cs" %}
```csharp
Assert.NotNull(settings.Mapping);
Assert.NotEmpty(settings.Mapping);
Assert.Equal(3, settings.Mapping.Length);
Assert.Equal("s1", settings.Mapping[0].Source, true);
Assert.Equal("t1", settings.Mapping[0].Target, true);
Assert.Equal("s2", settings.Mapping[1].Source, true);
Assert.Equal("t2", settings.Mapping[1].Target, true);
Assert.Equal("same", settings.Mapping[2].Source, true);
Assert.Equal("same", settings.Mapping[2].Target, true);
```
{% endcode %}

These snippets come from ConnectionSettingsTest: the Mapping property belongs to MyConnectionSettings, MappingEntry and MappingEntryConverter are in the same file, and settings come from MyDriver's parsing of the test connection string. The above declaration allows `mapping=s1:t1,s2=t2,same` to be parsed into three `MappingEntry` elements: `s1 -> t1`, `s2 -> t2`, and `same -> same`. Ordinary `CollectionConverter` will use the conversion rules of the element type itself; `CollectionConverter<TElementConverter>` explicitly creates `TElementConverter` to parse each element, which is suitable for scenarios where the element type itself does not register a global converter, or a certain attribute requires a dedicated parsing format.

{% hint style="warning" %}
The collection converter splits text by simple delimiters and does not handle quote escapes or nested structures. When the element value itself may contain `|`, `;`, `,`, or wrap, a more explicit configuration structure should be used instead, or a dedicated converter should be provided for the outer type.
{% endhint %}

Converters are suitable for handling general-purpose, reusable text-to-object conversions. When it comes to business semantics, permissions, external resource search or complex verification, it is recommended to complete it in the command, handler or configuration loading process. Before adding a new format to a command option or plugin attribute, you can also confirm whether the existing converter has been overwritten to avoid multiple sets of parsing rules for the same type.

## Reference Implementation

* [Converters source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Components/Converters)
* [CollectionConverter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/Converters/CollectionConverter.cs)
