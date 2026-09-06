---
description: "Use forum top topics, browsing records and voting statistics to illustrate conditions, modes, paging and aggregation."
icon: book-open
---

# Queries and Navigation


A Discussions query expresses four things at once: which model to read, which records to filter, which fields or relationships to return, and how to sort and paginate. You can start with the real path of reading the pinned topic from ForumService.

## Query Visible Pinned Topics

Source: [src/Services/ForumService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ForumService.cs#L83) (excerpt; see source for context).

{% code title="ForumService.cs" %}
```csharp
public IEnumerable<Models.Thread> GetPinnedThreads(ushort forumId, string schema, Paging paging = null)
{
	return this.DataAccess.Select<Models.Thread>(
		Condition.Equal(nameof(Models.Thread.ForumId), forumId) &
		Condition.Equal(nameof(Models.Thread.IsPinned), true) &
		Condition.Equal(nameof(Models.Thread.Visible), true),
		schema, paging, Sorting.Descending(nameof(Models.Thread.ThreadId)));
}
```
{% endcode %}

ForumId, IsPinned and Visible are combined using AND; the schema is provided by the caller, paging controls the root result set, and ThreadId determines the return order in reverse order. The visible mark does not mean review and approval, and the review and processing of the text will also go through [Results filter](services.md). Site conditions are governed by the current identity and DataValidator, and cross-site data should not be queried based on just a forum number.

## Global and Top Themes on the First Screen

Source: [src/Services/ForumService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ForumService.cs#L92) (excerpt; see source for context).

{% code title="ForumService.cs" %}
```csharp
public Models.Thread[] GetTopmosts(ushort forumId, string schema, int count = 10)
{
	count = Math.Max(5, Math.Min(50, count));

	var globals = this.GetGlobalThreads(0, schema, Paging.Page(1, count));
	var pinneds = this.GetPinnedThreads(forumId, schema, Paging.Page(1, count));

	return globals.Union(pinneds).OrderByDescending(t => t.ThreadId).Take(count).ToArray();
}
```
{% endcode %}

The maximum limit here is 50, read the global topic and the top topic of the current forum, and then combine and sort them. This is the top rule of the business layer, not the default paging behavior automatically executed by the data engine. Regular topic queries also exclude records that already appear at the top of the first screen to avoid repeated display.

## Expand Navigation with Patterns

Source: [src/Services/UserService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/UserService.cs#L67) (excerpt; see source for context).

{% code title="UserService.cs" %}
```csharp
public IEnumerable<History> GetHistories(uint userId, Paging paging = null)
{
	if(userId == 0)
		userId = this.Principal.Identity.GetIdentifier<uint>();

	return this.DataAccess.Select<History>(Condition.Equal(nameof(History.UserId), userId), $"*, {nameof(History.Thread)}" + "{*}", paging);
}
```
{% endcode %}

History's Thread navigation comes from mapping; the asterisk in the pattern reads simple fields, and the curly braces after Thread require expansion of the associated topic. Navigation increases read costs and should not unconditionally expand the entire object graph. The fields involved in review and authority judgment must be retained. For specific syntax, see [Data Schemas](schema.md).

## Get Only One Record Required for Identity Initialization

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

This is the query during the authentication challenge phase, where the cancellation token is passed along the call chain. Paging.Limit(1) limits the amount of reading, and FirstOrDefault consumes an asynchronous sequence. [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core)'s [Collection Extensions](../core/collections/extensions.md) is used here. After introducing `Zongsoft.Collections`, `.FirstOrDefault(cancellation)` can be called; it handles the empty sequence and releases the enumerator, without writing another first element auxiliary method in the business project. Do not regard this identity initialization path as an open user search interface.

## Restore Voting Statistics with Counts

Source: [src/Services/PostService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/PostService.cs#L229) (excerpt; see source for context).

{% code title="PostService.cs" %}
```csharp
private bool SetPostVotes(ulong postId)
{
	//获取当前帖子的点赞总数，即统计帖子投票表中投票数大于零的记录数
	var upvotes = this.DataAccess.Count<Post.PostVoting>(Condition.Equal(nameof(Post.PostVoting.PostId), postId) & Condition.GreaterThan(nameof(Post.PostVoting.Value), 0));

	//获取当前帖子的被踩总数，即统计帖子投票表中投票数小于零的记录数
	var downvotes = this.DataAccess.Count<Post.PostVoting>(Condition.Equal(nameof(Post.PostVoting.PostId), postId) & Condition.LessThan(nameof(Post.PostVoting.Value), 0));

	//更新指定帖子的累计点赞总数和累计被踩总数
	return this.DataAccess.Update(Model.Naming.Get<Post>(), new
	{
		PostId = postId,
		TotalUpvotes = upvotes,
		TotalDownvotes = downvotes,
	}) > 0;
}
```
{% endcode %}

The current implementation counts positive and negative vote records separately and then updates the post count; it does not sum the Value. Understanding this difference explains vote count and vote weighting. See [Write Operations](writing.md) for the transaction boundary of voting.

## What Else to Consider Before Returning a Collection

Stable sorting, pagination, field ranges, and identity constraints work together to determine whether a query is appropriate for a business page. Collections are usually lazy enumerations, and the caller should consume them within the effective lifecycle. The enumerator should also be released when it ends early. Discussions' filter uses [FilteredResult](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Data/FilteredResult.cs) to retain paginated notifications and process the body. Discussions There is currently no native data command call. For the format and driver boundary of the named command, please refer to [Mapping Files](mapping.md).
