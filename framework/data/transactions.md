---
description: "Understand transaction boundaries, commits, and external side effects from publishing topics and update statistics."
icon: book-open
---

# Transactions and Consistency


Posting a topic is not an isolated insertion: the topic, body post, and associated statistics must work together. Discussions organize this unit of work in ThreadService.OnInsert.

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L200) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
protected override int OnInsert(IDataDictionary<Models.Thread> data, ISchema schema, DataInsertOptions options)
{
	if(!data.TryGetValue(p => p.Post, out var post) || post == null || string.IsNullOrEmpty(post.Content))
		throw new InvalidOperationException("Missing content of the thread.");

	//确保数据模式含有“主题内容贴”复合属性
	schema.Include("Post{*}");

	//更新主题内容贴的相关属性
	post.Visible = false;
	post.Approved = this.ServiceProvider.ResolveRequired<ForumService>().CanPublish(data);
	data.SetValue(p => p.Approved, post.Approved);
	schema.Include(nameof(Models.Thread.Approved));

	var content = DataDictionary.GetDictionary<Post>(post);
	return Utility.MutateContent(content, () => this.Posting.GetContentFilePath(content), () =>
	{
		using(var transaction = new Transaction())
		{
			//调用基类同名方法，插入主题数据
			var count = base.OnInsert(data, schema, options);

			if(count < 1)
				return count;

			//更新发帖人关联的主题统计信息
			this.SetMostRecentThread(data);

			//提交事务
			transaction.Commit();

			return count;
		}
	});
}
```
{% endcode %}

## Why Create a Transaction Here

The method first confirms that the topic body exists and includes the Post navigation into the data schema. Then insert the relationship between the topic and the text, then update the statistics of the forum and author to which it belongs, and finally submit it. If the insertion is unsuccessful, it will return directly and Commit will not be executed.

This business code uses the ambient transaction mechanism of Zongsoft.Data.Transaction; it does not directly create a database connection transaction. How to add underlying resources and how to vote for nested scopes, see [Core Transactions](../core/data/transactions.md).

## Statistics Also Belong to Business Status

SetMostRecentThread updates the forum's last post information, author's cumulative number of topics, and recent topic information. If you only copy Insert and miss these steps, the topic will exist but the list summary and personal statistics will be inconsistent.

## Transaction Coverage

The same service call may also trigger a body file write. Database transactions cannot automatically roll back file storage; external files need to be compensated, retried, or cleaned up afterwards. Atomicity across drivers and connections must also be based on the actual participating resources and cannot be guaranteed solely by the using scope.

## Verify This Path

Use the isolation site and temporary database to check the topic, text, forum summary and author statistics after successful publication; then simulate the insertion or statistics phase failure and check the rollback results and file residues. Also verify that authorization failures leave no side effects.
