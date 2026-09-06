---
description: "Illustrate real writing and its side effects with post voting, topic moderation, and browsing history."
icon: book-open
---

# Write Operations


Data writing changes records and may affect statistics, audit status, and file storage. Discussions Place these rules in a business service; callers should prefer using service actions rather than modifying tables directly from the controller.

## Create a New Model and Write a Vote

Source: [src/Services/PostService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/PostService.cs#L50) (excerpt; see source for context).

{% code title="PostService.cs" %}
```csharp
public bool Upvote(ulong postId, byte value = 1)
{
	if(value == 0)
		value = 1;

	var userId = this.Principal.Identity.GetIdentifier<uint>();

	using(var transaction = new Transaction())
	{
		this.DataAccess.Delete<Post.PostVoting>(
			Condition.Equal(nameof(Post.PostVoting.PostId), postId) &
			Condition.Equal(nameof(Post.PostVoting.UserId), userId));

		this.DataAccess.Insert(Model.Build<Post.PostVoting>(voting =>
		{
			voting.PostId = postId;
			voting.UserId = userId;
			voting.Value = (sbyte)Math.Min(value, (sbyte)100);
			voting.Timestamp = DateTime.Now;
		}));

		//如果帖子投票统计信息更新成功
		if(this.SetPostVotes(postId))
		{
			//提交事务
			transaction.Commit();

			//返回成功
			return true;
		}
	}

	return false;
}
```
{% endcode %}

The actual process first deletes the current user's old votes for this post, then creates a new record through Model.Build, and finally recalculates the number of positive and negative votes. The transaction is committed only if the statistics update is successful. Value is limited to 1 to 100 and counts the number of records rather than the sum of weights.

## Deletion Also Requires Business Scope

The above Delete specifies both PostId and UserId: what is deleted is the current user's old vote for the current post. The absence of any one of these conditions will increase the scope of the effect. The general Delete API cannot replace the authorization judgment of the current identity and target resources.

## Update Topic Status

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L103) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
public bool SetLocked(ulong threadId, bool value)
{
	return this.DataAccess.Update<Models.Thread>(new
	{
		IsLocked = value,
	}, Condition.Equal(nameof(Models.Thread.ThreadId), threadId) & GetIsModeratorCriteria()) > 0;
}
```
{% endcode %}

Locking is an explicit business action, and moderator conditions along with topic numbers control updates. The meaning of the fields, whether the reply entry checks for locking, and interface feedback need to be checked according to the complete call chain; just setting IsLocked does not mean that all custom write entries will automatically block replies.

## Add or Update Browsing History

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L283) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
private void SetHistory(ulong threadId)
{
	//新增或更新当前用户对指定主题的浏览记录（自动递增浏览次数）
	this.DataAccess.Upsert<History>(new
	{
		UserId = this.Principal.Identity.GetIdentifier<uint>(),
		ThreadId = threadId,
		ViewedCount = Operand.Field(nameof(History.ViewedCount)) + 1,
		MostRecentViewedTime = DateTime.Now,
	});
}
```
{% endcode %}

Upsert performs a new or update around the view record key, while incrementing ViewedCount. Whether the corresponding write expression is supported depends on the database driver; do not generalize one driver's SQL form to all databases.

## Content File and Database Are Not in the Same Transaction

Long text may be saved to a file first, and then the path is written to the database. Database rollback does not automatically delete object storage files, so related services use failure compensation. Production integration also requires exception checking, cancellation, retries, and old file cleanup. See [file system](../core/io.md) and [affairs](transactions.md) for details.

Discussions does not have a business process that directly calls IDataAccess.Import; bulk import capabilities still require independent verification of site fields, permissions, and duplicate data policies.
