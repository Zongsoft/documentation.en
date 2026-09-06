---
description: "Understand the overall model of deployment, packaging, and upgrades in Zongsoft."
icon: truck-ramp-box
---

# Deployment Model

Zongsoft's deployment model revolves around plugin-based applications. Deployment is not simply copying the compilation output, but assembling plugins, configurations, mappings, certificates and resources into the target directory of the host according to rules.

## Deploy

Deployment is performed by the `dotnet-deploy` tool and the rules are written in the `.deploy` file. The deployment file describes the source files, NuGet packages, target directories, and filter conditions.

Typical deployment content includes:

- master plugin file.
- Plugin assembly.
- Options configuration file.
- data mapping file.
- Localized resources.
- Certificate or site configuration.

## Pack

After the deployment is completed, you can use `dotnet-pack` to make the release directory into a common Linux package format:

- `.tar.gz`
- `.deb`
- `.rpm`

The packaging phase focuses on installation paths, systemd services, lifecycle scripts, package metadata, and file permissions.

## Automatic Upgrades

Automatic upgrades are supported by the upgrading component in the framework repository and the `dotnet-upgrade` tool. An upgrade package typically contains application release files, manifests, checksums, and deployment executor information.

## Recommended Path

During the development phase, the `terminal` host is preferred for debugging plugins; when jointly debugging Web APIs, the `web/default` host is used; when preparing for deployment, the `daemon` or `web` host is used with deployment, packaging, and upgrade tools.

## Do Not Mix the Three Types of Products

| stage | input | Output and acceptance |
| --- | --- | --- |
| deploy | Host release files, plugin packages and solution configurations | Startable runtime directory to complete a business call |
| Install Packages | Run directory and installation metadata verified | tar/deb/rpm, check paths, services and upgrade/uninstall scripts |
| Upgrade package | New version run file and publishing identity | ZIP/manifest, check discovery, handover and new process health |

The compilation configuration in the deployment parameters, the product identity of the installation package, and the version distribution name of the upgrade package have their own purposes, and the same values should not be applied just because the options have the same name. Persistent data and environment secrets should also not be collected indiscriminately into packages.

Specific operations: [Deploy a Host](../hosting/deployment.md), [Packaging Tool](../tools/packager.md), [Automatic Upgrades](../framework/upgrading.md).
