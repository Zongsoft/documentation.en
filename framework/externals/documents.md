---
description: "Template data assembly is explained with the Discussions user archive provider and user-list.xlsx."
icon: table
---

# Spreadsheet and Template Extensions


Discussions provides the real user list template docs/templates/user-list.xlsx, as well as the UserDataTemplateModelProvider. This case illustrates how the business prepares archived data, and how the spreadsheet engine consumes templates is determined by the implementation of the host deployment.

## Template Model Provider

Source: [src/Features/Archiving/UserDataTemplateModelProvider.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Features/Archiving/UserDataTemplateModelProvider.cs#L41) (excerpt; see source for context).

{% code title="UserDataTemplateModelProvider.cs" %}
```csharp
[Service(typeof(IDataTemplateModelProvider))]
public class UserDataTemplateModelProvider : DataTemplateModelProviderBase
{
	#region 构造函数
	public UserDataTemplateModelProvider(IServiceProvider services) : base("User-List", services) { }
```
{% endcode %}

The provider name is User-List. It participates in template model matching through service registration, and cannot just copy xlsx without missing assembly scanning and archiving engine deployment.

## From Filters to Users

Source: [src/Features/Archiving/UserDataTemplateModelProvider.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Features/Archiving/UserDataTemplateModelProvider.cs#L54) (excerpt; see source for context).

{% code title="UserDataTemplateModelProvider.cs" %}
```csharp
public override IDataTemplateModel GetModel(IDataTemplate template, object argument)
{
	var schema = $"*, {nameof(UserProfile.Site)}" + "{*}";

	if(argument is Stream stream)
		argument = Serializer.Json.Deserialize<UserProfileCriteria>(stream);

	var data = argument switch
	{
		string key => this.Services.ResolveRequired<UserService>().Get(key, schema),
		IModel model => this.Services.ResolveRequired<UserService>().Select(Criteria.Transform(model), schema),
		_ => this.Services.ResolveRequired<UserService>().Select(null, schema),
	};

	return new DataTemplateModel(new { Users = data });
}
```
{% endcode %}

When the input is a stream, it is first deserialized into UserProfileCriteria; the string is queried by key, and the condition model is converted into data conditions. In other cases, ordinary selection is performed. The schema additionally expands Site, and what is finally handed over to the template is a model containing the Users member. There is no self-created demo user list here.

## Templates Are Delivered with the Web Package

Source: [src/api/Zongsoft.Discussions.Web.deploy](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/api/Zongsoft.Discussions.Web.deploy#L4) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.Web.deploy" %}
```ini
[templates]
artifacts/templates/*.xlsx
```
{% endcode %}

Template paths, provider names, return model members, and in-workbook references need to be maintained together. When modifying the Users or Site structure, in addition to checking C# compilation, verify that the actual workbook can be rendered.

## The Difference Between Engines and Cases

Discussions implements data preparation and does not have ClosedXml or OpenXml pinned in its manifest. Deployers can refer to frames [ClosedXml template test](https://github.com/Zongsoft/framework/blob/main/externals/closedxml/test/SpreadsheetTemplateTest.cs) and [OpenXml test](https://github.com/Zongsoft/framework/blob/main/externals/openxml/test/SpreadsheetDocumentTest.cs) when selecting an engine. This is a case of framework supplementation and one cannot claim that the forum has enabled both implementations.

## Exported and Imported Business Boundaries

Exports must preserve user and site permissions and cannot allow arbitrary filtering to expand the scope of the data. The import also requires row and column verification, duplicate recording strategy and transaction design; the user list export template does not mean that the complete forum import workflow has been implemented.

Continue reading: [ClosedXml](projects/closedxml.md), [OpenXml](projects/openxml.md), [Reporting](../reporting.md).
