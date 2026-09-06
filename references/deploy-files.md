---
description: "Illustrating resource layout and variables with a real deployment manifest of Discussions area and Web plugin."
icon: file-code
---

# Deployment File Format


.deploy describes how to collect files from a package or local directory and place them into the target layout. Discussions provides separate manifests for domain libraries and web libraries, delivered as NuGet packages.

## The Root Layout of the Domain Package

Source: [src/Zongsoft.Discussions.deploy](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.deploy#L1) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.deploy" %}
```ini
artifacts/Zongsoft.Discussions.plugin
artifacts/Zongsoft.Discussions.option
artifacts/Zongsoft.Discussions.mapping
lib/$(Framework)/Zongsoft.Discussions.*
```
{% endcode %}

In artifacts are manifests, options and mappings; under lib select the target framework assembly according to the Framework variable. There is no fictitious business directory and no automatic database building steps. When there is a lack of products corresponding to the target framework, build or version selection should be resolved first.

## Web Packages and Template Directory

Source: [src/api/Zongsoft.Discussions.Web.deploy](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/api/Zongsoft.Discussions.Web.deploy#L1) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.Web.deploy" %}
```ini
artifacts/Zongsoft.Discussions.Web.plugin
lib/$(Framework)/Zongsoft.Discussions.Web.*

[templates]
artifacts/templates/*.xlsx
```
{% endcode %}

The bracketed paragraph places subsequent files in the templates subdirectory; the source path on the right is still interpreted relative to the package layout. Web templates are collected from docs/templates by project files. The deployment manifest must be checked together with the Pack and PackagePath of csproj, otherwise there may be files in the source code but not in the package.

## Variables Come from the Deployment Process

The Framework is provided by the target parameter of the deployer, not an arbitrary guess of the SDK version at runtime. Other variables, conditional copying, renaming and import instructions belong to the deployment tool capabilities. For complete sources, refer to [deployer](../tools/deployer.md) and framework [Profile analysis](../framework/core/configuration.md). Host command line parameters should not be confused with deployment phase variables.

## Checklist Verification Order

First, go to the local package or release directory, then check whether the manifest reference can be resolved, and finally check the DLL, plugin, option, mapping and templates in the target directory. NuGet package building and pushing are separate operations; Discussions' Cake pack task also includes pushing, so do not call it when doing document validation.

Read the actual delivery path: [Deploy Your First Plugin](../get-started/deploy-first-plugin.md).
