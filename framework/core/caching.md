---
description: "Responsibilities and main types of the Zongsoft.Caching namespace."
icon: database
---

# Zongsoft.Caching

`Zongsoft.Caching` provides cache abstraction and memory cache implementation in the core class library to support in-process caching, cache expiration, capacity control, buffer refresh and cache event notification.

## Main Responsibilities

* Define cache access abstractions such as `IDistributedCache`.
* Provides in-process cache implementation and configuration such as `MemoryCache` and `MemoryCacheOptions`.
* Express cache expiration policies, cache priorities, eviction reasons, and cache change events.
* Provides `Spooler<T>` asynchronous buffer for batch refresh of high-frequency writes.
* Provide unified cache dependencies for other modules instead of directly binding a specific cache implementation.

## Type

<table data-view="cards">
	<thead>
		<tr>
			<th></th>
			<th></th>
			<th data-hidden data-card-target data-type="content-ref">Page</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><strong>MemoryCache</strong></td>
			<td>In-process caching, expiration policies, dependency tokens, retirement events, and capacity reminders.</td>
			<td><a href="caching/memory-cache.md">memory-cache.md</a></td>
		</tr>
		<tr>
			<td><strong>Spooler&lt;T&gt;</strong></td>
			<td>Channel-based asynchronous buffer for batch refresh of high-frequency writes.</td>
			<td><a href="caching/spooler.md">spooler.md</a></td>
		</tr>
	</tbody>
</table>

## Related Resources

* [Caching source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Caching)
