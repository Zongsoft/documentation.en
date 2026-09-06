---
description: "Query along the Discussions forum to verify mappings, identities, connections, and results."
icon: play
---

# Run Your First Data Query

For the first query, use Discussions which already has a forum interface. The premise is [Business plugin has been deployed](../../get-started/deploy-first-plugin.md), and the host is connected to its own isolation environment.

## 1. Check the Real Data Contract

Also read the selected database script under Models/Forum.cs, [Zongsoft.Discussions.mapping](https://github.com/Zongsoft/discussions/blob/main/src/Zongsoft.Discussions.mapping) and database in Discussions. Forum uses SiteId and ForumId composite keys; external serial numbers require corresponding services; Message also has ClickHouse driver tags. These constraints cannot be solved by simply modifying the database connection string.

Database scripts may rebuild objects and should be reviewed first and executed only on temporary databases. The mapping file does not automatically create these tables.

## 2. Verify Accessor, Connection and Identity

Source: [src/Module.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Module.cs#L52) (excerpt; see source for context).

{% code title="Module.cs" %}
```csharp
public IDataAccess Accessor => _accessor ??= this.Services.ResolveRequired<IDataAccessProvider>().GetAccessor(this.Name);
```
{% endcode %}

The accessor name is Discussions. The connection configuration and driver must allow the accessor to select the correct data source; the identity needs to include the Discussions scheme and SiteId, see [Connection Configuration](connections.md) and [Certification](../security/authentication.md).

## 3. Use Forum Requests in the Repository

The following only excerpts the request line, omitting Host and Authorization; page is the environment variable in the original request, provided by your request tool:

Source: [docs/http/forum.http](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/docs/http/forum.http#L2) (excerpt; see source for context).

{% code title="forum.http" %}
```http
GET /Discussions/Forums?page={{page}} HTTP/1.1
```
{% endcode %}

This request reads the forum collection. Visibility rules are handled by the ForumService, and site conditions are supplemented by data validators; an empty collection may indicate that there is no data in the current scope, not necessarily a query failure.

## 4. Follow the Service Check Results

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

This is the actual call to read the pinned topic in the same module: conditions, data schema, pagination and sorting all have clear sources. After understanding the interface collection query, you can set breakpoints in this method and data filter to observe how the accessor works.

## Success Criteria and Troubleshooting

Confirm that the request enters the correct controller, the accessor selects the expected connection, the data only belongs to the current site, the field shape conforms to the schema, and the unaudited body is not exposed. Use independent test data for further writes and transaction verification; do not treat a successful SELECT as acceptance of the entire forum business.

Check the mapping deployment when the entity cannot be found; check the plugin when the driver cannot be found; check the data source when the connection cannot be made; check the dependent assembly when the serial number service is missing. More reading: [mapping](mapping.md), [Query](querying.md), [service](services.md).
