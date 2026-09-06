---
description: "Create ZIP upgrade packages and manifests, and understand file selection, verification and update, and release sequence."
icon: arrows-rotate
---

# Upgrade Tool

`Zongsoft.Tools.Upgrader` is located in `upgrading/tool` of the framework and generates ZIP and manifest for consumption by [Automatic Upgrades](../framework/upgrading.md). It is a different tool than `dotnet-pack` which makes deb/rpm.

## Installation and Input Directories

{% code title="InstallUpgradeTool.ps1" %}
```powershell
dotnet tool install -g Zongsoft.Tools.Upgrader
```
{% endcode %}

Build, publish, and deploy the complete application to a standalone directory before packaging. Avoid packaging running directories directly: logs or databases may be locked, and contents may change during collection.

## Make Full Package

Discussions There is no independent upgrade packaging script. Please refer to the existing daemon host hosting/daemon/upgrade.pack.cmd. The following is its actual command. The variables are set at the front of the script, using cmd's line continuation and variable syntax. To execute the complete script, first check the source code root path and output directory.

Source: [hosting/daemon/upgrade.pack.cmd](https://github.com/Zongsoft/hosting/blob/main/daemon/upgrade.pack.cmd#L56) (excerpt; see source for context).

{% code title="upgrade.pack.cmd" %}
```bat
dotnet-upgrade pack               ^
	--name:Zongsoft.Daemon        ^
	--kind:fully                  ^
	--edition:%edition%           ^
	--version:%version%           ^
	--checksum:sha1               ^
	--compilation:%compilation%   ^
	--framework:%framework%       ^
	--platform:%platform%         ^
	--architecture:%architecture% ^
	--source:"D:\\Zongsoft\\hosting\\daemon\\bin\\$(compilation)\\$(framework)" ^
	--output:../../../
```
{% endcode %}

This script currently selects sha1, and this article retains the actual parameters for verification; release integrity requirements and algorithms should be specified by the application release process, and checksum itself does not provide signature authentication.

`name` must match the runtime application name, and `edition` is the release distribution name; it is different from the variable of the same name in deployer that represents the compiled output configuration. `version`, `platform`, and `framework` are necessary information, and the architecture must conform to the target application.

When the output file name is not specified, name it according to `{name}[-edition]@{version}_{runtime}`, generate ZIP and pair `.manifest`. The manifest contains identity, type, size, checksum, package path, and optional description and executor.

## Select Files and Variables

When there is no positional parameter, the entire source directory is collected. After specifying the parameter, select by file or directory. Supports the last level `*`, `?`, `source:target` for relocation; the target is empty, `~` or `/` can represent the package root directory. Duplicate ZIP entries are warned and skipped, not silently overwritten.

Exclusion rules are separated by semicolons and support for `**` globstar should not be assumed. Variables support `$(name)` and `%name%`, use command options first, then use environment variables; PowerShell should pass literal variables that require tool expansion in single quotes.

`Delta` is just an overlay incremental file package and does not automatically generate binary differences. Before choosing full or incremental, read [Release types and cleanup boundaries](../framework/upgrading.md).

## Checksum Will Modify the List

Source: [framework/upgrading/tool/README.zh-Hans.md](https://github.com/Zongsoft/framework/blob/main/upgrading/tool/README.zh-Hans.md#L289) (excerpt; see source for context).

{% code title="README.zh-Hans.md" %}
```shell
dotnet-upgrade checksum --algorithm:sha1 Zongsoft.Daemon-stable@1.1.0_win-x64.zip
```
{% endcode %}

The command accepts ZIP, manifest, or package name without extension, and recalculates and updates the paired manifest. It is not a read-only "validation pass/fail" check; when comparing the original release to see if it has been tampered with, the trusted original verification information should be saved and not recalculated first to overwrite it.

## Release Channels and Order

`publish` supports `amazon.s3` (alias `s3`) and `web`. The S3 channel uploads the package and manifest; the Web channel is executed in the order of importing the manifest, uploading the corresponding package, and marking it as published. See [publishing options](https://github.com/Zongsoft/framework/blob/main/upgrading/tool/README.zh-Hans.md) for specific credentials and target parameters.

Check ZIP entries, manifest identities, and pairings before publishing. After publishing, you also need to verify that the discovery interface can return the correct version for the target application, rather than just checking for the presence of a ZIP in the storage.

## Executor and Completion Judgment

The executor option uses `--executor.<name>@<event>:"command"`, currently built-in Copy, Move, Link, and Delete, and the events include Deploying and Deployed. They operate on files during the deployment phase, not normal release notes; missing event name definitions will warn and ignore them.

{% hint style="warning" %}
🚨 The current packaging failure may leave an incomplete ZIP and the process exit code may still be zero. The acceptance must also check the log, ZIP readability and pairing manifest. Success cannot be judged only by the exit code.
{% endhint %}

After the upgrade is completed, you need to check the target process and business health. For details, see [Upgrade integration and recovery](../framework/upgrading/workflow.md).
