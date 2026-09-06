---
description: "AggregateExceptionUtility Aggregate exception handling tool."
icon: circle-info
---

# AggregateExceptionUtility

`AggregateExceptionUtility` provides the `Handle<TException>` extension method, which is used to find the specified exception type from [`AggregateException`](https://learn.microsoft.com/en-us/dotnet/api/system.aggregateexception) _[Source](https://source.dot.net/#System.Private.CoreLib/AggregateException.cs)_ and call the handler function.

Source: [framework/externals/wechat/gateway/Controllers/FallbackController.cs](https://github.com/Zongsoft/framework/blob/main/externals/wechat/gateway/Controllers/FallbackController.cs#L72) (excerpt; see source for context).

{% code title="FallbackController.cs" %}
```csharp
return (IActionResult)ae.Handle<OperationException>(ex => ex.Reason switch
{
	nameof(OperationException.Unfound) => this.NotFound(new { ex.Reason, ex.Message }),
	nameof(OperationException.Unsupported) => this.BadRequest(new { ex.Reason, ex.Message }),
	nameof(OperationException.Unprocessed) => this.UnprocessableEntity(new { ex.Reason, ex.Message }),
	nameof(OperationException.Unsatisfied) => this.StatusCode(StatusCodes.Status412PreconditionFailed, new { ex.Reason, ex.Message }),
	_ => this.StatusCode(StatusCodes.Status500InternalServerError, new { ex.Reason, ex.Message }),
});
```
{% endcode %}

## Actual Call and Return Value

Discussions This tool is not used directly; the above is a real processing fragment after the WeChat callback controller captures System.AggregateException. It converts the Reason of the internal OperationException into the corresponding HTTP result, and the returned object is the return value of the extension method. The full controller also separately handles directly thrown OperationExceptions.

## Match Boundary

The tool Flattenes nested exceptions before looking for the first matching type. Immediately call the delegate and return after it is found, and will not continue to traverse the remaining exceptions; the return of true by the delegate does not mean "all exceptions have been processed". If there is no match, the unprocessed items will be reassembled into an aggregate exception and an exception will be thrown. Returns null when the input exception is null; throws a parameter exception when the handler delegate is null.

{% hint style="warning" %}
🚨 This is different from the itemized Handle semantics that comes with System.AggregateException. When you need to fully record all failures of a batch, you should retain the original exception collection; also do not assume that awaiting a failed task will always throw an aggregated exception. This article uses the processing branch after the controller has captured the aggregated exception.
{% endhint %}

## Related Resources

* [AggregateExceptionUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/AggregateExceptionUtility.cs)
