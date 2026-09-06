---
description: "Notification change token tool."
icon: message
---

# Notification

`Notification` is used to create or obtain change tokens, often in conjunction with cache dependencies, configuration refresh, and status observation.

| member | Description |
| --- | --- |
| `Notified` | A token that is always in a changed state. |
| `GetToken(CancellationToken)` | Wrap a cancelable token as a change token. |
| `GetToken(CancellationTokenSource)` | Creates a change token from the cancellation source; returns `Notified` when `null` is passed in. |

Source: [framework/Zongsoft.Core/src/Data/DataAccessBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Data/DataAccessBase.cs#L120) (excerpt; see source for context).

{% code title="DataAccessBase.cs" %}
```csharp
public IChangeToken Disposed => Notification.GetToken(_cancellation);
```
{% endcode %}

The above is the Disposed attribute of the framework data accessor. Discussions Obtains an accessor from Module.Accessor and uses this property as a dependency when caching the accessor: releasing the accessor will trigger an invalidation, causing it to be re-created the next time it is acquired. See [memory cache](../caching/memory-cache.md) for the complete link.

Notification itself does not send business messages, nor does it hold reload logic; it converts a cancellation notification into an observable change signal for the cache and configuration components. The token cannot be reset after it expires, and a new cancellation source should be created when the next round of observation is required.

`Notified` is suitable for expressing dependencies that immediately expire, such as scenarios where cache items need to be forced to expire after they are created.

## Related Resources

* [Notification.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/Notification.cs)
