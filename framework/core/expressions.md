---
description: "Zongsoft.Expressions' expression evaluation abstraction, lexer, token model, and tokenizer extensions."
icon: code
---

# Zongsoft.Expressions

`Zongsoft.Expressions` provides expression evaluation abstraction and lexical analysis infrastructure. It first splits an expression text into constants, identifiers, keywords and symbol tokens, and then gives it to a specific expression evaluator or an upper-level resolver to interpret the semantics.

This namespace is more like an "expression base layer" than a complete scripting engine with a fixed syntax. Conditional expressions, configuration expressions, command parameter expressions and some data query expressions in the framework can all define their own keywords, operators and evaluation rules on this basic layer.

{% hint style="info" %}
`Lexer` is only responsible for lexical segmentation and is not responsible for determining how `1 + 2` should be calculated, nor is it responsible for handling operator priority. Arithmetic, conditional, member access, or database expression semantics need to be done by the caller or a concrete `IExpressionEvaluator` implementation.
{% endhint %}

## Main Responsibilities

* Define the expression evaluator interface, base classes, and run options.
* Provides reusable lexical analyzer `Lexer` and scanner `TokenScanner`.
* Split the expression text into `TokenType.Constant`, `TokenType.Identifier`, `TokenType.Symbol` and `TokenType.Keyword`.
* Built-in null, boolean, number, string, identifier, and symbol tokenizers.
* Supports callers to append or adjust tokenizers to recognize domain keywords and custom symbols.
* Provides underlying capabilities for plugin configuration, command interpretation, conditional parsing, data expressions and lightweight script evaluation.

## Core Types

| Type | function | Common usage |
| --- | --- | --- |
| [`IExpressionEvaluator`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/IExpressionEvaluator.cs) | Expression evaluator contract. | Returns evaluation results by expression text, options, and variable collections. |
| [`ExpressionEvaluatorBase`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/ExpressionEvaluatorBase.cs) | Evaluator base class. | Unified name matching, default options, global variables and release logic. |
| [`ExpressionEvaluatorOptions`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/ExpressionEvaluatorOptions.cs) | Evaluation run options. | Specify input, output, error output, and extended attributes. |
| [`Lexer`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/Lexer.cs) | Lexical analysis entry. | Create `TokenScanner` from a string or stream. |
| [`TokenScanner`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/TokenScanner.cs) | token scanner. | Scan tokens individually, or enumerate all tokens as a collection. |
| [`ITokenizer`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/ITokenizer.cs) | Tokenizer interface. | Provide recognition logic for new literals, keywords, or domain symbols. |
| [`Token`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/Token.cs) | token object. | Carry token type and value. |
| [`SymbolToken`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/SymbolToken.cs) | symbol token. | Represents `+`, `==`, `&&`, `??`, brackets and other built-in symbols. |
| [`SyntaxException`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Expressions/SyntaxException.cs) | Syntax exception. | Thrown when illegal characters or incomplete literals are encountered during word segmentation. |

## Lexical Analysis Process

`Lexer` maintains a tokenizer list. `TokenScanner.Scan()` will first skip whitespace characters every time it scans, and then try each tokenizer in list order; the first tokenizer that returns a valid token determines the recognition result of the current position. If none of the tokenizers recognize the current character and the input has not ended, `SyntaxException` is thrown.

The default tokenizer order of `Lexer` is as follows:

| order | tokenizer | Identify content |
| --- | --- | --- |
| 1 | `NullTokenizer` | `null`, ignoring case. |
| 2 | `NumberTokenizer` | Integers, decimals, and numeric suffixes `L`, `F`, `D`, `M`. |
| 3 | `StringTokenizer` | Single or double quoted string. |
| 4 | `BooleanTokenizer` | `true`, `false`, case is ignored. |
| 5 | `IdentifierTokenizer` | An identifier that begins with a letter or `_` and is followed by letters, numbers, or `_`. |
| 6 | `SymbolTokenizer` | Built-in symbol token. |

