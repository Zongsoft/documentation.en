---
description: "Understand filter timing, lifecycle and permissions from Discussions query post-text processing."
icon: filter
---

# Filters


filter adds cross-cutting logic before and after existing operations. Discussions' PostFilter is a data access filter that uses query context to process text; it is different from the general executor filter interface, but exhibits the same important timing and lifecycle constraints.

## Before and After Inquiry

Source: [src/Data/PostFilter.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Data/PostFilter.cs#L43) (excerpt; see source for context).

{% code title="PostFilter.cs" %}
```csharp
public void OnFiltering(DataSelectContextBase context) { }
public void OnFiltered(DataSelectContextBase context)
{
	if(context.Result == null)
		return;

	var identity = context.Principal?.Identity;
	context.Result = FilteredResult.Create(context, item => Filter(item, identity));
}
#endregion
```
{% endcode %}

The results are not wrapped before querying because the underlying operations have not yet produced the final collection. After the query is wrapped in [FilteredResult](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Data/FilteredResult.cs), processing still occurs during the enumeration phase. This thin adaptation layer satisfies both the non-generic collection requirements of the query context and the caller's generic asynchronous enumeration requirements, and forwards paging notifications; the actual enumeration, filtering, cancellation and release are handed over to Core's [Collection Extensions](../collections/extensions.md) and [Pageable](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Data/Pageable.cs).

{% hint style="warning" %}
🚨 This implementation of Discussions requires the paging filtering fix in Core 7.59.0. Before this version is released, you need to use the local framework reference according to [Prerequisites](../../../get-started/prerequisites.md); the paging filtering of Core 7.58.0 will pass in the wrong current element, and compatibility cannot be determined by compilation alone.
{% endhint %}

## Process the Body Without Changing the Number of Records

Source: [src/Data/PostFilter.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Data/PostFilter.cs#L55) (excerpt; see source for context).

{% code title="PostFilter.cs" %}
```csharp
private static bool Filter(object item, System.Security.Principal.IIdentity identity)
{
	var dictionary = DataDictionary.GetDictionary<Models.Post>(item);
	if(!dictionary.TryGetValue(p => p.Content, out var content))
		return true;

	if(!(dictionary.TryGetValue(p => p.Approved, out var approved) && approved) &&
	   !(identity?.IsAuthenticated == true && dictionary.TryGetValue(p => p.CreatorId, out var creatorId) && identity.GetIdentifier<uint>() == creatorId))
	{
		dictionary.TrySetValue(p => p.Content, string.Empty);
		if(dictionary.TryGetValue(p => p.ContentType, out var hiddenType))
			dictionary.TrySetValue(p => p.ContentType, Utility.GetContentType(hiddenType, true));
	}
	else if(dictionary.TryGetValue(p => p.ContentType, out var contentType) && !Utility.IsContentEmbedded(contentType))
	{
		dictionary.SetValue(p => p.Content, string.IsNullOrEmpty(content) ? string.Empty : Utility.ReadTextFile(content));
		dictionary.SetValue(p => p.ContentType, Utility.GetContentType(contentType, true));
	}

	return true;
}
```
{% endcode %}

Texts that have not been reviewed and are not by the author are left blank and the record is retained. If the judgment field is missing, it cannot be regarded as approved. After the external file is read, the content type is adjusted synchronously to prevent subsequent services from reading the text as a file path again.

There is also a reading order issue for topic details: ThreadFilter is desensitized first, and ThreadService then checks whether the current user is a moderator. Now filter associates the original text with the current model instance and saves it in the internal weak reference table; synchronous and asynchronous detail reading only restores the text after the moderator's authorization is successful. It does not modify Approved, nor does it allow normal list queries to bypass desensitization. The external text is also read after authorization is completed. For related implementation, see [ThreadFilter](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Data/ThreadFilter.cs) and [ThreadService](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs).

## Do Not Treat Filter as Full Authorization

It handles returned content and does not automatically authorize editing, moderation, deletion, or cross-site access. The SiteId is constrained by the validator, and the action permissions are controlled by the service and security mechanisms; these rules need to be checked together.

For the general filtering contract and executor implementation, refer to [component source code](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Components), and for the data access process, see [Data Services](../../data/services.md).
