---
description: "Use an interactive terminal to verify plugins, commands, and backend services, and distinguish between running identities and deployment plans."
icon: terminal
---

# Terminal Host

The terminal host is suitable for observing plugin loading, calling business commands and reproducing background tasks. It preserves command interaction, so it is easier to see input, output, and error context than directly installing the business as a system service.

## Start the Portal

The current key calls of `hosting/terminal/Program.cs` are as follows:

Source: [hosting/terminal/Program.cs](https://github.com/Zongsoft/hosting/blob/main/terminal/Program.cs#L9) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
static void Main(string[] args)
{
	Zongsoft.Plugins.Hosting.Application
		.Terminal("zongsoft.terminal", [.. args, "host=terminal", "site=daemon"])
		.Run();
}
```
{% endcode %}

`host=terminal` identifies the interactive host, and `site=daemon` enables it to combine the configuration and plugins of the background scene. Do not change both to the same value for the sake of "unification of names", existing deployment manifests will use them to select different resources.

## Start with a Minimal App

If there is no business deployment plan yet, complete [Deploy Your First Plugin](../get-started/deploy-first-plugin.md) first. This tutorial is based on Discussions and existing hosts, and is suitable for learning the relationship between Main, Terminal and command plugins.

When using an existing terminal from the hosting repository, follow the root directory instructions to prepare the framework output before checking `terminal/.deploy`, scenario manifest, and `deploy.cmd`. The script combines host compilation, plugin deployment and solution selection. `dotnet build` cannot be regarded as the entire deployment work.

The deployed directory can be launched through the corresponding executable file or DLL; execute from the run directory:

{% code title="RunTerminal.ps1" %}
```powershell
dotnet ./Zongsoft.Hosting.Terminal.dll
```
{% endcode %}

## Verify Command

In the terminal, first execute `help` to view the actual command tree, `plugin.list` to view the deployed plugins, and then execute the business command. When the command does not exist, first check the builtin path, command plugin and dependencies; when the command exists but the call fails, then check the service resolution and business configuration.

[The first business plugin](../get-started/first-business-plugin.md) is mounted and deployed from the Discussions module, data service and plugin; for the actual implementation of the site message command, see [command model](../framework/core/components/commands.md). The existence of the command class does not mean that it has been mounted to the current terminal. Please refer to the actual plugin tree.

## Debugging and Exiting

When attaching the debugger, the running DLL, PDB, and source code versions should match; after modifying the source code, it only rebuilds but does not copy the output, and breakpoints may still correspond to the old implementation. Stop the host before manually replacing files, and keep the target directory and deployment version records.

To exit, use `exit -yes`. If it stops slowly, check whether the [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) is still waiting for external requests, message processing, or unresponsive cancellation. Do not change all background tasks to force exit first.

The auto-upgrade test must match the release with the actual application name `zongsoft.terminal` and check that `.deployer` is still in the deployment directory. See [upgrade integration](../framework/upgrading/workflow.md) for details.

Source references: [terminal project](https://github.com/Zongsoft/hosting/tree/main/terminal).
