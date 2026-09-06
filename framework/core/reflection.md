---
description: "Zongsoft.Reflection's reflective access, dynamic accessors and member expression evaluation capabilities."
icon: magnifying-glass
---

# Zongsoft.Reflection

`Zongsoft.Reflection` provides a set of lightweight reflection tools for runtime object access. It encapsulates common field, attribute, indexer and member path access into a reusable dynamic accessor model, which is suitable for configuration binding, data mapping, template evaluation, command parameter parsing, plugin expansion and other "member names are determined at runtime" scenarios.

Reflective access itself should not replace ordinary strongly typed code: when members have been determined at compile time, direct calls are still the clearest way to write; when field names, property names, or access paths come from configuration, scripts, user input, or metadata, then use the capabilities provided by this namespace.

## Main Responsibilities

* Unified reading and writing of fields, properties, and indexers via `Reflector`.
* Use [`System.Reflection.Emit`](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.emit) to dynamically compile and cache getter and setter accessors for System.Reflection.FieldInfo and System.Reflection.PropertyInfo to reduce the cost of repeated reflection calls.
* Supports reading or writing object members by string member name, name lookups ignore case, and access to default members.
* Parse member expressions and convert paths such as `Address.City`, `Items[0].Name`, and `Get("key")` into expression node chains.
* Evaluate an expression or set the final member via `MemberExpressionEvaluator`.
* Provides low-level member access capabilities for modules such as model mapping, data binding, configuration binding, serialization, and report field selection.

## Core Types

| Type | function | Common usage |
| --- | --- | --- |
| `Reflector` | High-rise reflective access entrance. | Read and write fields and properties by System.Reflection.MemberInfo or string member name. |
| `FieldInfoExtension` | Field accessor extension. | Get cached getters and setters from System.Reflection.FieldInfo. |
| `PropertyInfoExtension` | Property accessor extension. | Gets cached getters, setters from System.Reflection.PropertyInfo, and supports indexer parameters. |
| `MemberExpression` | Member expression base class and parsing entry. | Parse or manually construct member path expressions. |
| `IMemberExpression` | Expression node interface. | Represents a node in the expression chain and is concatenated through `Previous` and `Next`. |
| `IdentifierExpression` | Identifier node. | Represents a common field or attribute name. |
| `IndexerExpression` | Indexer node. | Represents `[]` access and carries index parameters. |
| `MethodExpression` | method node. | Represents method calls and parameter lists. |
| `ConstantExpression` | Constant node. | Represents parameter values such as strings, numbers, and null values. |
| `MemberExpressionEvaluator` | Expression evaluator. | Read the path value to the target object, or write the path end member. |

## Member Access

Discussions does not call Reflector directly; it uses these mechanisms indirectly through mapping and configuration. This page reads API shapes from [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core)'s existing tests, where MyValue and ClassEntity are test fixture types.

`Reflector` is the most commonly used entry. It can either receive the already parsed System.Reflection.MemberInfo, or it can directly find the public fields or properties of the object by member name.

