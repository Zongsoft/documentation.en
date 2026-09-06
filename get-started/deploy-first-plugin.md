---
description: "Deploy the Discussions field and web plugin along with the existing hosting solution."
icon: rocket
---

# Deploy Your First Plugin

This article uses the real web host of the hosting repository and the Discussions deployment item. Requires peer framework, hosting, discussions source code, and its own isolation database, identity and file storage configuration.

## 1. Confirm That the Existing Solution Already Contains Business Plugins

Source: [hosting/web/web.deploy](https://github.com/Zongsoft/hosting/blob/main/web/web.deploy#L29) (excerpt; see source for context).

{% code title="web.deploy" %}
```ini
[plugins zongsoft discussions]
nuget:Zongsoft.Discussions@0.8.0
nuget:Zongsoft.Discussions.Web@0.8.0
```
{% endcode %}

0.8.0 is the package version locked by the current plan, which does not mean that it will always be the latest on NuGet. The domain library and web library are deployed together; data, security and other public plugins are provided by the remaining entries in the same scheme.

## 2. Distinguish Between Host Publishing and Business Construction

The hosting project is located at hosting/web/default, and the entry point is Zongsoft.Hosting.Web.dll. By default, Core, Web, Plugins, and Plugins.Web are referenced from the sibling framework output, so the same target framework and configuration of these projects must be built before publishing. The command is executed from hosting/web/default:

{% code title="Publish a real web host" %}
```powershell
dotnet publish Zongsoft.Hosting.Web.csproj -c Release -f net10.0 -o ./out
```
{% endcode %}

This command only prepares the host run file. Locally built discussions will not automatically replace NuGet 0.8.0 in the deployment manifest; see [Business plugin](first-business-plugin.md) for local source code building steps.

## 3. Prepare the Configuration for the Isolation Environment

Check the application, data, security, and file configurations in hosting/.deploy/default/options and replace them with your own test endpoints. The mapping of Discussions includes external serial numbers and entity-driven selections, and the database script must match the actual deployment, see [First query](../framework/data/quickstart.md).

Do not run deploy.cmd directly as a learning validation: the script also cleans the plugin directory and may continue packaging. Below we use the same deployer and existing scheme, with the target set to the out directory from the previous step:

{% code title="Collect plugins from hosting/web/default" %}
```powershell
dotnet deploy .deploy --destination:./out --host:web --site:default --scheme:default --environment:development --debug:off --edition:Release --framework:net10.0 --platform:win --architecture:x64
```
{% endcode %}

Parameters come from real deployment scripts; the platform and architecture should be adjusted according to the actual operating environment. The configuration in the environment directory must be reviewed first. The command will not create a security test database for you.

## 4. Check Whether the Local Modification Has Entered the Runtime Directory

When using a NuGet manifest, the in-package version is run. To debug local modifications, after stopping the host, update the DLL/PDB built by discussions and the corresponding plugin, option, and mapping to out/plugins/zongsoft/discussions, and retain the Web template directory. You cannot mix old packages and new source code and still troubleshoot according to the same version.

See [Deployment File Format](../references/deploy-files.md) for a list of domain and web resources. After deployment you should see both plugin manifests, realm mapping, options and user list templates.

## 5. Start from the Runtime Directory and Verify

{% code title="Start the web host" %}
```powershell
Set-Location ./out
dotnet ./Zongsoft.Hosting.Web.dll
```
{% endcode %}

The listening address is provided by the host configuration. Confirm /Application first, and then verify the forum interface with a read-only query of discussions/docs/http/forum.http. 401/403, 404, connection failure and empty results represent different problems. Check them layer by layer according to [Run and Debug](run-and-debug.md).

This path connects the business dependencies in the configuration. Offline compilation and regression checking during the document migration phase cannot replace a complete deployment acceptance of your environment.
