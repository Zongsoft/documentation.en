---
description: "Responsibilities and main types of Zongsoft.Data.Transaction class and Zongsoft.Data.Transactions namespace."
icon: rotate
---

# Zongsoft.Data.Transaction

`Zongsoft.Data.Transaction` provides lightweight ambient transaction objects in the core class library. Transaction status, transaction information, transaction stages and transaction participation registration types are located in the `Zongsoft.Data.Transactions` namespace. It is used to express a commit or rollback application layer transaction in a data service, batch or cross-component operation.

{% hint style="warning" %}
The `Zongsoft.Transactions` namespace has been removed in the core class library; new code should use `Zongsoft.Data.Transaction` and `Zongsoft.Data.Transactions` instead.
{% endhint %}

## Main Responsibilities

* Maintain the ambient transaction in the current asynchronous context via `Transaction.Current`.
* Supports the creation of transactions with common isolation levels using factory methods such as `ReadCommitted()`, `RepeatableRead()`, and `Serializable()`.
* Provides synchronous and asynchronous commit/rollback, and rollback when uncommitted transactions are released.
* Support transaction participant registration and stage notification through `IEnlistment`, `EnlistmentContext` and `EnlistmentPhase`.
* Provides a unified transaction abstraction for data services, batch processing, and cross-component operations.

## Typical Usage

Discussions puts the master record, topic content posts, and author statistics into transactions when creating a topic. For complete use cases of synchronous and asynchronous, see [Transactions and Consistency](../../data/transactions.md).

If you need to make the component aware of transaction commit or rollback, you can implement `Zongsoft.Data.Transactions.IEnlistment` and register it to the current transaction. Registered objects receive phase `Commit` or `Rollback` notifications when the transaction completes.

## Related Resources

* [Zongsoft.Data](../data.md)
* [Data Engine](../../data/README.md)
* [Transaction source code](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Data/Transaction.cs)
* [Transactions source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Data/Transactions)

## Asynchronous Completion and Environment Scope

Below are the actual asynchronous insertion hooks for Discussions; the Posting, Utility, and Statistics methods all come from the module itself. The body file is cleaned up by MutateContentAsync when the database operation fails, it is not a resource that the database transaction automatically rolls back.

Asynchronous data operations can use `await using` to manage the transaction, and then wait for `CommitAsync` to let the caller observe the actual completion or failure:

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L336) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
protected override async ValueTask<int> OnInsertAsync(IDataDictionary<Models.Thread> data, ISchema schema, DataInsertOptions options, CancellationToken cancellation)
{
	cancellation.ThrowIfCancellationRequested();
	if(!data.TryGetValue(p => p.Post, out var post) || post == null || string.IsNullOrEmpty(post.Content))
		throw new InvalidOperationException("Missing content of the thread.");

	//确保数据模式含有“主题内容贴”复合属性
	schema.Include("Post{*}");

	//更新主题内容贴的相关属性
	post.Visible = false;
	post.Approved = await this.ServiceProvider.ResolveRequired<ForumService>().CanPublishAsync(data, cancellation);
	data.SetValue(p => p.Approved, post.Approved);
	schema.Include(nameof(Models.Thread.Approved));

	var content = DataDictionary.GetDictionary<Post>(post);
	return await Utility.MutateContentAsync(content, () => this.Posting.GetContentFilePath(content), async () =>
	{
		await using(var transaction = new Transaction())
		{
			//调用基类同名方法，插入主题数据
			var count = await base.OnInsertAsync(data, schema, options, cancellation);

			if(count < 1)
				return count;

			//更新发帖人关联的主题统计信息
			await this.SetMostRecentThreadAsync(data, cancellation: cancellation);

			//提交事务
			await transaction.CommitAsync(cancellation);

			return count;
		}
	}, cancellation);
}
```
{% endcode %}

Asynchronous participants receive completion notifications via `IEnlistment.OnEnlistAsync`. After the transaction begins to terminate, subsequent cancellation will not abort the commit/rollback that has already been performed; the cancellation request cannot be used as proof that the database is not committed.

`DisposeAsync` will first exit the current ambient transaction scope, and then asynchronously roll back the unfinished transaction to prevent the old scope from affecting subsequent code. It does not automatically include arbitrary HTTP requests, message acknowledgments, or external system operations into atomic transactions.

See [Transactions and Consistency](../../data/transactions.md) for connections, isolation, and cross-system boundaries in the data engine.
