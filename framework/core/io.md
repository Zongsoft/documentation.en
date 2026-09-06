---
description: "Upload the description of the virtual file system and path in the Discussions long text and attachments."
icon: folder-tree
---

# Zongsoft.IO


Discussions places short text in the database, saves longer text as a file, and saves the file path back to the data model. Zongsoft.IO provides a unified file interface, the actual local disk or object storage implementation is determined by the deployment.

## The Base Path Comes from Configuration

Source: [src/Zongsoft.Discussions.option](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.option#L1) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.option" %}
```xml
<?xml version="1.0" encoding="utf-8" ?>

<options>
	<option path="/Discussions">
		<general siteId="1" basePath="zfs.s3:/zongsoft-discussions/" />
	</option>
</options>
```
{% endcode %}

The real configuration uses the zfs.s3 scheme. Operation requires a corresponding file system implementation and its own storage configuration; this path is not a public space directly available to document readers. Utility.GetFilePath then organizes directories by site, user, and relative paths.

## Write Text Using the Same Entry

Source: [src/Utility.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Utility.cs#L241) (excerpt; see source for context).

{% code title="Utility.cs" %}
```csharp
public static bool WriteTextFile(string path, string content)
{
	if(string.IsNullOrWhiteSpace(path))
		throw new ArgumentNullException(nameof(path));

	if(string.IsNullOrWhiteSpace(content))
		return false;

	using(var stream = FileSystem.File.Open(path, FileMode.Create, FileAccess.Write))
	{
		using(var writer = new StreamWriter(stream, System.Text.Encoding.UTF8))
		{
			writer.Write(content);
		}
	}

	return true;
}
```
{% endcode %}

FileSystem.File implementation is selected based on path; stream and text writers are released by scope. Database transactions cannot roll back the files that have been generated here, and the calling business needs to handle failure compensation.

## The Difference Between Content and Path

Source: [src/Utility.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Utility.cs#L48) (excerpt; see source for context).

{% code title="Utility.cs" %}
```csharp
public static bool IsContentEmbedded(string contentType)
{
	if(string.IsNullOrEmpty(contentType))
		return true;

	return contentType.TrimEnd().EndsWith(CONTENT_TYPE_EMBEDDED_SUFFIX, StringComparison.OrdinalIgnoreCase);
}
```
{% endcode %}

Empty type or +embedded suffix indicates that the content field saves the text; other types indicate external files. This tag is a Discussions business convention, not a universal MIME transport rule. After reading the external file, it should be marked as inline synchronously to prevent the downstream from using the text as a path again.

## HTTP Attachment Upload

Source: [src/api/Controllers/FileController.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/api/Controllers/FileController.cs#L79) (excerpt; see source for context).

{% code title="FileController.cs" %}
```csharp
var infos = this.Accessor.Write(this.Request,
							  this.DataService.GetDirectory(id),
							  args => args.FileName = $"{Timestamp.Millennium.Epoch.GetElapsed().Days}-{Randomizer.GenerateString()}", cancellation);
```
{% endcode %}

The upload directory is calculated by the FileService, and the filename uses the epoch day and a random string. The controller then converts the file information into a business File model and cleans up the just-saved file when writing fails. The original client file name is different from the server storage name, and display and download should be interpreted according to the business model.

## Operation and Troubleshooting

Utility.ReadTextFile currently returns an empty string when the file does not exist or the read fails. When troubleshooting, you must check the path saved in the data, the file system plugin, the storage access permissions, and whether the file exists. You cannot attribute empty text to the database.

Set up an independent directory for each isolation environment; the site directory is a storage organization method and cannot replace interface permissions. For file models, content tags, and external storage sources, see [real case](../../cases.md) and [Amazon Extension](../externals/projects/amazon.md).
