---
description: "Prepare SDK, tooling, and source build output based on selected tutorials, distinguishing package usage from framework source code development."
icon: list-check
---

# Prerequisites

![Prerequisites](../.gitbook/assets/zongsoft-start-cover.png)

Learning plugin usage allows you to get the framework directly from NuGet without having to build the entire repository first. You only need to prepare adjacent source build output if you need to debug the framework source code, use unreleased local changes, or build an existing hosting.

## Choose a Path

| target | Required content | starting point |
| --- | --- | --- |
| Create a minimal plugin application | Match SDK, deployment tools, available package sources | [first plugin](deploy-first-plugin.md) |
| Debugging frameworks and existing hosts | Git, framework/hosting source code, matching compilation output | [Host Overview](../hosting/hosting.md) |
| Connect to a database or messaging system | Previous content and selected external services | Corresponding driver and connection topics |

## Development Guidelines

Before writing business plugins or Web APIs, developers should read and follow these guidelines:

- [C# Coding Guidelines](https://github.com/Zongsoft/Guidelines/blob/main/zongsoft.csharp.guidelines.md) (in Chinese): follow consistent naming, formatting, and coding conventions when writing and reviewing C# code.
- [REST API Design Guidelines](https://github.com/Zongsoft/Guidelines/blob/main/zongsoft.rest-api.guidelines.md) (in Chinese): follow the conventions for resource names, HTTP methods, status codes, and parameters when designing and implementing HTTP APIs.

Use the latest versions at these links. Read examples alongside the guidelines, and check compliance during development and code review.

## SDK Version

The current root configuration of the hosting repository targets .NET 10; this documentation’s minimal tutorial also uses `net10.0`. Multiple framework class libraries support .NET 8, 9, and 10, but specific projects, examples, and tools need to be based on their own `.csproj`, `Directory.Build.props`, and package dependencies.

{% code title="CheckDotnet.ps1" %}
```powershell
dotnet --info
dotnet --list-sdks
dotnet --list-runtimes
```
{% endcode %}

SDK is used for compilation and runtime is used for execution. Installing a newer runtime does not mean that the machine has all the runtimes required for older targets; web and Windows desktop tools also have their own operating environment requirements.

## Source Directories

When the source code path is required, it is recommended to put the repository in the same parent directory:

{% code title="Workspace.layout" %}
```text
Zongsoft/
	framework/
	discussions/
	hosting/
	tools/
	documentation.en/
```
{% endcode %}

{% code title="CloneRepositories.ps1" %}
```powershell
git clone https://github.com/Zongsoft/framework.git
git clone https://github.com/Zongsoft/hosting.git
git clone https://github.com/Zongsoft/Zongsoft.Discussions.git discussions
git clone https://github.com/Zongsoft/tools.git
git -C framework submodule update --init --recursive
```
{% endcode %}

The framework's OpenTelemetry protocol source uses submodules; corresponding content is required when building related diagnostics projects. By default, hosting refers to the adjacent output of the framework. The existence of the source code directory is not enough. The corresponding configuration and target of the required class library must be compiled first.

## Tools and Optional Environments

See [Install Packages](install.md) for installation steps. The editor can use an IDE that supports .NET. Shell commands should be executed according to the respective syntax of PowerShell or Bash. Line continuation and variable interpolation cannot be mixed.

Only compiling Discussions does not require running external services; complete forum query requires preparing the database, identity and related dependencies according to the real mapping and hosting scheme. MQTT and model servers are not inherent prerequisites for forum business. Prepare dependent services as needed and check endpoint readiness first; read [Container Environments](../hosting/containerization.md) when using Podman.

After the preparation is completed, you should be able to clearly answer: what is the target framework, where does the package come from, where is the output directory, and which program is actually started. Then [Choose a host](hosting.md) and complete your first end-to-end run.
