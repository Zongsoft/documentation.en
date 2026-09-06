---
description: "Understand business service boundaries through ThreadService, DataValidator and query filters."
icon: book-open
---

# Data Services


Discussions' service inherits DataServiceBase and organizes data access into business actions. They share a dependency on mapping, current identity, and plugin assembly, so they are not CRUD wrappers that can run independently just by getting a database connection.

## Models, Conditional Models and Service Registration

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L41) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
[Service(nameof(ThreadService))]
[DataService(typeof(ThreadCriteria))]
public class ThreadService : DataServiceBase<Models.Thread>
{
	#region 成员字段
	private PostService _posting;
	#endregion

	#region 构造函数
	public ThreadService(IServiceProvider serviceProvider) : base(serviceProvider) { }
```
{% endcode %}

Thread is the business data model, ThreadCriteria describes the query conditions, and the Service attribute allows the module container to discover services. General query and writing are provided by the base class, and actions such as auditing and pinning are implemented by derived services.

## A Service Depends on Another Service

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

When Posting is used, it obtains the PostService from the service container to which it belongs. Container-managed services are not released one by one here, nor should the requesting user be saved in a static field.

## Three Rules Placed in Different Locations

| rules | Discussions entrance | function |
| --- | --- | --- |
| business action | ThreadService.Approve、PostService.Upvote | Organization conditions, writing and statistics |
| Site and audit fields | DataValidator | For action-restricted sites with SiteId, fill in the creator and time |
| Query result content | ThreadFilter、PostFilter | Block unapproved text and read external content |

Filter processes the returned content and cannot replace request authorization and query scope. The validator cannot replace the business layer's judgment on rules such as locking, moderators, and authors.

## Process the Results After the Query Is Completed

Source: [src/Data/PostFilter.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Data/PostFilter.cs#L44) (excerpt; see source for context).

{% code title="PostFilter.cs" %}
```csharp
public void OnFiltered(DataSelectContextBase context)
{
	if(context.Result == null)
		return;

	var identity = context.Principal?.Identity;
	context.Result = FilteredResult.Create(context, item => Filter(item, identity));
}
```
{% endcode %}

OnFiltering occurs before the query; there is no final result to wrap yet. OnFiltered only establishes delayed filtering for results. [FilteredResult](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Data/FilteredResult.cs) only adapts to the synchronous/asynchronous interface and forwards paging notifications. Enumeration and filtering directly use [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) implementation. See [Filters](../core/components/filter.md) for its version requirements and lifecycle. The file cannot be read repeatedly in the Current property, otherwise the text may be mistaken for the path if the same element is read multiple times.

## Integration HTTP

The controller binds the service through generic parameters, and additional actions call the same business method. See [Requests and Data Service APIs](../web/data-services.md) for details. Maintain clear response semantics for audit failure, non-existence, and lack of permission; do not re-assemble a set of updates at the interface layer that can bypass service conditions.


## Audit Rules When Created

Forum.Approvable in Discussions indicates whether the post needs to be reviewed. When ThreadService creates a topic, the body Post is written cascaded through the data engine; this process does not automatically call PostService.OnInsert. Therefore, both the topic entrance and the ordinary reply entrance must obtain the forum rules explicitly, and cannot rely on the default value of Approved in the mapping, nor can they trust the audit flag passed in the request.

Source: [src/Services/ForumService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ForumService.cs#L132) (excerpt; see source for context).

{% code title="ForumService.cs" %}
```csharp
internal async ValueTask<bool> CanPublishAsync(IDataDictionary<Models.Thread> thread, CancellationToken cancellation)
{
	cancellation.ThrowIfCancellationRequested();
	var forum = await this.DataAccess.SelectAsync<Forum>(GetForumCriteria(thread), nameof(Forum.Approvable), cancellation: cancellation).FirstOrDefault(cancellation);
	if(forum == null)
		throw new InvalidOperationException("The specified forum does not exist.");

	return !forum.Approvable || this.Principal?.Identity?.IsAuthenticated == true && await this.IsModeratorAsync(thread.GetValue(p => p.ForumId), cancellation: cancellation);
}
```
{% endcode %}

CanPublishAsync uses the same rules as synchronous CanPublish: if the forum does not exist, it will fail; forums that do not require review can publish directly, and only the moderators of the forum will be allowed to pass directly when review is required. GetForumCriteria retains the explicitly specified SiteId and ForumId, and the data validator additionally appends the current site constraints.

The topic entrance will write the results to Thread.Approved and Post.Approved at the same time. For ordinary replies, first search the forum to which it belongs based on ThreadId; both will put Approved into the actual writing mode to prevent the custom field list from skipping the value determined by the server. The approval action still uses ThreadService.Approve, which is two business steps with "determining the initial status according to the forum rules when creating".

This rule has isolated data access stand-ins covering sync/async, topic/reply, moderation required/no moderation required, moderator/normal user combinations. Cascading writes, transaction rollback and permission configuration of the real database still need to be accepted in the deployment environment; see [Discussions Checker](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/test/Program.cs) for regression use cases.
