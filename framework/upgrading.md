---
description: "Understand the cooperation of upgrade packages, discovery channels, in-app upgraders, and out-of-process deployers."
icon: rotate
---

# Automatic Upgrades

Automated upgrades break application delivery into package production, release discovery, download preparation, and downtime deployment. The four components in `framework/upgrading` share a release agreement but assume responsibilities at different stages.

| components | operating position | Responsibilities |
| --- | --- | --- |
| `Zongsoft.Tools.Upgrader` | Build or release environment | Create ZIP and manifest, recalculate checksum, publish |
| `Zongsoft.Upgrading.Web` | Package management server | Manage release metadata, package uploads, and discovery filters |
| `Zongsoft.Upgrading.Upgrader` | Within the upgraded app | Check, download, verify, decompress, generate deployment and handover |
| `Zongsoft.Upgrading.Deployer` | Independent process outside the application | Wait for the app to exit, copy files, execute deployment commands and restart |

For first-time integration, it is recommended to read this page first and then configure the isolation test application according to [Upgrade Integration and Recovery](upgrading/workflow.md); see [Upgrade Tool](../tools/upgrader.md) for the parameters of the production package.

## Why You Need a Standalone Deployer

A running application may occupy assembly and resource files, and the application cannot continue to execute its own replacement code after exiting. Therefore, the in-app upgrader is only responsible for preparing the package, and finally starts the standalone deployer and exits.

`.deployment` is the handover description of both parties, referencing the manifest and the decompressed file directory. Its existence represents work to be deployed, but does not mean that the target version is already running. The deployer waits for the original process to exit and reads the description file exclusively to avoid contention.

## Publish Identity

A release is identified by application name, version distribution name, version number, platform and architecture. The application name must match the actual runtime name, do not guess based on the project directory or historical packaging scripts. The version distribution name can be used to distinguish release series such as `stable`, `community`, etc.; it is a different dimension from the build configuration `Debug/Release`.

When selecting a release, the current version, optional highest target version, obsolete status, and visible and published status on the web are also considered. With only a ZIP uploaded, without matching metadata and release status, clients may still not find updates.

## Full Amount and Incremental

| Type | File deployment method | Things that require application guarantees |
| --- | --- | --- |
| `Fully` | Clean the application root directory first, then copy new files; keep the deployer itself and other content required for handover | Contains complete runtime artifacts; persistent data must have an independent storage strategy |
| `Delta` | Overwrite the package contents to the original directory | The required old files do exist; deleting old files must be designed separately. |

If the client selects full release, the latest matching full package will be used as the backbone, and then updated incremental packages will be applied in ascending version order; if there is no full package, matching incremental packages will be applied in order. Delta is not a binary difference algorithm and does not automatically find old files that should be deleted.

{% hint style="danger" %}
🚨 Full deployment may delete the database, uploaded files, logs and local configuration in the application root directory. Place persistent data in a controlled, independent location or complete verifiable backup and recovery arrangements before upgrading. The `Deploying` executor runs after cleaning and cannot be relied upon to back up files that have been cleaned.
{% endhint %}

## Channels and Trust Boundaries

The application side supports `File` and `Web` channels. `File` looks for manifests and packages from the configured file system URL, which can use a deployed file system provider; `Web` requests the discovery interface of the package manager. The tool release supports `amazon.s3` (alias `s3`) and `web`. Do not confuse the channel names at both ends.

The checksum is used to check whether the package is consistent with the content declared in the manifest, and is not equal to publisher identity authentication. Apps still need to control publishing permissions, storage write permissions, and download channels. Executors in the manifest perform file operations and should come from the same trusted release process as the application.

## Completion Conditions

An upgrade round is only completed when the new process is started, the version identification is as expected, and the business health check passes. Successful package upload, successful download and `.deployment` generation are only intermediate stages. The framework's current file cleanup and overwrite process does not provide atomic switching across files, nor does it promise automatic rollback.

Source references: [Upgrade components](https://github.com/Zongsoft/framework/tree/main/upgrading), [sharing agreement](https://github.com/Zongsoft/framework/tree/main/upgrading/.shared).
