---
description: "Use dotnet-deploy to combine local and NuGet artifacts into a verifiable host run directory."
icon: truck-ramp-box
---

# Deployment Tool

`dotnet-deploy` reads the `.deploy` file, performing file copying, NuGet package parsing, and explicit deletion. It solves the assembly problem of the runtime directory and is not responsible for business data migration, system service installation or ensuring that all DLL versions are automatically compatible.

## Installation and Version

{% code title="InstallDeployer.ps1" %}
```powershell
dotnet tool install -g Zongsoft.Tools.Deployer
dotnet tool list -g
```
{% endcode %}

Use `dotnet tool update -g Zongsoft.Tools.Deployer` when installed. The team’s release process should pin tool versions and record package versions to avoid different machines selecting different latest packages.

## Minimal Deployment

Execute in the project directory containing `.deploy` and specify the target run directory and framework parameters:

{% code title="DeployApplication.ps1" %}
```powershell
dotnet deploy .deploy --destination:./out --edition:Release --framework:net10.0 --platform:win --architecture:x64
```
{% endcode %}

{% code title=".deploy" %}
```ini
[plugins]
nuget:Zongsoft.Plugins/plugins/Main.plugin

[plugins zongsoft data]
nuget:Zongsoft.Data
```
{% endcode %}

This is just a plugin combination fragment. The target directory also needs the host’s runtime artifacts. See [Deploy Your First Plugin](../get-started/deploy-first-plugin.md) for the complete workflow. When specifying other files, replace `.deploy` with the corresponding files; you can also pass in multiple lists according to the application plan.

## File Selection and Overwriting

Sections represent target subdirectories, and entries specify local files, packages, or deletion actions. See [Deployment format reference](../references/deploy-files.md) for variables, conditions and path details.

The existing values of `--overwrite` are `alway`, `never`, and `newest`; `alway` is the actual spelling of the tool, do not change it to `always` by yourself. `newest` decides whether to overwrite based on its file policy, which does not mean selecting the highest version based on assembly version.

{% hint style="warning" %}
🚨 Deployment is a sequential file operation. Transitive dependencies of subsequent packages may overwrite previously copied DLLs or even become older versions; NuGet entries do not resolve versions across the entire deployment graph as a single unit. Check the final directory’s assembly versions and the first business call.
{% endhint %}

## NuGet and Ancillary Resources

When the path within the package is not specified, the package root `.deploy` is processed first, otherwise the `lib` asset that is most suitable for the target framework is selected. Manifests within framework packages may also copy `.plugin`, `.option`, `.mapping`, SQL, language libraries, or native resources.

Some dependency prefixes are ignored by default, including `System.`, `Microsoft.Extensions.`, `Zongsoft.`. Therefore, when relying on the host base library or other Zongsoft plugins, you need to check the host artifacts and explicit manifest combinations, and cannot just rely on transitive copying.

## Inspection and Acceptance

When the path contains a Windows drive letter, prevent it from being mistakenly recognized as a resolver, such as using `:D:/...`; when the framework assets are missing, check `framework` and the actual package directory; when the configuration does not appear, check the conditions and application variables.

Check the error output, target files and running results at the same time during acceptance. Currently some undefined resolver errors do not result in a non-zero exit code, and the deployment cannot be deemed successful based on the script exit code alone. Windows Automation also requires a valid console handle, and console errors should be checked in the actual terminal where the tool is running.

Source references: [deployer](https://github.com/Zongsoft/tools/tree/main/deployer), [Complete command description](https://github.com/Zongsoft/tools/blob/main/deployer/README.zh-Hans.md).
