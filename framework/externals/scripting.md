---
description: "Choose Lua, Python, or Scriban with expression contracts and manage syntax, variables, and runtime isolation."
icon: code
---

# Scripts and Expressions

Expressions are good for abstracting small-scale, audited calculation rules out of business code, such as price calculations or field transformations. Scripting languages can express more complex control flows. With increased flexibility, type validation, execution costs, and accessibility must also be explicitly constrained by the application.

## Unified Calling and Different Languages

All three plugins register `IExpressionEvaluator`, which matches by name, but does not automatically translate the language syntax.

| Name | realize | How to write the same calculation | Deployment concerns |
| --- | --- | --- | --- |
| `Lua` | NLua / KeraLua | `return x + y` | Native libraries matching platform and architecture |
| `Python` | IronPython | `x + y` | Deploy the `lib` standard library; not the system CPython environment |
| `Scriban` | Scriban pure script evaluation | `x + y` | Do not treat text with template delimiters directly as a pure expression |

Whether a library can be imported into Python depends on IronPython compatibility and deployment content. Installing the system Python package does not mean that the runtime is available.

## Called from Business Plugin

Discussions There is currently no script evaluation business. The Lua plugin's JSON round-trip test is used here: first serialize the script object, then pass it back to the script through the variable text, modify the id and check the result. This allows you to see the boundaries between scripting languages, host variables, and return types.

Source: [framework/externals/lua/test/LuaExpressionEvaluatorTest.cs](https://github.com/Zongsoft/framework/blob/main/externals/lua/test/LuaExpressionEvaluatorTest.cs#L121) (excerpt; see source for context).

{% code title="LuaExpressionEvaluatorTest.cs" %}
```csharp
public void TestEvaluateSerializeJson()
{
	using var evaluator = new LuaExpressionEvaluator();

	var result = evaluator.Evaluate(@"obj = {id = 100, name=""name""}; return Json:Serialize(obj);");
	Assert.NotNull(result);

	var variables = new Dictionary<string, object>() { { "text", result } };
	result = evaluator.Evaluate(@"obj = Json:Deserialize(text); obj.id=200; return obj;", variables);
	Assert.NotNull(result);
	Assert.IsAssignableFrom<IDictionary<string, object>>(result);

	if(result is IDictionary<string, object> dictionary)
	{
		Assert.Equal(2, dictionary.Count);
		Assert.Equal(200L, dictionary["id"]);
		Assert.Equal("name", dictionary["name"]);
	}
}
```
{% endcode %}

Python also has [JSON round trip testing](https://github.com/Zongsoft/framework/blob/main/externals/python/test/PythonExpressionEvaluatorTest.cs), using Json.Deserialize(text) and Python dictionary indexing; the Lua colon syntax above cannot be executed directly. Scriban has not found the same test project yet, and its calling and variable import methods should be based on [ScribanExpressionEvaluator](https://github.com/Zongsoft/framework/blob/main/externals/scriban/src/ScribanExpressionEvaluator.cs).

In the plugin host, the business relies on the IExpressionEvaluator contract and is implemented via [FindRequired](../core/services/locating.md) by Lua, Python or Scriban name choice; the test above constructs the instance itself and is therefore responsible for the release. Discussions' first business plugin tutorial does not include script execution steps.

The current Scriban implementation will read `variables.Count` directly, and an empty dictionary should be passed in even if there are no variables. The default value of the optional parameter cannot be regarded as supporting empty object input.

## Variables and Return Values

Each call creates its own set of variables, validating return value types and scopes at application boundaries. Numeric widths, nulls, sets, and multiple return values are not completely equivalent between languages; do not directly cast arbitrary script results deep in the business.

Sharing `Global` is suitable for registering stable functions or constants when the application starts and should not be modified repeatedly in concurrent requests. When exposing the host object, only the necessary data and restricted operations should be passed in, rather than the complete service container.

## Runtime and Concurrency

Lua creates and releases its own Lua state every time it is evaluated; Python reuse engine switches runtime input and output during calls and resumes the previous stream in finally; Scriban establishes an evaluation context every time. These implementations cannot be uniformly summarized as "thread isolation by passing in an independent dictionary".

In particular, Python's IO and global state should be serialized within application-owned execution boundaries, or isolated by separate processes. The evaluators registered by the container are managed by the host; only the independent instances constructed directly are released by the constructor.

{% hint style="warning" %}
🚨 Script execution does not equal security sandboxing. The current interface cannot guarantee that any script will stop within a specified time; for untrusted scripts, independent permission, process and resource isolation designs are required. Don't rely on exception catching to prevent infinite loops or excessive resource usage.
{% endhint %}

## Verification and Error Location

Save masked input and expected results for the rules actually used by the application, covering missing fields, null values, illegal syntax, division by zero, type mismatches, and repeated executions. diagnostics logs rule IDs, versions, and error locations to avoid logging complete variables or scripts containing secrets.

Template generation is another type of task: it renders data into complete text or files. Workbook templates should use [spreadsheet archiving and template services](documents.md), you cannot assume that all template formats are integrated just because Scriban is installed.

Source references: [Lua](https://github.com/Zongsoft/framework/tree/main/externals/lua), [Python](https://github.com/Zongsoft/framework/tree/main/externals/python), [Scriban](https://github.com/Zongsoft/framework/tree/main/externals/scriban).

## Continue Reading by Project

[Lua](projects/lua.md) · [Python](projects/python.md) · [Scriban](projects/scriban.md)
