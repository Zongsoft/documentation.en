---
description: "Parameters parameter packs, type keys, string keys, chain building and JSON serialization."
icon: sliders
---

# Parameters

`Parameters` is a lightweight parameter package that can save parameters by string name or [`Type`](https://learn.microsoft.com/en-us/dotnet/api/system.type) _[Source](https://source.dot.net/#System.Private.CoreLib/Type.cs)_. It is commonly used for command invocations, service contexts, data access options, event parameters, and passing small amounts of context data across layers.

## Type Relationship

| Type | Description |
| --- | --- |
| `Parameters` | Parameter container, implementing [`IDictionary<object, object>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.idictionary-2) _[Source](https://source.dot.net/#System.Private.CoreLib/IDictionary.cs)_. |
| `Parameters.Json` | JSON converter implementation for `Parameters`. |
| `ParametersUtility` | Provides extension methods for adding parameters in a chained manner. |

## String Keys and Concurrent Initialization

String keys ignore case. `null` names are converted to empty strings.

Source: [framework/Zongsoft.Core/test/Collections/ParametersTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/ParametersTest.cs#L18) (excerpt; see source for context).

{% code title="ParametersTest.cs" %}
```csharp
public async Task GetOrAdd_ConcurrentSameName_CreatesSingleValueAsync()
{
	const int COUNT = 64;
	var parameters = new Parameters();
	var factoryCalls = 0;
	var start = new TaskCompletionSource(TaskCreationOptions.RunContinuationsAsynchronously);
	var tasks = new Task<object>[COUNT];

	for(var index = 0; index < tasks.Length; index++)
	{
		tasks[index] = Task.Run(async () =>
		{
			await start.Task;
			return parameters.GetOrAdd("Shared", _ =>
			{
				Interlocked.Increment(ref factoryCalls);
				return new object();
			});
		});
	}

	start.SetResult();
	var results = await Task.WhenAll(tasks);

	Assert.Equal(1, factoryCalls);
	Assert.Single(parameters);
	Assert.Same(results[0], parameters.GetValue("shared"));
	Assert.All(results, result => Assert.Same(results[0], result));
}
```
{% endcode %}

`TryGetValue<TValue>(string name, out TValue value)` attempts to convert the original value to the target type.

## Type Key and Value Reuse

When saving parameters by type, they can be read directly through generics. If an exact matching type key is not found when reading, `Parameters` will look for an assignable type key.

Source: [framework/Zongsoft.Core/test/Collections/ParametersTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/ParametersTest.cs#L49) (excerpt; see source for context).

{% code title="ParametersTest.cs" %}
```csharp
public void GetOrAdd_TypeKey_ReusesExistingValueWithoutCallingFactory()
{
	var parameters = new Parameters();
	var expected = parameters.GetOrAdd(() => new ParameterMarker());

	var actual = parameters.GetOrAdd<ParameterMarker>(() => throw new InvalidOperationException("The existing value must win."));

	Assert.Same(expected, actual);
	Assert.Same(expected, parameters.GetValue<ParameterMarker>());
}
```
{% endcode %}

When setting parameters by type, the value must be assignable to the declared type; if the declared type is a non-null value type, it cannot be set to `null`.

## Chain Building

`ParametersUtility` allows existing parameter packages to continue adding parameters in a chain.

PostService.OnInsert of Discussions reads the associated object named Thread in options.Parameters, which is used to distinguish topic content posts from ordinary replies. Callers of this branch must explicitly pass in the associated object; the presence of the Post composite property in the mapping itself does not automatically populate Parameters. See [post write](../../data/writing.md).

`Append` can merge entries from another `Parameters` into the current instance.

## JSON Serialization

The following is an excerpt from TestJson in the same test file. Parameters comes from the parameter package constructed in the first half of the method, covering named parameters, type keys, time values, byte arrays and IPerson test models; it is not an independent top-level program. ParameterMarkers in type key tests are also defined in the same file.

`Parameters.Json` provides a JSON converter for `Parameters`. String keys are output as attribute names, and type keys are output using the `$` prefix plus type alias.

Source: [framework/Zongsoft.Core/test/Collections/ParametersTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/ParametersTest.cs#L107) (excerpt; see source for context).

{% code title="ParametersTest.cs" %}
```csharp
var json = Serializer.Json.Serialize(parameters);
Assert.NotNull(json);
Assert.NotEmpty(json);

var result = Serializer.Json.Deserialize<Parameters>(json);
Assert.NotNull(result);
Assert.NotEmpty(result);
```
{% endcode %}

Complex objects are saved as objects containing `$type` and `$value`, and the values are restored by type alias when read.

## Related Resources

* [Parameters.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/Parameters.cs)
* [Parameters.Json.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/Parameters.Json.cs)
* [ParametersUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/ParametersUtility.cs)
* [ParametersTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/ParametersTest.cs)
