---
description: "Zongsoft.Components AliasAttribute alias declaration and reading."
icon: tag
---

# AliasAttribute

`AliasAttribute` is used to declare one or more aliases for an assembly, module, type, member, or parameter. It is commonly used to map external inputs, configuration names, command names, or plugin names to internal members.

The design focus of aliases is "stable matching", not display text. A component can have multiple historical names, abbreviations, or external configuration names and still be processed through the same type or member at runtime.

## Key Capabilities

| Ability | Description |
| --- | --- |
| Multiple declarations | `AllowMultiple = true`, the same target can have multiple aliases. |
| Inherited reading | Inherited reading is supported by default. |
| static reading method | `GetAliases(...)` can be obtained from [`MemberInfo`](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.memberinfo) _ [Source](https://source.dot.net/#System.Private.CoreLib/MemberInfo.cs) _, [`Assembly`](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.assembly) _ [Source](https://source.dot.net/#System.Private.CoreLib/Assembly.cs) _, [`Module`](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.module) _ [Source](https://source.dot.net/#System.Private.CoreLib/Module.cs) _, [`ParameterInfo`](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.parameterinfo) _ [Source](https://source.dot.net/#System.Private.CoreLib/ParameterInfo.cs) _ or object read alias. |

Source: [framework/Zongsoft.Core/test/Models.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Models.cs#L247) (excerpt; see source for context).

{% code title="Models.cs" %}
```csharp
[DefaultValue(Female)]
public enum Gender : byte
{
	[Zongsoft.Components.Alias("F")]
	Female,

	[Zongsoft.Components.Alias("M")]
	[Description("Gender.Male")]
	Male,
}
```
{% endcode %}

Discussions does not declare AliasAttribute directly; this refers to the Gender enumeration in the Core test model. F and M are the aliases of the enumeration members. When reading, GetAliases should be called on the corresponding members, instead of reading the enumeration type and assuming that the aliases of all members can be obtained. This test enum is of a different type than Discussions.Models.Gender.

## Usage Suggestions

* Aliases are suitable for compatibility with old names, abbreviated names, or external configuration names.
* Don't think of an alias as a localized title; it should be a stable, matchable identifier.
* Aliases can reduce hard-coded name differences when a component needs to be referenced simultaneously by commands, configurations, plugins, and UI.
* If the alias will participate in user input parsing, it is recommended to unify the capitalization and separator conventions to avoid multiple ways of writing the same concept.

{% hint style="info" %}
`AliasAttribute.GetAliases(object)` will be dispatched to assembly, module, member, parameter or target type read according to the target object type; when a normal object is passed in, the alias on its runtime type is read.
{% endhint %}

## Reference Implementation

* [AliasAttribute.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/AliasAttribute.cs)
