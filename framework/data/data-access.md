---
description: "Explain accessors, result lifecycle and business boundaries with Discussions module accessors and real service calls."
icon: book-open
---

# Data Access Interfaces


IDataAccess is a data engine public contract. Discussions Obtain named accessors in modules so that models, maps, validators and filters use consistent business module scope.

## Get Accessor from Module

Source: [src/Module.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Module.cs#L52) (excerpt; see source for context).

{% code title="Module.cs" %}
```csharp
public IDataAccess Accessor => _accessor ??= this.Services.ResolveRequired<IDataAccessProvider>().GetAccessor(this.Name);
#endregion
```
{% endcode %}

The name here is Discussions. The provider manages the accessor, and the business method should not release the shared accessor after each query. The assembly only references the Core contract; the data engine and driver are deployed by the host, see [First query](quickstart.md).

## Using Accessors Within Services

Source: [src/Services/UserService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/UserService.cs#L83) (excerpt; see source for context).

{% code title="UserService.cs" %}
```csharp
public int GetMessageUnreadCount(uint userId = 0)
{
	if(userId == 0)
		userId = this.Principal.Identity.GetIdentifier<uint>();

	return this.DataAccess.Count<UserMessage>(Condition.Equal(nameof(UserMessage.UserId), userId) & Condition.Equal(nameof(UserMessage.IsRead), false));
}
```
{% endcode %}

UserMessage saves recipients and read status. The current user number provides business meaning for the default parameters, and Count only returns the number of matching records. The public interface must also check whether the caller can access the user number passed in, the default value itself does not provide authorization.

## Lifecycle of Query Results

Select usually returns enumerable results, and the actual reading may occur during the enumeration phase. SelectAsync returns an asynchronous sequence, the caller should propagate cancellation tokens and consume them in sequence or release them early. Do not treat the query method return as if the database read has been completely completed.

Source: [src/Security/UserChallenger.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Security/UserChallenger.cs#L87) (excerpt; see source for context).

{% code title="UserChallenger.cs" %}
```csharp
protected virtual ValueTask<UserProfile> GetUserAsync(uint userId, CancellationToken cancellation) =>
	Module.Current.Accessor.SelectAsync<UserProfile>(
		Condition.Equal(nameof(UserProfile.UserId), userId),
		Paging.Limit(1),
		cancellation).FirstOrDefault(cancellation);
```
{% endcode %}

Post filtering and events may throw exceptions during asynchronous preparation or enumeration; exception handling needs to cover the consumption process. Paginated advice also relies on the result wrapper to remain IPageable.

## Which Layer Is Suitable for Direct Calling?

The authentication challenger needs to read user information before the Discussions identity has been established, so it uses Module.Current.Accessor directly; ordinary business requests call UserService or ThreadService first. The identity conditions of the two are different, and the authentication initialization code cannot be copied as the anonymous query interface.

Reference: [Query](querying.md), [write](writing.md), [Data Services](services.md).
