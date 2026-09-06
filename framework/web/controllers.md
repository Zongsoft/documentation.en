---
description: "Build and deploy Discussions Web class library, check controller discovery and request prerequisites."
icon: plug
---

# Deploy Controller Plugins


The API project of Discussions is a Web class library, and the OutputType is Library. It contains the controller but has no independently launchable entry and must be deployed together with [Web Host](../../hosting/web.md) and domain plugins.

## Build the Actual Project

First build Core and Web in the sibling framework repository as described in [Your First Business Plugin](../../get-started/first-business-plugin.md), using the same configuration and target framework. See [Prerequisites](../../get-started/prerequisites.md) for the current dependency requirements.

{% code title="Build the API from the discussions root directory" %}
```powershell
dotnet build src/api/Zongsoft.Discussions.Web.csproj -f net10.0 -p:ZongsoftFrameworkPathReferenced=true -p:GeneratePackageOnBuild=false
```
{% endcode %}

This also builds the domain library. Local references do not build framework automatically. Once the required Core version is available from your NuGet source, you can omit the property and return to the default package-reference path.

## Web Manifest Dependency Domain Plugin

Source: [src/api/Zongsoft.Discussions.Web.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/api/Zongsoft.Discussions.Web.plugin#L9) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.Web.plugin" %}
```xml
<manifest>
	<dependencies>
		<dependency name="Zongsoft.Discussions" />
	</dependencies>

	<assemblies>
		<assembly name="Zongsoft.Discussions.Web" />
	</assemblies>
</manifest>
```
{% endcode %}

Dependencies ensure that domain modules are assembled first, and the assembly declaration allows the host to discover the controller. Deploying only Web DLLs without domain mapping, identity extensions, and data drivers will result in controllers existing but business calls failing.

## Resources Delivered with the Package

Source: [src/api/Zongsoft.Discussions.Web.deploy](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/api/Zongsoft.Discussions.Web.deploy#L1) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.Web.deploy" %}
```ini
artifacts/Zongsoft.Discussions.Web.plugin
lib/$(Framework)/Zongsoft.Discussions.Web.*

[templates]
artifacts/templates/*.xlsx
```
{% endcode %}

User archive templates are in templates. The project also enters the docs/http request file into artifacts/http for interface verification; the address and credentials in the request must be provided by its own environment.

## Verify Path

First confirm that the Discussions and Discussions.Web lists are loaded, and then confirm that Threads, Forums, Users and other controllers enter the application model. Then check the authentication, site, connection, mapping and external file configuration, and finally verify the query and business action.

Public actions are based on the current routing of the controller. repository The early docs/api.md is a historical interface draft; singular paths therein should not be considered current executable examples without checking. See [Requests and Data Service APIs](data-services.md) for the specific code of subject review.
