---
description: "Use Discussions' resource keys and framework existing test instructions to localize resources, position fallbacks, and component titles."
icon: book
---

# Zongsoft.Resources

Resources provides a unified access portal for assembly resources. A resource key can remain stable while the display text changes with cultural settings; the type and member position help the framework determine which resource set to look for. It is commonly used in component titles, enumeration descriptions, command help, and error prompts.

## Resource Organization in Discussions

[Properties/Resources.resx](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Properties/Resources.resx) from Discussions has saved the forum field text, eliminating the need to create a hypothetical UserCategory or security module resource set.

| real resource key | Chinese text | Purpose |
| --- | --- | --- |
| Accessibility | accessibility | The display name of the access scope |
| Accessibility.Internal | insider | enum value text |
| Accessibility.Moderator | Moderator | enum value text |
| Accessibility.Specified | Limited personnel | enum value text |
| Approved | Reviewed | Display name for review status |
| Forum | Forum | Domain object name |
| ForumId | Forum number | Field name |

Resources.Designer.cs is the corresponding build accessor. To adjust the text, you should edit resx, and then use the project's resource generation process to update the accessor; directly modifying the generated file cannot retain the changes stably. The presence of a resource key only indicates that the text is available for lookup, not that every frontend is using it.

## How the Main Objects Collaborate

| object | Responsibilities |
| --- | --- |
| IResource | Read strings, objects, and the results of attempts to read by key |
| Resource | Scan the resource set within the assembly and organize the reading |
| IResourceLocator | Convert call locations to resource set candidate names |
| ResourceLocator | Provides default hierarchical targeting rules |
| ResourceUtility and Resource extensions | Receive types, members and candidate keys to reduce repeated conversions by the caller |

Resource set location and resource key lookup are two steps: first find the resource set that may contain text, and then look for the key in it. Locations usually come from the type's namespace, and location strings cannot be treated as resource keys.

## Complete Reading Example: Enumeration Description Test

Discussions does not have a standalone resource reading test, so the framework ResourceTest is used. The Gender here comes from the test project, not Discussions.Models.Gender; both projects have their own resource files.

Source: [framework/Zongsoft.Core/test/Resources/ResourceTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Resources/ResourceTest.cs#L14) (excerpt; see source for context).

{% code title="ResourceTest.cs" %}
```csharp
public void Test()
{
	var resource = Resource.GetResource<Gender>();
	Assert.NotNull(resource);

	var text = resource.GetString("Gender.Male");
	Assert.NotNull(text);
	Assert.Equal("男士", text);

	text = resource.GetString("Gender.Female");
	Assert.NotNull(text);
	Assert.Equal("女士", text);

	text = EnumUtility.GetEnumDescription(Gender.Male);
	Assert.NotNull(text);
	Assert.Equal("男士", text);

	text = EnumUtility.GetEnumDescription(Gender.Female);
	Assert.NotNull(text);
	Assert.Equal("女士", text);
}
```
{% endcode %}

The test first directly reads Gender.Male / Gender.Female, and then obtains the enumeration description through EnumUtility. Both paths should get the same text. This shows that the interface can display resource text and the business still retains stable enumeration values. See [Enum Utilities](common/enum-utility.md) for further usage.

## True Fallback Rules for Locators

Source: [framework/Zongsoft.Core/src/Resources/ResourceLocator.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Resources/ResourceLocator.cs#L44) (excerpt; see source for context).

{% code title="ResourceLocator.cs" %}
```csharp
public IEnumerable<string> Locate(string origin)
{
	//如果资源管理器中没有资源集，则无需定位
	if(_resource.Count == 0)
		yield break;

	//如果资源管理器中只有一个资源集，则只能定位它
	if(_resource.Count == 1)
		yield return _resource.Resources.First().BaseName;

	if(!string.IsNullOrEmpty(origin))
	{
		foreach(var location in GetLocations(origin))
			yield return location;
	}

	foreach(var location in GetLocations($"{_resource.Assembly.GetName().Name}"))
		yield return location;
}
```
{% endcode %}

The default locator first handles the case where there is no resource set and only one resource set, then tries the source location and the assembly name. GetLocations will fallback from the full location, each level trying Properties.Resources, Resources, and then the location itself. The actual candidate name is determined by the assembly and call type, eliminating the need to hand-write another module's resource tree.

## How Does a Component Select a Candidate Key?

Source: [framework/Zongsoft.Core/src/Collections/CategoryBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/CategoryBase.cs#L130) (excerpt; see source for context).

{% code title="CategoryBase.cs" %}
```csharp
protected virtual string GetTitle() => Resources.ResourceUtility.GetString(_resource,
[
	$"{this.FullPath.Trim(PathSeparator).Replace(PathSeparator, '.')}.{nameof(Category)}.{nameof(this.Title)}",
	$"{this.FullPath.Trim(PathSeparator).Replace(PathSeparator, '.')}.{nameof(Category)}",
	$"{this.FullPath.Trim(PathSeparator).Replace(PathSeparator, '.')}.{nameof(this.Title)}",
	this.FullPath.Trim(PathSeparator).Replace(PathSeparator, '.'),
	$"{this.Name}.{nameof(Category)}.{nameof(this.Title)}",
	$"{this.Name}.{nameof(Category)}",
	$"{this.Name}.{nameof(this.Title)}",
	this.Name,
]);
```
{% endcode %}

Category first searches the title according to the complete category path, and then returns the node name. This rule allows nodes with the same name to display different text in different locations; it also allows small classification trees to provide only common names. See [Category](collections/category.md) for a complete classification tree example.

## Object Resources and Custom Positioning

GetObject is used to read non-string resources, the actual type is determined by the resource file. Discussions currently do not have a complete use case for reading icons and rendering through this interface, so code that relies on RenderIconAsync not being implemented is not provided. When a resource object needs to be displayed, the caller should first check the type and then hand it to the corresponding rendering component.

If the project uses a different resource set naming method, it can implement IResourceLocator and pass the instance to the Resource constructor. The default implementation already works for the Properties.Resources organization of Discussions, so there is no need to add a new locator for this case.

{% hint style="info" %}
💡 Resource.GetResource caches instances by assembly. When a dedicated locator is needed, explicitly create a Resource with the locator, or ensure that the correct configuration is used when caching for the first time; subsequent cached objects will not automatically replace the locating rules.
{% endhint %}

## How to Troubleshoot When Text Is Missing

First check whether resx contains the target key, then check whether the compiled product has the corresponding resource set, and finally check the current culture and source location. Distinguish between "key miss" and "hit text is empty"; the main process can accept the fallback text when missing, and configuration errors should be reported clearly. See [Resource.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Resources/Resource.cs) for framework implementation.
