---
description: "Zongsoft.Components Weighter Smooth weighted polling selector."
icon: scale-balanced
---

# Weighter

`Weighter<T>` is an in-process smooth weighted polling selector. It is suitable for distributing requests by weight among multiple equal candidates, such as multiple data sources, suppliers, gateways, connections or handler instances; the higher the weight, the more times it will be selected, but the selection results will be spread as much as possible throughout the scheduling sequence.

Compared with ordinary weighted polling, the focus of `Weighter<T>` is "smoothing". If the weight is `4:2:1`, ordinary polling can easily get a concentrated sequence such as `A,A,A,A,B,B,C`, while smooth weighted polling will be closer to `A,B,A,C,A,B,A`, thus preventing high-weight nodes from being overwhelmed by too many requests in a short period of time.

{% hint style="info" %}
For a comparison of algorithm background, random algorithm, consistent hashing, and Nginx and LVS weighted polling algorithms, you can read "[Smooth weighted round robin equalization algorithm](https://blog.zongsoft.com/ping-hua-de-jia-quan-lun-xun-jun-heng-suan-fa)".
{% endhint %}

## Design Intent

`Weighter<T>` solves the local selection problem: when a set of candidates are available and can all handle the same type of requests, but have different capacities, costs, or priorities, the caller wants to allocate traffic proportionally while minimizing request spikes in a short period of time.

It does not save external distributed state and is not responsible for health checks, failure circuit breaking, or global quotas. When multiple processes each hold `Weighter<T>`, only the selection sequence within each process can be guaranteed to be smooth; if cross-process consistency is required, it should usually be handled in the upper-layer load weighted selector, scheduler or service governance mechanism.

## Algorithm Process

Smooth weighted polling maintains the current weight for each candidate. Each time it is selected, the algorithm first accumulates the static weights of all candidates, then selects the candidate with the highest current weight, and finally subtracts the total weight from the current weight of the candidate. After multiple cycles, high-weighted candidates will be selected more often, but the results will be distributed as evenly as possible in the sequence.

Assume that the weights of `A`, `B`, and `C` are `4`, `2`, and `1` respectively, and the total weight is `7`. The calculation of a complete cycle is as follows:

| rounds | Current weight after accumulation | Selected items | Current weight after selection |
| --- | --- | --- | --- |
| 1 | `{ 4, 2, 1 }` | `A` | `{ -3, 2, 1 }` |
| 2 | `{ 1, 4, 2 }` | `B` | `{ 1, -3, 2 }` |
| 3 | `{ 5, -1, 3 }` | `A` | `{ -2, -1, 3 }` |
| 4 | `{ 2, 1, 4 }` | `C` | `{ 2, 1, -3 }` |
| 5 | `{ 6, 3, -2 }` | `A` | `{ -1, 3, -2 }` |
| 6 | `{ 3, 5, -1 }` | `B` | `{ 3, -2, -1 }` |
| 7 | `{ 7, 0, 0 }` | `A` | `{ 0, 0, 0 }` |

Therefore, the number of hits in a cycle corresponds to `4:2:1`, the sequence is `A,B,A,C,A,B,A`, and returns to the initial state after the cycle ends.

## Common Members

| member | Description |
| --- | --- |
| `Weighter(entries, weightThunk)` | Create a selector using a candidate set and an optional weight resolution function. |
| `Add(value, weight)` | Add a candidate and its fixed weight. |
| `Add(value, weighter)` | Add a candidate and calculate the candidate's weight via a function. |
| `Remove(value)` | Remove candidates. |
| `Get()` | Select the next candidate. |
| `Clear()` | Clear all candidates. |
| `Count` | Get the current number of candidates. |

The constructor ignores `null` candidates in the initial collection. When the weight parsing function is not provided, the default weight is `100`; the final effective weight will not be less than `1`. If you add a candidate via `Add(value, weight)` and pass in a non-positive weight, it will fallback to the default weight.

## Applicable Scenarios

* The data access framework selects data sources based on read-write mode and weight.
* Share volume among multiple SMS providers by capacity or cost.
* Smooth selection between multiple gateways, connections, and queue consumers.
* When there are multiple handlers for the same function, the executor is selected according to priority or weight.
* Perform lightweight load balancing within the local process without introducing external load weighted selector.

Source: [framework/Zongsoft.Core/test/Components/WeighterTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Components/WeighterTest.cs#L13) (excerpt; see source for context).

{% code title="WeighterTest.cs" %}
```csharp
var servers = new []
{
	new Server("A", 4),
	new Server("B", 2),
	new Server("C", 1),
};

var weighter = new Weighter<Server>(servers, server => server.Weight);

//Round 1
var server = weighter.Get();
Assert.NotNull(server);
Assert.Equal("A", server.Name);
Assert.Equal(4, server.Weight);
```
{% endcode %}

The above is from WeighterTest.TestGet1, Server is the test model in the same file, and the full test verifies A, B, A, C, A, B, A in sequence. Discussions are not configured with multiple equivalent data sources; the above weight test is used to understand the framework algorithm.

## Note on Usage

Each call to `Get()` advances the current weight internally, so it is not a pure query method. Callers should think of `Weighter<T>` as a stateful selector rather than a cache that can read the same result repeatedly.

When the candidate is empty, `Get()` returns the target type default value. Typically `null` for reference types, the caller should handle the empty collection case before sending a request, establishing a connection, or executing a handler.

Candidates can be dynamically adjusted via `Add(...)`, `Remove(...)` and `Clear()`. For temporarily unavailable nodes, it is recommended to remove them from the selector first, or complete availability filtering before creating the selector; `Weighter<T>` itself will not detect the node health status, nor will it automatically reduce the weight based on call failure.

## Related Implementation

The data source selector of [`Zongsoft.Data`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data) will maintain `Weighter<T>` for readable and writable data sources respectively, and select the read library or write library according to the data access method. This scenario reflects the typical boundaries of `Weighter<T>`: it is only responsible for selecting by weight among the filtered candidate set, and the downstream data access context, command variability, and data source mode are still judged by the data framework.

## Reference

* [Weighter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/Weighter.cs)
* [DataSourceSelector.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/src/Common/DataSourceSelector.cs)
* [Smooth weighted round robin equalization algorithm](https://blog.zongsoft.com/ping-hua-de-jia-quan-lun-xun-jun-heng-suan-fa)
