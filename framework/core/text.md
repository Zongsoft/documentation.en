---
description: "Zongsoft.Text text template interface, regular validator and common text matching usage."
icon: font
---

# Zongsoft.Text

`Zongsoft.Text` provides lightweight text template abstraction and regular validator to separate "how text is generated, formatted, and verified" from the specific business code. It is suitable for basic scenarios such as processing command parameters, configuration values, SMS template parameters, URI judgment, email and common Chinese text format verification.

The current core types are concentrated in two directions: one is `ITemplate` / `ITemplateFormatter`, which is used to define templates and template data formatting extension points; the other is `ITextRegular` / `TextRegular`, which is used to encapsulate reusable regular matchers and extract the normalized results when the match is successful.

## Main Responsibilities

* Define the text template interface so that business modules can evaluate data into text according to the template name.
* Define a template formatter interface so that external service template parameters can be converted into the structure required by the provider before being sent.
* Provides `TextRegular` regular encapsulation to uniformly handle matching, failure tolerance and result extraction.
* Provides commonly used validators such as email, URL, HTTP/FTP URL, Chinese mobile phone number, landline phone number, ID number, postal code, etc.
* Allows commands, configurations, messages, resource output, and external communication to reuse the same set of lightweight text processing conventions.

## Key Types

| Type | Description |
| --- | --- |
| `ITemplate` | Text template interface, providing template name and `Evaluate(...)` evaluation entry. |
| `ITemplateFormatter` | Template data formatter interface returns formatted template data based on template name, data and additional parameters. |
| `ITextRegular` | Text matching interface, inherits the matchable object convention, and supports outputting matching results. |
| `TextRegular` | Regular text matcher, providing `Match(...)` and `IsMatch(...)`. |
| `TextRegular.Web` | A commonly used validator on the Web, currently providing an email validator. |
| `TextRegular.Uri` | URL validators including any protocol, HTTP, FTP and create-by-protocol validators. |
| `TextRegular.Chinese` | China mobile number, landline number, ID number and postal code validator. |

## Regular Validator

`TextRegular` Constructs a reusable text validator using a regular expression. It internally enables compilation, ignores case, ignores pattern whitespace and explicit capture, and sets a match timeout. Calling `Match(...)` will only return whether there is a match, while calling `IsMatch(...)` will return the result text if the match is successful.