Source: [framework/Zongsoft.Core/test/Expressions/LexerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Expressions/LexerTest.cs#L15) (excerpt; see source for context).

{% code title="LexerTest.cs" %}
```csharp
const string EXPRESSION = @"1+2f	_abc123'text\'suffix'	-30L*4.5 / 5.5m (true || FALSE?yes:no)null??nothing";

var scanner = Lexer.Instance.GetScanner(EXPRESSION);
Assert.NotNull(scanner);

var token = scanner.Scan();
Assert.NotNull(token);
Assert.Equal(TokenType.Constant, token.Type);
Assert.IsType<int>(token.Value);
Assert.Equal(1, (int)token.Value);

token = scanner.Scan();
Assert.NotNull(token);
Assert.Equal(SymbolToken.Plus, token);

token = scanner.Scan();
Assert.NotNull(token);
Assert.Equal(TokenType.Constant, token.Type);
Assert.IsType<float>(token.Value);
Assert.Equal(2.0f, (float)token.Value);
```
{% endcode %}

Discussions does not construct its own lexer; the Core test above asserts the integer constant 1, the plus sign, and the single-precision constant 2, in that order. Full testing continues to cover identifiers, strings, minus signs, and other operators. Subsequently, whether it is a normal condition, a data query condition, or a command parameter condition depends on how the caller interprets these tokens.

## Token Rules

Built-in tokens are roughly divided into four categories:

| token type | Example | Description |
| --- | --- | --- |
| `Constant` | `null`、`true`、`30L`、`4.5`、`5.5m`、`'text'` | Literal constant, `Token.Value` will save the parsed .NET value. |
| `Identifier` | `Name`、`_abc123`、`Field1` | Variable name, field name, parameter name, method name, or caller-defined identifier. |
| `Symbol` | `+`、`-`、`*`、`/`、`==`、`&&`、`??`、`(`、`)`、`[`、`]` | Built-in operator or delimiter. |
| `Keyword` | `in`、`between` | Will only appear if `KeywordTokenizer` is explicitly added by the caller. |

Numeric literals start with numbers: without a decimal point, the default parsing is `int`, and with a decimal point, the default parsing is `double`; the suffix `L` represents `long`, `F` represents `float`, `D` represents `double`, and `M` represents `decimal`. Numbers cannot end with a decimal point or contain multiple decimal points.

Strings can be wrapped in single quotes or double quotes and support common escapes: `\\`, `\'`, `\"`, `\s`, `\t`, `\n`, `\r`. String literals cannot span lines, and `SyntaxException` will be thrown if the closing quote is missing.

Built-in symbols cover common operations and separation scenarios:

| Category | symbol |
| --- | --- |
| Arithmetic | `+`、`-`、`*`、`/`、`%` |
| logic and comparison | `!`、`&&`、`||`、`==`、`!=`、`<`、`<=`、`>`、`>=` |
| Null values and conditions | `??`、`?`、`:` |
| access and separation | `.`、`,`、`;`、`|` |
| brackets | `(`、`)`、`[`、`]`、`{`、`}` |

{% hint style="warning" %}
The numeric minus sign is not part of a numeric literal. The expression `-30L` is scanned as the symbol `-` and the constant `30L`, and whether it is interpreted as a negative number is determined by subsequent syntax parsing or evaluation stages.
{% endhint %}

## Keywords and Custom Tokenizers

The default lexicalizer does not enable business keywords, so texts such as `in` and `between` will first be recognized as identifiers. When you need the domain keyword, you can create a new `Lexer` instance and put `KeywordTokenizer` before `IdentifierTokenizer`.

Source: [framework/Zongsoft.Core/test/Expressions/LexerTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Expressions/LexerTest.cs#L139) (excerpt; see source for context).

{% code title="LexerTest.cs" %}
```csharp
const string EXPRESSION = @"Field1 == 100 && Field2<1.23f && Field3 >=10.5m && (PI between ""3.1415926~3.1415927"" || Number IN [10,20,30] )";

var lexer = new Lexer();
lexer.Tokenizers.Insert(0, new KeywordTokenizer(true, "in", "Between"));

var scanner = lexer.GetScanner(EXPRESSION);
Assert.NotNull(scanner);

var token = scanner.Scan();
Assert.NotNull(token);
Assert.Equal(TokenType.Identifier, token.Type);
Assert.Equal("Field1", token.Value);
```
{% endcode %}

`KeywordTokenizer(true, ...)` means ignoring case, so `IN`, `in` and `In` can all be recognized as the same keyword token. After keyword recognition, only tokens are still generated, and collection inclusion, range matching, or SQL translation are not automatically implemented; these semantics should be handled by subsequent resolvers or evaluators.

Custom tokenizers are suitable for these scenarios:

* Domain keywords need to be identified, such as `between`, `like`, `contains`.
* You need to convert certain types of literals directly into specific values, such as dates, time periods, enumeration names, or variable placeholders.
* It is necessary to limit the available symbols or to add special symbols for a certain DSL.

One principle should be followed when implementing a custom tokenizer: only return the token when the current position can be completely recognized; return a failure result when it cannot be recognized, and restore the reader offset to the position before the attempt. In this way, subsequent tokenizers can continue to judge the same text.

## Expression Evaluator

`IExpressionEvaluator` defines the common shape for expression evaluation. The caller passes in an expression text, a collection of optional variables, and optional run options, and the evaluator returns an object result.

Source: [framework/externals/python/test/PythonExpressionEvaluatorTest.cs](https://github.com/Zongsoft/framework/blob/main/externals/python/test/PythonExpressionEvaluatorTest.cs#L14) (excerpt; see source for context).

{% code title="PythonExpressionEvaluatorTest.cs" %}
```csharp
public void TestEvaluate1()
{
	var evaluator = new PythonExpressionEvaluator();
	var variables = new Dictionary<string, object>();

	var result = evaluator.Evaluate("1+2", null);
	Assert.NotNull(result);
	Assert.Equal(3, Zongsoft.Common.Convert.ConvertValue<int>(result));

	variables["subtract"] = (Delegate)Subtract;
	result = evaluator.Evaluate("subtract(100, 20)", variables);
	Assert.NotNull(result);
	Assert.Equal(80, Zongsoft.Common.Convert.ConvertValue<int>(result));

	evaluator.Evaluate("a=1;b=2;result=a+b;", variables);
	Assert.NotEmpty(variables);
	Assert.True(variables.TryGetValue("result", out result));
	Assert.Equal(3, Zongsoft.Common.Convert.ConvertValue<int>(result));
}
```
{% endcode %}

The above is the actual test of the Python plugin. Subtract is a static method in the same test class. The variable dictionary also receives the result written back by the script. Evaluators are usually implemented by concrete modules with their own syntax and semantics. `ExpressionEvaluatorBase` provides these general capabilities:

* `Name` represents the evaluator name.
* Implements the service matching interface to find evaluators by name ignoring case.
* `Global` holds evaluator-level global variables.
* `Options` saves default input, output, error output, and extended attributes.
* Global variables will be cleaned up when `Dispose()` is released.

`ExpressionEvaluatorOptions` uses `Console.In`, `Console.Out` and `Console.Error` by default, and the input and output can also be replaced by static methods or extension methods.

Source: [framework/externals/python/test/PythonExpressionEvaluatorTest.cs](https://github.com/Zongsoft/framework/blob/main/externals/python/test/PythonExpressionEvaluatorTest.cs#L98) (excerpt; see source for context).

{% code title="PythonExpressionEvaluatorTest.cs" %}
```csharp
public void TestEvaluateOutputDoesNotLeakAcrossCallsOrEvaluators()
{
	const string PRINT_MESSAGE = "First evaluator output";
	using var evaluator = new PythonExpressionEvaluator();

	using(var output = new StringWriter())
	{
		evaluator.Evaluate($"print('{PRINT_MESSAGE}')", ExpressionEvaluatorOptions.Out(output));
		Assert.Equal(PRINT_MESSAGE, output.ToString());
	}

	var result = evaluator.Evaluate("print('Subsequent call output'); result=41");
	Assert.Equal(41, Zongsoft.Common.Convert.ConvertValue<int>(result));

	using var subsequentEvaluator = new PythonExpressionEvaluator();
	result = subsequentEvaluator.Evaluate("print('Second evaluator output'); result=42");

	Assert.Equal(42, Zongsoft.Common.Convert.ConvertValue<int>(result));
}
```
{% endcode %}

The output isolation test releases the first round of output objects before executing the next call and another evaluator call to verify that the output stream is restored. It does not prove complete isolation of the shared runtime when multiple calls are made concurrently; see the Scripting topic for more boundaries.

If multiple `IExpressionEvaluator` implementations are registered in the application, they can be selected by name in combination with the service model. For example, `ExpressionEvaluatorBase` already supports ignoring case matching, so callers can find a named evaluator through the service discovery mechanism.

Existing language implementations are registered under the names Lua, Python, and Scriban. See [Scripts and Expressions](../externals/scripting.md) for selection, deployment and syntax differences. Discussions The JavaScript evaluator is not registered, and a successful name lookup cannot be considered to be available for any language.

## Usage Suggestions

Use `Lexer` in boundary layers that require understanding the text structure first: such as parsing configuration values, command parameters, query conditions, filter rules, or lightweight expressions. For text with a fixed format and simple semantics, ordinary string processing may be more straightforward; for scenarios that require combinations of variables, brackets, operators, and literals, lexical analysis can make subsequent parsing clearer.

When adding keywords to domain syntax, try to use new `Lexer` instances instead of directly modifying global `Lexer.Instance`. Global instances are suitable as default universal tokenizers; domain grammars should generally have their own tokenizer order to avoid affecting other modules.

Expressions from user input should be syntactically restricted and whitelisted before evaluation. The lexicalizer can tell you which tokens the text has been cut into, but it will not automatically determine whether an identifier, method name, field name or operator is allowed to be exposed to external users.

{% hint style="warning" %}
Do not concatenate tokenization results directly into SQL, scripts, or command text. When a database query or command needs to be generated, a clear syntax tree or operation model should be constructed during the parsing phase, and then the corresponding driver handles parameterization and escaping.
{% endhint %}

## Sub Namespace

| namespace | Description |
| --- | --- |
| `Zongsoft.Expressions.Tokenization` | Built-in tokenizer, literal tokenizer base class and keyword, string, number, boolean, null, identifier, symbol tokenizer. |

## Related Resources

* [Expressions source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Expressions)
* [Tokenization source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Expressions/Tokenization)
* [Lexer test](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Expressions/LexerTest.cs)
* [Service Discovery](services.md#search-by-parameters)
* [data engine conditions and operand](../data/conditions-and-operands.md)
