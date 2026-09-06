---
description: "Zongsoft toolchain overview."
icon: screwdriver-wrench
---

# Tool Overview

Zongsoft toolchain is used to assist deployment, packaging, upgrade and development and debugging.

## Tool List

| Tools | Commands | Purpose |
| --- | --- | --- |
| Zongsoft.Tools.Deployer | `dotnet deploy` | Deploy plugins and accompanying files based on `.deploy` files |
| Zongsoft.Tools.Packager | `dotnet-pack` | Make `.tar.gz`, `.deb`, `.rpm` installation packages |
| Zongsoft.Tools.Upgrader | `dotnet-upgrade` | Produce, verify and publish automatic upgrade packages |
| Zongsoft.Tools.Regular | GUI | Regular expression testing tool |

## Recommended Order of Use

1. Use `dotnet deploy` to deploy the plugin to the host.
2. Use host to complete run verification.
3. For initial installation delivery, you can use `dotnet-pack` to create an installation package.
4. For self-upgrading existing applications, you can use `dotnet-upgrade` to create an upgrade package; it is not required to generate deb/rpm first.

## Related Resources

* [tools repository](https://github.com/Zongsoft/tools)
* [tools Chinese README](https://github.com/Zongsoft/tools/blob/main/README.zh-Hans.md)
* [tools English README](https://github.com/Zongsoft/tools/blob/main/README.md)
* [Zongsoft.Tools.Deployer NuGet package](https://www.nuget.org/packages/Zongsoft.Tools.Deployer)
* [Zongsoft.Tools.Packager NuGet package](https://www.nuget.org/packages/Zongsoft.Tools.Packager)
* [Zongsoft.Tools.Upgrader NuGet package](https://www.nuget.org/packages/Zongsoft.Tools.Upgrader)


Parameters and formats of each tool: [deployer](deployer.md), [Install packager](packager.md), [Upgrade packager](upgrader.md), [Regular tester](regular.md). The command examples distinguish between PowerShell and Bash. Before execution, confirm the source directory, target directory, and actual application identity.
