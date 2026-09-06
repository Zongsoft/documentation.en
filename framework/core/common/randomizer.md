---
description: "Explain the real purpose of the random string in the body file name of Discussions."
icon: shuffle
---

# Randomizer


Randomizer provides random bytes, integers and strings. Discussions uses a random string as part of the file name to avoid sharing the same file name when new content has not yet obtained a database number.

Source: [src/Services/PostService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/PostService.cs#L222) (excerpt; see source for context).

{% code title="PostService.cs" %}
```csharp
protected virtual string GetContentFilePath(ulong postId, string contentType)
{
	return Utility.GetFilePath(string.Format("posts/post-{0}-{1}.txt", postId.ToString(), Zongsoft.Common.Randomizer.GenerateString()));
}
```
{% endcode %}

PostId provides business clues, and the random suffix reduces the chance of path conflicts with the same number or unallocated numbers. The path also goes through Utility.GetFilePath plus configuration and user scope. Random file names are not a substitute for access control and do not constitute a guarantee of absolute uniqueness.

## Method Selection

| method | function |
| --- | --- |
| Generate | Random bytes of specified length |
| GenerateInt16、GenerateInt32、GenerateInt64 | signed integer |
| GenerateUInt16、GenerateUInt32、GenerateUInt64 | unsigned integer |
| GenerateSecret | key style string |
| GenerateString | Ordinary random string, can be limited to numbers |

This business does not directly use GenerateSecret as a credential, so the file naming method cannot be directly promoted as an authentication token scheme. See [RandomizerTest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Common/RandomizerTest.cs) for framework use cases for string length and character range.

Related reading: [file system](../io.md), [Write Operations](../../data/writing.md).