Source: [framework/Zongsoft.Core/test/Reflection/ReflectorTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Reflection/ReflectorTest.cs#L14) (excerpt; see source for context).

{% code title="ReflectorTest.cs" %}
```csharp
var target = new MyValue(5);

Assert.Equal(100, (int)Reflector.GetValue(ref target, nameof(MyValue.IntegerField)));
Assert.Equal(123.05m, (decimal)Reflector.GetValue(ref target, nameof(MyValue.DecimalProperty)));
Assert.Equal("StringField", (string)Reflector.GetValue(ref target, nameof(MyValue.StringField)));
Assert.Equal("StringProperty", (string)Reflector.GetValue(ref target, nameof(MyValue.StringProperty)));
Assert.Null(Reflector.GetValue(ref target, nameof(MyValue.NullableField)));
Assert.Null(Reflector.GetValue(ref target, nameof(MyValue.NullableProperty)));

Assert.Equal("#0", Reflector.GetValue(ref target, "Item", 0));
Assert.Equal("#1", Reflector.GetValue(ref target, "Item", 1));
Assert.Equal("#2", Reflector.GetValue(ref target, "Item", 2));
Assert.Equal("#3", Reflector.GetValue(ref target, "Item", 3));
Assert.Equal("#4", Reflector.GetValue(ref target, "Item", 4));

Reflector.SetValue(ref target, nameof(MyValue.IntegerField), 200);
Reflector.SetValue(ref target, nameof(MyValue.DecimalProperty), 456.78);
Reflector.SetValue(ref target, nameof(MyValue.StringField), "NewStringField");
Reflector.SetValue(ref target, nameof(MyValue.StringProperty), "NewStringProperty");

Assert.Equal(200, (int)Reflector.GetValue(ref target, nameof(MyValue.IntegerField)));
Assert.Equal(456.78m, (decimal)Reflector.GetValue(ref target, nameof(MyValue.DecimalProperty)));
Assert.Equal("NewStringField", (string)Reflector.GetValue(ref target, nameof(MyValue.StringField)));
Assert.Equal("NewStringProperty", (string)Reflector.GetValue(ref target, nameof(MyValue.StringProperty)));
```
{% endcode %}

`GetValue` and `SetValue` will throw exceptions when the target is empty, the member does not exist, the attribute is unreadable, etc.; writing will only take effect on writable attributes and non-read-only fields, and `false` will be returned when a read-only field or unwritable attribute is encountered. `TryGetValue` and `TrySetValue` are more suitable for handling external input, optional fields or scenarios that are compatible with old models.

If the caller already holds System.Reflection.FieldInfo, System.Reflection.PropertyInfo, or System.Reflection.MemberInfo, the extension method can be used directly. GetGetter/GetSetter will reuse the cache. The following test explicitly calls GenerateGetter/GenerateSetter to verify the dynamic accessor generation and read and write results; do not generate repeatedly in a hot loop.

Source: [framework/Zongsoft.Core/test/Reflection/PropertyInfoTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Reflection/PropertyInfoTest.cs#L13) (excerpt; see source for context).

{% code title="PropertyInfoTest.cs" %}
```csharp
var entity = new ClassEntity()
{
	Id = 100,
	Name = "Popeye",
};

//init the target variable
var target = (object)entity;

//test the instance property(get and set)
var property = typeof(ClassEntity).GetProperty(nameof(ClassEntity.Id));
var getter = property.GenerateGetter();
var setter = property.GenerateSetter();

Assert.NotNull(getter);
Assert.NotNull(setter);

var value = getter(ref target);
Assert.Equal(100, value);

setter(ref target, 200);
value = getter(ref target);
Assert.Equal(200, value);
```
{% endcode %}

## Dynamic Accessors and Performance

Field and property extension methods will internally generate dynamic methods based on [`System.Reflection.Emit`](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.emit), and then compile the dynamic methods into getters or setter delegates. When accessing a member for the first time, you need to complete dynamic method generation, IL emission and delegate creation; after the generation is completed, the accessor will be cached in the cache item corresponding to the member information. When subsequently reading or writing the same member, the delegate will be called directly to avoid repeatedly following the `GetValue` and `SetValue` paths of traditional reflection.

This design is suitable for high-frequency, repeated runtime member access: for example, data mapping continuously populates the model, configuration binding repeatedly writes properties, and templates or reports read objects by field names. It does not turn a one-time member lookup into a strongly typed call, nor does it eliminate the cost of looking up members by name, parsing expressions, and parameter conversion; therefore, in loop or batch processing scenarios, try to reuse the obtained System.Reflection.FieldInfo, System.Reflection.PropertyInfo, or parsed expression objects.

The BenchmarkDotNet benchmark for attribute reading and writing is provided in the `Zongsoft.Core/benchmark/Reflection` directory of the source code repository. The test uses ordinary reflection `PropertyInfo.GetValue` and `PropertyInfo.SetValue` as the baseline, and compares the paths of `Reflector` package access and direct reuse of `GetGetter<T>` and `GetSetter<T>` delegates respectively; the actual benefits depend on the runtime, member shape and number of calls, and should be measured in the target environment. Specific multiples cannot be inferred simply by the existence of a benchmark project.

{% hint style="info" %}
The benefit of dynamic accessors comes from "generate once, call many times". If a member is accessed only once, generating the accessor itself also incurs a small overhead; if the member will be accessed repeatedly, caching the delegate is usually more stable than using reflection call each time.
{% endhint %}

{% hint style="info" %}
String member name lookup defaults to public instance members and public static members, and ignores case. If the member name is empty, an attempt is made to access the default member of the target type, often used for indexer access.
{% endhint %}

## Indexer Access

Property accessors support index parameters, so the indexer can be accessed either through a concrete System.Reflection.PropertyInfo call or through a default member.

ReflectorTest.TestValueType has read items 0 to 4 of the Item indexer in the above snippet. The indexer and field definition of MyValue are saved in the same test file, and the fixture should be retained when reused; index out-of-bounds and parameter type errors are still determined by the target member.

This type of writing is suitable for dealing with collections, dictionaries, dynamic models, or objects with default members. For strongly typed code paths such as dictionaries and collections, it is often more intuitive to access the collection directly if the keys and indexes are known at compile time.

## Member Expression

The `Zongsoft.Reflection.Expressions` subnamespace provides member path resolution and evaluation capabilities. The expression is parsed as a bidirectional chain of nodes, each node representing a member, method, or indexer access.

Source: [framework/Zongsoft.Plugins/src/Configuration/OptionParser.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/Configuration/OptionParser.cs#L40) (excerpt; see source for context).

{% code title="OptionParser.cs" %}
```csharp
public override object Parse(ParserContext context)
{
	if(string.IsNullOrWhiteSpace(context.Text))
		return null;

	var expression = Collections.HierarchicalExpression.Parse(context.Text);

	if(expression != null)
	{
		object target = ApplicationContext.Current.Configuration.GetOption(context.MemberType, expression.Path);

		if(target != null && expression.Accessor != null)
			return Reflection.Expressions.MemberExpressionEvaluator.Default.GetValue(expression.Accessor, target);
		else
			return target;
	}

	return null;
}
```
{% endcode %}

The above is the actual evaluation entry of the plugin option expression: first obtain the object according to the configuration path, and then hand it to the MemberExpressionEvaluator when there is an additional member path. The following spreadsheet is a reference to the syntax form and does not indicate that the Discussions model has these properties.

Common expression forms are as follows:

| expression | meaning |
| --- | --- |
| `Name` | Read the `Name` field or property of the target object. |
| `Address.City` | Read nested members level by level. |
| `Items[0]` | Access the first element of the default indexer. |
| `Items[0].Name` | Get the collection elements first, then read the element members. |
| `Options["theme"]` | Use string constants as indexer parameters. |
| `GetValue("name")` | Call the public method and pass in the string constant parameter. |
| `GetItem(Index).Name` | A method or indexer parameter can also be another member expression. |

Expression parameters support single- or double-quoted strings, numeric constants, and member expressions. Numeric constants start with a number, and can use the `L`, `F`, `M` suffixes to represent long integer, single precision, or decimal numbers; numbers containing a decimal point are interpreted as double precision.

{% hint style="warning" %}
Member expressions are not null propagation expressions. If the intermediate object is `null` during the evaluation process, subsequent member lookup will usually fail; when fault tolerance is required, a default object should be prepared before the call, or the values of some nodes should be taken over through the evaluation callback of `MemberExpressionEvaluator`.
{% endhint %}

## Custom Evaluation

Both the read and write methods of `MemberExpressionEvaluator` accept evaluation callbacks. The callback will be executed after each node resolves the members. The caller can check the current node, target object, member information and parameters, or set the node value in advance to override the default reflection evaluation logic.

Frame [Criteria.Transform](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Data/Criteria.cs) is the actual custom write call point: it parses the query member name into a path, converts the text into a boolean, array, set, or scalar according to the target member type, and returns the final write value through the valueFactory. When processing nested members, intermediate objects are also prepared using the evaluate callback. See [Conditions and Operands](../data/conditions-and-operands.md) for the complete process.

This is suitable for implementing default value completion, permission filtering, external dictionary values, lazy loading, or special mapping of certain path nodes. Callbacks should only handle nodes that really need to be involved; ordinary member access can be left to the default evaluator.

## Usage Suggestions

Prefer limiting reflective access to system boundaries, such as configuration, plugins, mappings, bindings, serialization, report fields, and command parameter handling. If the core business process can be expressed using interfaces, generics or common attribute access, there is no need to introduce member name strings.

For repeated execution paths, it is recommended to parse and cache `IMemberExpression` first, and do not call `MemberExpression.Parse` repeatedly in a loop. Similarly, when you have obtained System.Reflection.FieldInfo or System.Reflection.PropertyInfo, give priority to reusing their dynamic accessors instead of searching by string name each time.

Member paths from user input or external configuration should be whitelisted first. Reflection tools can reduce calling costs, but they will not automatically determine whether a member is suitable for exposure to external use; when exposing field paths, method calls, or indexer access, the caller should define a clear accessible scope.

{% hint style="warning" %}
Member access by name lookup only covers public fields, public properties, and public methods. If access to non-public members is required, the caller should explicitly obtain the corresponding System.Reflection.MemberInfo and verify that this does not break encapsulation or security boundaries.
{% endhint %}

## Sub Namespace

| namespace | Description |
| --- | --- |
| `Zongsoft.Reflection.Expressions` | Member path expression parsing, expression node model, and expression evaluator. |

## Related Resources

* [Reflection source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Reflection)
* [Expressions source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Reflection/Expressions)
* [Property Read Performance Benchmark](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/benchmark/Reflection/PropertyGetterBenchmark.cs)
* [Property writing performance benchmark](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/benchmark/Reflection/PropertySetterBenchmark.cs)
