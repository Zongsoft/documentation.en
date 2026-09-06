---
description: "Understand schema controls on fields and navigation from user archives, browsing history, and topic auditing."
icon: brackets-curly
---

# Data Schemas


Data schema is the schema text passed to data access or data service, describing which members are included in this operation. The relationship definition comes from [mapping](mapping.md), and the schema only selects the shape used this time; it is not another database structure.

## User Archive Requires Site Information

Source: [src/Features/Archiving/UserDataTemplateModelProvider.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Features/Archiving/UserDataTemplateModelProvider.cs#L56) (excerpt; see source for context).

{% code title="UserDataTemplateModelProvider.cs" %}
```csharp
var schema = $"*, {nameof(UserProfile.Site)}" + "{*}";
```
{% endcode %}

This true pattern selects the User's simple fields and expands the Site's simple fields. An asterisk alone does not automatically expand all navigation, nor does it automatically complete dependent fields of model computed properties. The archiving provider then packages the results as Users and gives them to user-list.xlsx, see [spreadsheets and templates](../externals/documents.md).

## Browsing Historyexpand Topic

Source: [src/Services/UserService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/UserService.cs#L67) (excerpt; see source for context).

{% code title="UserService.cs" %}
```csharp
public IEnumerable<History> GetHistories(uint userId, Paging paging = null)
{
	if(userId == 0)
		userId = this.Principal.Identity.GetIdentifier<uint>();

	return this.DataAccess.Select<History>(Condition.Equal(nameof(History.UserId), userId), $"*, {nameof(History.Thread)}" + "{*}", paging);
}
```
{% endcode %}

History.Thread relationships are connected by maps. Expand only if the list truly requires subject details; collections or deep relationships may increase query times and result size.

## Write Mode Contains Associated Members

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L205) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
//确保数据模式含有“主题内容贴”复合属性
schema.Include("Post{*}");
```
{% endcode %}

When ThreadService inserts a topic, it first confirms that the text exists, and then adds Post to writing mode. The audit action limits the Approved field of Post to prevent a status action from updating irrelevant text by the way.

## Syntax Reference and Framework Use Cases

| Grammar | meaning |
| --- | --- |
| Comma separated members | Select multiple fields or navigation |
| asterisk | Simple members that can be matched by the current layer |
| Exclamation mark plus member | Exclude members from current selection |
| Dotted path or curly brace | Select members along a mapping relationship |
| colon plus quantity | Collection navigation limit, not page number |
| Order members within parentheses | Specify the sorting of navigation results. A negative sign or tilde indicates reverse order. |

Discussions There is no business fragment using navigation limits, and the relevant boundaries use the frames [SchemaTest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Data/SchemaTest.cs) and [SchemaParserTest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/test/SchemaParserTest.cs) as a reference. The root result set is paginated using Paging, see [Query](querying.md).

## Permissions and Calculated Fields

Mode controls read shapes, not authorization rules. When asking the caller to exclude audit judgment fields such as Approved and CreatorId, the text cannot be visible by default; the result filter of Discussions uses shielding processing for this. Calculated members also do not automatically derive their required original fields, the schema should be qualified by the service.

{% hint style="warning" %}
🚨 Write mode does not give the caller permission to modify site, creator, or other immutable fields. Mappings, validators, service actions, and caller identities still need to be checked simultaneously.
{% endhint %}
