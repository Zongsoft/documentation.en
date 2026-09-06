---
description: "Understanding Authentication Boundaries from Discussions' Identity Challenges, Claim Transformations, and SiteId Validation."
icon: shield-halved
---

# Authentication and Authorization


Discussions adds a forum identity after basic authentication. The user ID establishes who the user is, SiteId defines the forum’s business scope, and moderator permissions and visibility rules determine which actions are allowed. Handle these concerns in separate layers.

## Mount Identity Extensions Through Plugins

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L41) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<extension path="/Workbench/Security/Authentication/Challengers">
	<object value="{static:Zongsoft.Discussions.Security.UserChallenger.Instance, Zongsoft.Discussions}" />
</extension>
```
{% endcode %}
Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L46) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<extension path="/Workbench/Security/Authentication/Transformers">
	<object value="{static:Zongsoft.Discussions.Security.UserIdentity+Transformer.Instance, Zongsoft.Discussions}" />
</extension>
```
{% endcode %}

The challenger reads or creates the forum user profile and adds an identity for the Discussions scheme to the principal. The transformer reconstructs UserIdentity from the claims. They serve different stages of the process rather than authenticating the user twice.

## Turn a User Profile into Claims

Source: [src/Security/UserChallenger.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Security/UserChallenger.cs#L126) (excerpt; see source for context).

{% code title="UserChallenger.cs" %}
```csharp
private ClaimsIdentity Identity(UserProfile user)
{
	var identity = user.Identity(UserIdentity.Scheme, "Zongsoft");

	identity.SetClaim(nameof(UserProfile.SiteId), user.SiteId);
	identity.SetClaim(nameof(UserProfile.Gender), user.Gender);
	identity.SetClaim(nameof(UserProfile.Avatar), user.Avatar);
	identity.SetClaim(nameof(UserProfile.Grade), user.Grade);
	identity.SetClaim(nameof(UserProfile.TotalPosts), user.TotalPosts);
	identity.SetClaim(nameof(UserProfile.TotalThreads), user.TotalThreads);

	//进行其他声明定义
	this.OnClaims(identity, user);

	//返回新构建的身份
	return identity;
}
```
{% endcode %}

SiteId, Gender, Avatar, Grade and statistics are all derived from the user profile. Identity snapshots should not be regarded as permanent real-time business data; when to refresh the credentials after field changes is still determined by the validity period and update policy of the security host.

## Current Forum Identity

Source: [src/Security/UserIdentity.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Security/UserIdentity.cs#L110) (excerpt; see source for context).

{% code title="UserIdentity.cs" %}
```csharp
public static UserIdentity Current => ClaimsIdentityModeling.GetModel<UserIdentity>(Scheme);
```
{% endcode %}

The scheme is [Zongsoft.Discussions](https://github.com/Zongsoft/discussions/tree/main/src). The current model is available only when an identity for that scheme has been created and its transformer registered. An ordinary ClaimsPrincipal or anonymous request does not automatically include this identity.

## Site Scope and Resource Permissions

DataValidator constrains the current site for queries and writes containing SiteId, and also fills in the site, creator and time when creating. Even if the request specifies another SiteId, it does not override the current identity scope. There is no forum identity yet during authentication initialization, so related queries must be called by a controlled authentication process.

Site isolation does not equal moderator authority, nor does it equal moderation visibility. The business actions of ThreadService, the visibility conditions of ForumService, and the query result filter jointly participate in the judgment, see [Data Services](../data/services.md).

{% hint style="warning" %}
🚨 The validator’s ability to fill in SiteId does not make every anonymous query, related entity, or custom API automatically secure. At each entry point, verify when the identity is established, how entities are mapped, and which resource permissions apply.
{% endhint %}
