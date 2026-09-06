---
description: "Use the Windows regex tester to understand matching, grouping, and repeat capture, and keep options consistent with your app."
icon: magnifying-glass
---

# Regular Expression Tool

`Zongsoft.Tools.Regular` is a .NET regular matching tester based on WinForms. The current project target is `net10.0-windows`. It displays matching, grouping and capturing levels, and is suitable for debugging extraction rules; the current main interface logic does not have an independent replacement execution process, and it should not be regarded as a complete replacement editor.

## Prepare and Run

Build the project from the tools repository root directory:

{% code title="BuildRegular.ps1" %}
```powershell
dotnet build ./regular/src/Zongsoft.Tools.Regular.csproj
```
{% endcode %}

Then run the Windows executable file from the corresponding output directory. This tool is not a plugin host and does not require `.plugin` or `dotnet deploy`; the program files and required desktop runtime are determined by the own project.

## Read Matching Results

Put the test text in the input area, fill in the regular expression in the expression area, and observe the result tree after matching. A match represents an overall hit, a group represents a named or numbered subpart of an expression, and a capture represents the results of a repeated grouping in a match.

Discussions There is no independent regular debugging example. You can use Core's TextRegular.Web.Email to define observation named groups. Here's an excerpt of the actual rules; when copied into the tool's expression box, only the contents of the C# verbatim string are taken, without @, quotes, or field declarations.

Source: [framework/Zongsoft.Core/src/Text/TextRegular.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Text/TextRegular.cs#L115) (excerpt; see source for context).

{% code title="TextRegular.cs" %}
```csharp
public static readonly TextRegular Email = new(@"^\s*(?<value>[A-Za-z0-9]([-_\.]?[A-Za-z0-9]+)*@([A-Za-z0-9]+([-_]?[A-Za-z0-9]+)*)(\.[A-Za-z0-9]+([-_]?[A-Za-z0-9]+)*)*\.[A-Za-z]+)\s*$");
```
{% endcode %}

The value group of this rule extracts the mailbox body, and the surrounding whitespace does not belong to the value. Use the application input to be checked to observe the overall matching and grouping results; it is the framework's existing email format rules and does not represent a complete implementation of all legal email syntax. When a result is selected, the hit location is compared against the index and length, not just the displayed text.

## Option Impact

IgnoreCase, IgnorePatternWhitespace, ExplicitCapture are currently enabled by default. They affect case, expression whitespace interpretation, and capture behavior for unnamed groups respectively. The interface also provides Multiline and Singleline; both affect line anchor points and point number matching respectively, and are not the same "multiline mode".

See [`System.Text.RegularExpressions.RegexOptions`](https://learn.microsoft.com/en-us/dotnet/api/system.text.regularexpressions.regexoptions) for the complete enumeration. When moving rules back to your application, you should keep the same options and escaping methods; C# strings, JSON, and regex are themselves different escaping layers.

## Validation Boundaries

At least prepare for normal input, no match, multiple matches, named groups, duplicate captures, zero-length results, and illegal expressions. Successful interactive small sample matching does not demonstrate large input performance, and applications should still set matching timeouts based on input size and avoid high traceback modes.

Open and save operations should only be performed on your own test files and avoid using sensitive real text as a default sample. Source references: [Regular](https://github.com/Zongsoft/tools/tree/main/regular).
