---
description: "AnnotationUtility member annotation reading tool."
icon: book
---

# AnnotationUtility

`AnnotationUtility` is used to read display-related comments from type members, including classification, display name, and description.

| method | Description |
| --- | --- |
| `GetCategory` | Read [`CategoryAttribute`](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.categoryattribute) _[Source](https://source.dot.net/#System.ComponentModel.Primitives/CategoryAttribute.cs)_. |
| `GetDisplayName` | Read [`DisplayNameAttribute`](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.displaynameattribute) _[Source](https://source.dot.net/#System.ComponentModel.Primitives/DisplayNameAttribute.cs)_ or `DisplayAttribute.Name`. |
| `GetDescription` | Read [`DescriptionAttribute`](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.descriptionattribute) _[Source](https://source.dot.net/#System.ComponentModel.Primitives/DescriptionAttribute.cs)_ or `DisplayAttribute.Description`. |

Source: [framework/externals/aliyun/src/Telecom/PhoneTransmitter.cs](https://github.com/Zongsoft/framework/blob/main/externals/aliyun/src/Telecom/PhoneTransmitter.cs#L71) (excerpt; see source for context).

{% code title="PhoneTransmitter.cs" %}
```csharp
_descriptor = new TransmitterDescriptor(this.Name, AnnotationUtility.GetDisplayName(this.GetType()), AnnotationUtility.GetDescription(this.GetType()));
```
{% endcode %}

## From Annotations to Display Metadata

The above is the actual code for the framework Aliyun PhoneTransmitter to create the transmitter descriptor. This type uses the DisplayName and Description attributes to declare the resource key. AnnotationUtility reads the annotations and cooperates with the resource mechanism to obtain the title and description. See [template sender](../communication/transmitter.md) for a complete example, and [Resource management](../resources.md) for resource search. Discussions' model fields and enumerations also rely on resource descriptions, but without this sender code.

## When to Use

When the description of a model, command, or configuration item is maintained uniformly by a type declaration, reading the annotation can avoid saving the name repeatedly in each interface. Annotations are responsible for displaying metadata, do not participate in database column name matching, and cannot replace parameter verification. Missing annotations or resources should be handled according to the caller's display convention, and the returned title should not be used as a permission key or persistent identifier.

## Related Resources

* [AnnotationUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/AnnotationUtility.cs)
