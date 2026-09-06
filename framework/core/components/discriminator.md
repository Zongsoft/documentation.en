---
description: "Zongsoft.Components Discriminator discriminator and plugin builtin assembly."
icon: tags
---

# Discriminator

`IDiscriminator` is a small but useful interface: it lets a container object identify a target type or a subset of targets based on input parameters. When plugin builtin is assembled, if the parent object implements the recognizer, the child builtin can be appended to the correct location instead of relying on fixed property names or collection types.

It is suitable for solving the assembly problem of "allowing multiple sub-objects under the same node". The recognizer leaves type determination to the domain container itself, rather than having the plugin builder guess each collection property.

## Key Types

| Type | Description |
| --- | --- |
| `IDiscriminator` | Define `Discriminate(object argument)` to return the recognition result according to the parameters. |

## Example of Permission Classification

`PrivilegeCategory` contains both subcategory collections and permission collections. After it implements `IDiscriminator`, it can return which collection it should be appended to based on the input content.

Source: [framework/Zongsoft.Core/src/Security/Privileges/PrivilegeCategory.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/PrivilegeCategory.cs#L128) (excerpt; see source for context).

{% code title="PrivilegeCategory.cs" %}
```csharp
object IDiscriminator.Discriminate(object argument)
{
	switch(argument)
	{
		case string type:
			if(string.IsNullOrEmpty(type) || string.Equals(type, nameof(Category), StringComparison.OrdinalIgnoreCase))
				return this.Categories;

			if(string.Equals(type, nameof(Privilege), StringComparison.OrdinalIgnoreCase))
				return this.Privileges;

			break;
		case Privilege:
			return this.Privileges;
		case PrivilegeCategory:
			return this.Categories;
	}

	return null;
}
```
{% endcode %}

This is the actual implementation of the core permission classification. The empty type name and Category point to the subcategory collection, and Privilege points to the permission collection. String comparison ignores case; if it cannot be recognized, it returns empty. Discussions' plugin mounting still follows the same builtin assembly rules, but does not define its own IDiscriminator.

## Use in Plugin Assembly

`BuiltinType` in `Zongsoft.Plugins` will first ask the owner or default member whether it implements `IDiscriminator`. If [`Type`](https://learn.microsoft.com/en-us/dotnet/api/system.type) _[Source](https://source.dot.net/#System.Private.CoreLib/Type.cs)_ or a collection is returned, it will be used to determine the builtin type. When `ObjectBuilder` appends a child object, it will also first call the container's identifier and hand the child object to the correct collection.

{% stepper %}
{% step %}
## Read Plugin Node

The plugin system reads a builtin node in preparation for constructing or appending objects.
{% endstep %}

{% step %}
## Query Recognizer

If the parent object implements `IDiscriminator`, the builder passes the type name or child object to `Discriminate(...)`.
{% endstep %}

{% step %}
## Select Target

The recognizer returns the target type, target collection, or target container, and the builder completes the type resolution or append operation.
{% endstep %}
{% endstepper %}

This design is particularly suitable for plugin structures where "one node may contain multiple sub-objects". For example, permission categories can contain both permission items and sub-categories.

{% hint style="info" %}
The identifier return value can be a target type, a target collection, or a target container, and the specific interpretation is determined by the caller. The behavior in plugin assembly is subject to the builtin build process of `Zongsoft.Plugins`.
{% endhint %}

`IDiscriminator` is suitable for scenarios where there are clear classification rules inside the container. If the child object has only a fixed collection property, it is often simpler to expose the collection or property directly. Identification rules should also be kept predictable and try not to rely on external state or frequently changing runtime conditions.

## Reference Implementation

* [IDiscriminator.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IDiscriminator.cs)
* [PrivilegeCategory.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/PrivilegeCategory.cs)
* [BuiltinType.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/BuiltinType.cs)
* [ObjectBuilder.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/Builders/ObjectBuilder.cs)
