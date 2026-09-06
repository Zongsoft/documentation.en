---
description: "Understanding serialization boundaries from Discussions user archive filtering and model properties."
icon: brackets-curly
---

# Zongsoft.Serialization


Serializer.Json supports Zongsoft model and data dictionary and other framework types based on System.Text.Json. It is used by the Discussions user archive provider to reduce the input stream to a conditional model and then hands it to the data service.

## The Real Input Is Not an Arbitrary Object Graph

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

The stream input explicitly specifies UserProfileCriteria, which is then converted into data conditions via Criteria.Transform. String input is queried according to the user key; these two paths are different, and the input should not be deserialized into a Dictionary and then spliced into SQL.

## Model Fields and Output Ranges

Source: [src/Models/Post.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Models/Post.cs#L110) (excerpt; see source for context).

{% code title="Post.cs" %}
```csharp
[Serialization.SerializationMember(Ignored = true)]
public IEnumerable<PostVoting> Upvotes => this.Votes == null ? Array.Empty<PostVoting>() : this.Votes.Where(vote => vote.Value > 0);

/// <summary>获取帖子的被踩记录集。</summary>
[Serialization.SerializationMember(Ignored = true)]
public IEnumerable<PostVoting> Downvotes => this.Votes == null ? Array.Empty<PostVoting>() : this.Votes.Where(vote => vote.Value < 0);
#endregion
```
{% endcode %}

The ignored member attribute controls serialization visibility and does not mean that these members do not exist in the database, nor does it replace schema or authorization. Before the model returns the text, it will also go through audit filtering and external content reading.

## Framework Options Reference

| Options | Purpose |
| --- | --- |
| Indented | Readable indented output |
| NamingConvention | Member naming convention |
| IgnoreNull、IgnoreZero | Control empty or default value output |
| IncludeFields | Whether to include fields |
| MaximumDepth | object graph depth limit |
| Typified | Preserve type information for loose values |

Discussions does not currently use Typified or a custom JSON converter as an archive entry point. Related real tests can be found at [JsonSerializerTest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Serialization/JsonSerializerTest.cs) and [TextSerializationOptionsTest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Serialization/TextSerializationOptionsTest.cs).

## Security and Lifecycle

Successful deserialization only indicates that the input can be parsed, but does not indicate that the caller has the right to query or export the corresponding user. Limit input size and object depth, propagate cancellation signals, and let the service layer continue to validate business scope. The workbook structure of the archive output is shown in [spreadsheets and templates](../externals/documents.md).
