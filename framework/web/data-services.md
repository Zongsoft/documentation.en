---
description: "The topic Review Controller in Discussions explains the division of labor between Web and Business Services."
icon: route
---

# Requests and Data Service APIs


The Discussions Web plugin adapts the controller to HTTP and lets business services handle forum rules. ThreadController inherits the generic ServiceController, and the model and service are explicitly specified by type parameters.

Source: [src/api/Controllers/ThreadController.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/api/Controllers/ThreadController.cs#L40) (excerpt; see source for context).

{% code title="ThreadController.cs" %}
```csharp
[Authorization]
[ControllerName("Threads")]
public class ThreadController : ServiceController<Thread, ThreadService>
{
	#region 公共方法
	[ActionName("Approve")]
	[HttpPost("{id}/Approve")]
	public object Approve(ulong id)
	{
		return this.DataService.Approve(id) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Hidden")]
	[HttpPost("{id}/Hidden")]
	public object Hidden(ulong id)
	{
		return this.DataService.Visible(id, false) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Visible")]
	[HttpPost("{id}/Visible")]
	public object Visible(ulong id)
	{
		return this.DataService.Visible(id, true) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Lock")]
	[HttpPost("{id}/Lock")]
	public object Lock(ulong id)
	{
		return this.DataService.SetLocked(id, true) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Unlock")]
	[HttpPost("{id}/Unlock")]
	public object Unlock(ulong id)
	{
		return this.DataService.SetLocked(id, false) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Pin")]
	[HttpPost("{id}/Pin")]
	public object Pin(ulong id)
	{
		return this.DataService.SetPinned(id, true) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Unpin")]
	[HttpPost("{id}/Unpin")]
	public object Unpin(ulong id)
	{
		return this.DataService.SetPinned(id, false) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Valued")]
	[HttpPost("{id}/Valued")]
	public object Valued(ulong id)
	{
		return this.DataService.SetValued(id, true) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Unvalued")]
	[HttpPost("{id}/Unvalued")]
	public object Unvalued(ulong id)
	{
		return this.DataService.SetValued(id, false) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Global")]
	[HttpPost("{id}/Global")]
	public object Global(ulong id)
	{
		return this.DataService.SetGlobal(id, true) ? this.NoContent() : this.NotFound();
	}

	[ActionName("Unglobal")]
	[HttpPost("{id}/Unglobal")]
	public object Unglobal(ulong id)
	{
		return this.DataService.SetGlobal(id, false) ? this.NoContent() : this.NotFound();
	}
	#endregion
```
{% endcode %}

## Routes, Actions and Return Values

ControllerName specifies the controller name as Threads; the module provides the Discussions area and the base class provides the common data interface. The routing fragment of Approve is the topic number plus Approve. If the service is successfully called, 204 will be returned. If there is no matching update, 404 will be returned. The final address also depends on the host PathBase and routing conventions and cannot copy the singular name or api prefix from the old docs/api.md.

## Service Determines Business Conditions

ThreadService.Approve also determines the topic number, unapproval, and moderator qualifications. The controller does not directly update Approved, allowing other entries to reuse service logic. For updates on conditions and associated text, see [Conditions and Operands](../data/conditions-and-operands.md).

## Universal CRUD and Business Actions

The base class provides entrances such as query, counting, import and export; actual availability is also limited by data service capabilities and authorization configuration. Don't assume that just because a controller is inherited that all operations should be open to all users. New actions need to check identity, SiteId, target resource permissions and side effects.

## Query Mode and Request Scope

ForumController gets data schema from request headers and pagination from query parameters. This allows the same service to support different return shapes, and also requires the service to control sensitive fields, navigation costs, and audit decision fields. See [Data Schemas](../data/schema.md) for pattern syntax.

[Deploy Controller Plugins](controllers.md) before running, and check routing using the request structure in repository docs/http in your own isolated environment.
