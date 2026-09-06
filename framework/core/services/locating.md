---
description: "Service resolution scope and lifecycle, illustrated by dependencies between Discussions services."
icon: magnifying-glass
---

# Service Resolution and Ownership


Before resolving a service, establish which container provides it, which contract identifies it, and who owns its disposal. Discussions services usually resolve dependencies from their own ServiceProvider or from a template provider’s Services.

## Resolve Related Business Services

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L54) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
public PostService Posting
{
	get
	{
		if(_posting == null)
			_posting = this.ServiceProvider.ResolveRequired<PostService>();

		return _posting;
	}
}
#endregion
```
{% endcode %}

ThreadService caches the PostService reference returned by the container. It neither creates a new service scope nor disposes the service after each business operation. A container alone cannot prevent state leaking between requests if a shared service stores mutable request state.

## Template Providers Use the Same Business Services

Source: [src/Features/Archiving/UserDataTemplateModelProvider.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Features/Archiving/UserDataTemplateModelProvider.cs#L61) (excerpt; see source for context).

{% code title="UserDataTemplateModelProvider.cs" %}
```csharp
var data = argument switch
{
	string key => this.Services.ResolveRequired<UserService>().Get(key, schema),
	IModel model => this.Services.ResolveRequired<UserService>().Select(Criteria.Transform(model), schema),
	_ => this.Services.ResolveRequired<UserService>().Select(null, schema),
};
```
{% endcode %}

Both key-based lookups and condition-based selections go through UserService. Template assembly reuses service-layer constraints instead of opening its own database connection. Callers must still check the user’s filtering scope and export permissions.

## Module Dependencies and Missing Services

MessageSendCommand’s ServiceDependency specifies Discussions as its Provider; the module obtains its accessor by name. Naming, attribute registration, assembly scanning, and plugin mounting are separate steps. Missing any one of them can cause service resolution to fail.

The framework also supports service resolution by name, matching parameters, and tags. Discussions does not demonstrate every form. See [Services](../services.md) for the complete reference; there is no need to invent additional forum services just to demonstrate every overload.
