---
description: "OperationException Operation exception."
icon: circle-info
---

# OperationException

`OperationException` is a frame-level operation exception. It expresses the failure type via `Reason` and provides a set of static factory methods.

| factory method | Semantics |
| --- | --- |
| `Argument` | The parameter is incorrect. |
| `Unknown` | Unknown error. |
| `Unfound` | The target does not exist. |
| `Unsatisfied` | The conditions are not met. |
| `Unprocessed` | The operation was not processed. |
| `Unsupported` | This operation is not supported. |

Source: [framework/Zongsoft.Core/test/Messaging/MessageQueueBaseTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Messaging/MessageQueueBaseTest.cs#L59) (excerpt; see source for context).

{% code title="MessageQueueBaseTest.cs" %}
```csharp
public async Task UnsupportedDelayFailsBeforeDriverOperation()
{
	using var queue = new TestQueue();
	var options = new MessageEnqueueOptions(TimeSpan.FromSeconds(1));

	var exception = await Assert.ThrowsAsync<OperationException>(() => queue.ProduceAsync("tests/delay", ReadOnlyMemory<byte>.Empty, options).AsTask());

	Assert.Equal(nameof(OperationException.Unsupported), exception.Reason);
	Assert.Contains(MessageQueueFeature.Delay.Name, exception.Message);
	Assert.Equal(0, queue.ProduceCount);
}
```
{% endcode %}

## Practical Use Cases

The above is a capability test of the framework's message abstraction: queues that do not support delay reject send requests with the delay option. The test checks that Reason is Unsupported and confirms that the driver has not yet been executed. TestQueue belongs to the test fixture and is not connected to the actual middleware. Discussions There is no implementation of this queue, and related business exceptions should be based on its services.

## Boundaries and Error Responses

Reason is a classification for program judgment, and Message is for diagnostics or user instructions; do not determine business branches by matching error text. The factory only creates exceptions and does not automatically throw, retry, or transition HTTP status. See [Aggregation exception handling](aggregate-exception-utility.md) for how the gateway converts it into a response.

The caller can determine the cause of the exception through attributes such as `IsArgument`, `IsUnknown`, `IsUnfound`, and `IsUnsupported`.

## Related Resources

* [OperationException.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/OperationException.cs)
