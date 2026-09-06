---
description: "From Discussions User Identity understands the type and value that together form the identity."
icon: fingerprint
---

# Identifiers


Identifier puts the identification class together with the identification value, so that the general security component does not have to rely on a specific forum user class. Discussions' UserIdentity implements the user contract, converting a UserId into a universal identity.

Source: [src/Security/UserIdentity.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Security/UserIdentity.cs#L102) (excerpt; see source for context).

{% code title="UserIdentity.cs" %}
```csharp
Identifier IIdentifiable.Identifier
{
	get => new(typeof(IUser), this.UserId);
	set => this.UserId = value.Validate<uint>(out var id) ? id : this.UserId;
}
#endregion
```
{% endcode %}

The type here is IUser, not UserIdentity. When setting the identifier, first verify whether the value can be used as a uint, and then update the UserId; if it does not meet the requirements, keep the original number. Identification is used to answer who the subject is and does not automatically load user profiles or grant permissions.

## Related Contracts

| Type | responsibility |
| --- | --- |
| IIdentifiable | Provide unified identification attributes |
| Identifier | Save type, value and optional label, description |
| Identifier&lt;T&gt; | Identification of a determined value type |

UserIdentity also contains SiteId. The user number and site scope bear different responsibilities. After passing the Identifier across modules, business operations still need to restore or verify the current site and resource permissions. See [Authentication and Authorization](../../security/authentication.md).

Frame definition: [Identifier](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/Identifier.cs), [IIdentifiable](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IIdentifiable.cs).