Source: [framework/Zongsoft.Core/src/Text/TextRegular.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Text/TextRegular.cs#L115) (excerpt; see source for context).

{% code title="TextRegular.cs" %}
```csharp
public static readonly TextRegular Email = new(@"^\s*(?<value>[A-Za-z0-9]([-_\.]?[A-Za-z0-9]+)*@([A-Za-z0-9]+([-_]?[A-Za-z0-9]+)*)(\.[A-Za-z0-9]+([-_]?[A-Za-z0-9]+)*)*\.[A-Za-z]+)\s*$");
```
{% endcode %}

There is an important convention for result extraction of `IsMatch(...)`: if the regular expression contains a capture group named `value`, all captured values of the group will be concatenated into the result; when using result extraction, the value group should be explicitly defined; the current implementation's handling of the lack of this group cannot be used as a guarantee to return a complete matching text. This convention is suitable for removing non-core content such as spaces, delimiters, and country codes in user input, and returns a standardized value that is more suitable for saving or subsequent processing.

Source: [framework/Zongsoft.Core/src/Text/TextRegular.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Text/TextRegular.cs#L183) (excerpt; see source for context).

{% code title="TextRegular.cs" %}
```csharp
public static readonly TextRegular Cellphone = new(@"^\s*((\+|00)86\s*[-\.]?)?\s*(?<value>1\d{2})(?<separator>(\s*)|(-?))(?<value>\d{4})(?<separator>(\s*)|(-?))(?<value>\d{4})\s*$");
```
{% endcode %}

If you only need to determine whether a certain string is a URL, you can use `Match(...)`:

Source: [framework/Zongsoft.Core/src/IO/FileSystem.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/IO/FileSystem.cs#L78) (excerpt; see source for context).

{% code title="FileSystem.cs" %}
```csharp
public static string GetUrl(string virtualPath)
{
	if(string.IsNullOrWhiteSpace(virtualPath))
		return virtualPath;

	//如果传入的虚拟路径参数是一个URI格式，则直接返回它作为结果
	if(Zongsoft.Text.TextRegular.Uri.Url.Match(virtualPath))
		return virtualPath;

	return GetFileSystem(virtualPath, false, out Path path)?.GetUrl(path);
}
```
{% endcode %}

The file system tool in the framework uses `TextRegular.Uri.Url.Match(...)` to determine whether the incoming virtual path is already a URI; if it is a URI, it directly returns the original address instead of continuing to parse according to the local file system path.

## Preset Validator

| Validator | Purpose |
| --- | --- |
| `TextRegular.Web.Email` | Verify and extract email addresses. |
| `TextRegular.Uri.Url` | Verify any protocol URL. |
| `TextRegular.Uri.Http` | Verify the `http` or `https` URL. |
| `TextRegular.Uri.Ftp` | Verify the `ftp` or `ftps` URL. |
| `TextRegular.Uri.GetRegular(scheme)` | Creates and caches a URL validator for the specified protocol. |
| `TextRegular.Chinese.Cellphone` | Verify and normalize Chinese mobile phone numbers. |
| `TextRegular.Chinese.Telephone` | Validation and standardization of China's landlines. |
| `TextRegular.Chinese.IdentityNo` | Verify and normalize Chinese ID numbers. |
| `TextRegular.Chinese.PostalCode` | Verify and extract Chinese postal codes. |

These validators are more suitable for input format confirmation and basic normalization, and should not be regarded as complete business verification. For example, whether the ID number actually exists, whether the mobile phone number is real-name, and whether the email domain name can be delivered, all need to be confirmed by more specific business processes or external services.

## Custom Validator

Below is the actual construction entry of the framework. Business modules can create TextRegular with their own regularity. As long as `(?<value>...)` is used in the regular expression to capture the really needed fragments, the caller can get the normalized result.

Source: [framework/Zongsoft.Core/src/Text/TextRegular.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Text/TextRegular.cs#L43) (excerpt; see source for context).

{% code title="TextRegular.cs" %}
```csharp
public TextRegular(string pattern)
{
	if(string.IsNullOrWhiteSpace(pattern))
		throw new ArgumentNullException(nameof(pattern));

	_regex = new Regex(pattern, RegexOptions.Compiled | RegexOptions.ExplicitCapture | RegexOptions.IgnorePatternWhitespace | RegexOptions.IgnoreCase, TimeSpan.FromSeconds(3));
}
```
{% endcode %}

{% hint style="info" %}
If the matching of `TextRegular` fails, `false` will be returned, including empty text, regular matching failure and matching exception. It is suitable for tolerant input judgment; if the caller needs to expose the specific error cause, a more clear prompt should be added to the outer layer.
{% endhint %}

## Template Interface

Currently, there is no ready-made ITemplate evaluation implementation in the repository. The real interface contract is retained below, and unimplemented notification classes are not used as runnable examples. ITemplate represents an evaluable text template. It only specifies the template name and evaluation entry, but does not limit the template syntax, so the implementation can come from string templates, script templates, resource files, external service templates or custom template engines.

Source: [framework/Zongsoft.Core/src/Text/ITemplate.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Text/ITemplate.cs#L37) (excerpt; see source for context).

{% code title="ITemplate.cs" %}
```csharp
public interface ITemplate
{
	/// <summary>获取模板名称。</summary>
	string Name { get; }

	/// <summary>应用模板。</summary>
	/// <param name="data">待应用的模板数据。</param>
	/// <param name="arguments">附加参数集。</param>
	/// <returns>返回应用后的文本。</returns>
	string Evaluate(object data, params object[] arguments);
}
```
{% endcode %}

`ITemplateFormatter` prefers "format before sending". It receives the template name, original data and additional parameters, and returns the formatted data object. This return value can then be serialized into JSON, a query string, or a parameter structure required by the provider API.

Source: [framework/externals/aliyun/src/Telecom/Phone.cs](https://github.com/Zongsoft/framework/blob/main/externals/aliyun/src/Telecom/Phone.cs#L126) (excerpt; see source for context).

{% code title="Phone.cs" %}
```csharp
//尝试进行模板数据格式化
if(!string.IsNullOrEmpty(template.Formatter) && this.ServiceProvider.Resolve(template.Formatter) is ITemplateFormatter formatter)
{
	argument.Parameter = formatter.Format(template.Name, argument.Parameter, argument.Extra);
```
{% endcode %}

In external communication scenarios, the template configuration can specify a formatter name. Before sending, the service parses `ITemplateFormatter` by name, and then hands the template parameters to the formatter for processing. For example, the Alibaba Cloud voice and SMS sending process will call the formatter to convert business parameters into supplier template parameters before submitting the request.

## Typical Use Cases

{% tabs %}
{% tab title="Input validation" %}
Commands, configurations and web forms can use `TextRegular` for first-level format judgment. For example, when entering a mobile phone number as a command parameter, first use `TextRegular.Chinese.Cellphone.IsMatch(...)` to extract the standardized number, and then enter the authentication, sending or business query process.
{% endtab %}

{% tab title="Path judgment" %}
When processing file systems, resource references and download addresses, you can first use `TextRegular.Uri.Url.Match(...)` to determine whether the text is already a URL. If it is a URL, it will be processed according to the remote address; otherwise, it will continue to be resolved according to the local path or virtual path.
{% endtab %}

{% tab title="template parameters" %}
External services such as SMS, voice, notifications, and message queues often require template parameters to conform to a specific format. You can put the format conversion in `ITemplateFormatter`, so that the business code only submits the original data, and the formatter is responsible for adapting to different templates or suppliers.
{% endtab %}
{% endtabs %}

## Usage Suggestions

`TextRegular` is suitable for lightweight format judgment and result extraction. For scenarios that require strong security guarantees, such as identity verification, payment parameters, permission expressions or external callback signatures, business verification and security verification should be continued after regular verification.

Template interfaces are suitable for framework-level extension points, rather than specifying a certain template syntax. The project can choose string interpolation, resource files, script templates or third-party template engines according to actual needs, as long as `ITemplate` or `ITemplateFormatter` is finally achieved.

If the regular expression needs to be reused, it is recommended to define it as a static read-only validator to avoid repeated construction on the hot path; if the protocol or format is determined by the configuration, you can use a cached entry such as `TextRegular.Uri.GetRegular(...)`.

## Related Resources

* [Text source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Text)
* [FileSystem.cs URL Determination Use Case](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/IO/FileSystem.cs)
* [Alibaba Cloud Telecom Template Format Use Case](https://github.com/Zongsoft/framework/blob/main/externals/aliyun/src/Telecom/Phone.cs)
