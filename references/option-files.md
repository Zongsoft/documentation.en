---
description: "Describes configuration keys, file matching, and overwriting with real options and consumption codes from Discussions."
icon: sliders
---

# Option Configuration Files


.option provides run parameters; plugin manifest, assembly deployment, and configuration reading are different steps. Discussions' configuration currently only has the General section, which defines the site number and file storage base path.

## From File to Configuration Key

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

path specifies the configuration section, and the properties of general generate Discussions:General:SiteId and Discussions:General:BasePath. Attribute values are first loaded as text and then converted by the consumer; key name comparison ignores case.

Source: [src/Utility.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Utility.cs#L283) (excerpt; see source for context).

{% code title="Utility.cs" %}
```csharp
var basePath = ApplicationContext.Current.Configuration.GetOptionValue<string>("/Discussions/General.BasePath");
```
{% endcode %}

What Utility actually reads is BasePath. The presence of SiteId in the configuration does not mean that all services will use it as the tenant fallback value; the tenant identity comes from [Identity Transformers and Authenticators](../framework/security/authentication.md). This is an important distinction when checking "configuration exists" versus "business is actually consumed".

{% hint style="info" %}
💡 The S3 paths in the source code illustrate the real configuration and do not contain credentials for documentation readers. The runtime should provide the base directory and corresponding file system plugins from its own isolation environment.
{% endhint %}

## XML Scalars and Collections

Scalars use attributes; XML text nodes are processed as collection items, and attribute values should not be rewritten into sub-elements with the same name and assumed to have the same semantics. Named collections use the element name plus.name or.key to specify member names. For specific use cases, see the framework [Option parsing test](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Configuration/Xml/OptionConfigurationTest.cs).

## Primary Names for Lists and Options

Discussions' Zongsoft.Discussions.plugin has the same main file name as Zongsoft.Discussions.option. The plugin configuration provider matches options by the file name of the loaded manifest, not by an arbitrary search by the plugin's name attribute. Just having the correct XML won't work when the file isn't deployed or the manifest isn't loaded.

The environment, host, and site parameters also affect attachment file matching. Host-level and plugin-level rules are different. Some environment attachment patterns at the plugin level contain hyphens. When you need to add an environment file, you should check [Plugin configuration provider](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/Configuration/PluginConfigurationProvider.cs). Candidate rules in the document cannot be regarded as files that already exist in the repository.

## Overwrite and Reread

Within the same plugin, configuration providers added later will return keys with the same name first; do not rely on wildcard enumeration order or cross-plugin order to achieve key coverage. The file supports change loading, which does not mean that the consumer must rebind and the database connection must be rebuilt.

Utility.GetFilePath reads BasePath when called; other components may cache the setting and must check their respective implementations. The troubleshooting sequence is: manifest loading, option deployment, name matching, actual key generation, consumption code reading.

Related reading: [plugin manifest](../framework/plugins/plugin-file.md), [file system](../framework/core/io.md), [Deployment files](deploy-files.md).
