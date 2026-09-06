---
description: "Understand condition combinations and database-side expressions from moderator review and page views increment."
icon: book-open
---

# Conditions and Operands


Conditions determine which records are operable, and operand determines how fields are updated. Both should describe business constraints to avoid concurrent coverage caused by reading the value first and then calculating it in the application and then writing it back.

## Condition Combination: Only Moderators Can Review

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L70) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
public bool Approve(ulong threadId)
{
	var criteria = Condition.Equal(nameof(Models.Thread.ThreadId), threadId) &
	               Condition.Equal(nameof(Models.Thread.Approved), false) &
	               GetIsModeratorCriteria();

	return this.DataAccess.Update<Models.Thread>(new
	{
		Approved = true,
		ApprovedTime = DateTime.Now,
		Post = new
		{
			Approved = true,
		}
	}, criteria, "*,Post{Approved}") > 0;
}
```
{% endcode %}

The update runs only when all three conditions hold: the thread ID matches, the thread has not yet been approved, and the caller is a moderator. The data schema explicitly includes Post’s approval flag. Hiding a UI button is not sufficient; the service must enforce the action’s constraints.

## Existence Condition: Moderator Record in the Forum

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L239) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
private Zongsoft.Data.Condition GetIsModeratorCriteria()
{
	return Condition.Exists("Forum.Users",
	         Condition.Equal(nameof(Forum.ForumUser.UserId), this.Principal.Identity.GetIdentifier<uint>()) &
	         Condition.Equal(nameof(Forum.ForumUser.IsModerator), true));
}
```
{% endcode %}

Exists takes the Forum.Users relationship as the scope, and internally matches both the user number and the moderator logo. The navigation name comes from [mapping](mapping.md), not any SQL table name. The associated conditions together with the current SiteId form the complete business scope.

## Operand: the Number of Views Is Incremented on the Database Side

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L183) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
//递增当前主题的累计阅读量并更新最后查看时间
this.DataAccess.Update<Models.Thread>(new
{
	TotalViews = Operand.Field(nameof(Models.Thread.TotalViews)) + 1,
	ViewedTime = DateTime.Now,
}, Condition.Equal(nameof(Models.Thread.ThreadId), thread.ThreadId));
```
{% endcode %}

Operand.Field represents the current field value of the database. The addition operation is a write expression, which can reduce the contention window of reading first and then writing. Subsequent memory objects also increment the count to make this response consistent with the update that has already been performed; it is not a second database write.

## Tenant Conditions Cannot Be Replaced by the Caller

Source: [src/Data/DataValidator.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Data/DataValidator.cs#L81) (excerpt; see source for context).

{% code title="DataValidator.cs" %}
```csharp
public ICondition Validate(IDataAccessContextBase context, ICondition criteria)
{
	if(UserIdentity.Current == null)
		return criteria;

	//调用方提供的站点条件不能替代当前身份的站点约束。
	if(HasProperty(context, Fields.SiteId))
		criteria &= Condition.Equal(Fields.SiteId, UserIdentity.Current.SiteId);

	return criteria;
}
```
{% endcode %}

When a Discussions identity exists and the entity has a SiteId, the validator appends the current site condition. This constraint cannot be canceled even if the caller explicitly passes in other SiteIds. The authentication initialization path without the Discussions identity has different prerequisites, and it cannot be claimed that the authenticator alone covers all anonymous entrances.

Continue reading [Query](querying.md), [write](writing.md) and [Certification](../security/authentication.md).
