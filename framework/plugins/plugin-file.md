---
description: "Discussions list explaining assembly, dependencies, extension paths and builtins."
icon: puzzle-piece
---

# Plugin Manifests and Loading


Discussions Declare how the module integrates with the host via the.plugin manifest. Manifest is responsible for assembly, NuGet or.deploy is responsible for delivery, and they cannot replace each other.

## Load Assembly and Dependencies First

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L9) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<manifest>
	<assemblies>
		<assembly name="Zongsoft.Discussions" />
	</assemblies>

	<dependencies>
		<dependency name="Zongsoft.Data" />
		<dependency name="Zongsoft.Security" />
	</dependencies>
</manifest>
```
{% endcode %}

The assembly contains models, services and module entries; dependencies require the data engine and security modules to be assembled first. Dependency names are plugin names and are not equivalent to automatically downloading the corresponding NuGet packages; the host deployment must make them actually exist.

## Expose Shared Module Instances

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L20) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<extension path="/Workbench/Modules">
	<object name="Discussions" value="{static:Zongsoft.Discussions.Module.Current, Zongsoft.Discussions}">
		<expose name="Accessor" value="{path:../@Accessor}">
			<expose name="Filters" value="{path:../@Filters}" />
		</expose>

		<expose name="Events" value="{path:../@Events}" />
		<expose name="Properties" value="{path:../@Properties}" />
	</object>
</extension>
```
{% endcode %}

The static expression refers to Module.Current. expose integrates the members of the existing object into the plugin tree; the relative path here is parsed from the builtin context to facilitate subsequent mounting of filters, events and properties.

## Mount Query Filter to Accessor

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L35) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<extension path="/Workbench/Modules/Discussions/Accessor/Filters">
	<object name="PostFilter" type="Zongsoft.Discussions.Data.PostFilter, Zongsoft.Discussions" />
	<object name="ThreadFilter" type="Zongsoft.Discussions.Data.ThreadFilter, Zongsoft.Discussions" />
</extension>
```
{% endcode %}

The filter runtime is also controlled by the model matching feature; a successful mount does not mean that any query will execute the same filter. PostFilter handles posts, ThreadFilter handles topics and their bodies, see [Data Services](../data/services.md).

## Validators and Security Extensions

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L31) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
<extension path="/Workbench/Data/Validators">
	<object name="Discussions" type="Zongsoft.Discussions.Data.DataValidator, Zongsoft.Discussions" />
</extension>
```
{% endcode %}

The validator is named Discussions and corresponds to the business scope of the module accessor and mapping container. The identity challenger and converter use the other two extension paths, see [Certification](../security/authentication.md).

## Troubleshoot Loading Failure

Check in order whether the file is deployed, whether the XML is valid, whether dependencies exist, whether the assembly can be loaded, whether types and expressions can be parsed, and whether the target extension point exists. In particular, distinguish between plugin manifest filename, plugin name, and module name: they are related in Discussions, but participate in configuration matching, dependency ordering, and module service scoping respectively.
