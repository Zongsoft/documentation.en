---
description: "Combine hosts, plugins and environment configurations into runnable directories, and check variables, versions and update boundaries."
icon: truck
---

# Deploy a Host

Host deployment includes host release products, plugin assemblies and manifests, configurations, mappings and other resources. `dotnet publish` mainly prepares host runtime artifacts, and `dotnet deploy` combines plugins according to the deployment list; the output of the two should be merged into the same controlled run directory.

## Confirm the Plan and Variables First

The deployment script of hosting combines the public scheme files under hosts `.deploy` and `.deploy/{scheme}`.

| variable | meaning | confusion |
| --- | --- | --- |
| `scheme` | Deployment plan | Not the operating environment |
| `environment` | Development/test/production and other configuration environments | Not equal to Debug/Release |
| `edition` | Compile output configuration in this deployment tool call | The option with the same name in the upgrade tool represents the version distribution name |
| `framework` | Target framework, such as net10.0 | Need to match existing compilation output |
| `platform/architecture` | Native resources and runtime selection | It is not a necessary value for the machine currently executing the script. |
| `host/site` | Host and site combination selection | The use of terminal/daemon by the terminal is intentional |
| `debug` | Debug switches defined in the scheme | Possibility to choose additional configurations beyond just generating a PDB |

Variables are used by scripts, command lines, and deployment definitions. See [Deployment File Format](../references/deploy-files.md) for details. The target directory should be confirmed before copying the command to avoid accidentally overwriting another host.

## Where Should the Configuration Be Placed?

Apply general settings to the host application options, and plugin-specific settings can be `.option` that matches the base name of the manifest file. Environment variants such as `Zongsoft.Security.development.option`, additional debug variants such as `Zongsoft.Security.development-debug.option`.

See [options file](../references/option-files.md) for specific matching and priority. The same configuration key should be responsible for a clear final configuration as much as possible to avoid relying on the uncommitted coverage order between different plugins.

## Post-deployment Checks

1. Verify host entry, runtime configuration and final assembly version.
2. Check Main, host variants and business lists, check dependency names and target scan directories.
3. Check `.mapping`, SQL, standard libraries, native libraries, templates and other ancillary resources.
4. Check that the configuration has the expected environment, endpoint, and site selected.
5. After startup, perform a business operation to confirm that delayed connections and service calls are normal.

It is recommended to use [Minimal deployment tutorial](../get-started/deploy-first-plugin.md) when learning for the first time. Existing business solutions are executed according to the real parameters of the repository script.

## Partial Update

During development, you can stop the host and copy only the changed DLL, PDB and related resources, but only if the dependencies are confirmed to be compatible. Replacing only the DLL without new manifests, mappings or dependencies will result in "compile successfully but run failed".

Manual copying does not have the semantics of automatically undoing old files. When removing old plugins or splitting an assembly, you should be clear about which old artifacts to clean up and check whether they are still used by other plugins. Formal delivery is better suited to generating a complete catalog from a repeatable deployment manifest.

## Data and Upgrade Boundaries

Environmental secrets, persistent data, and replaceable program files should have clear ownership. The complete program directory is not necessarily suitable for packaging as is: run logs, cache databases, and temporary files may be locked or contain environment data.

When using [Automatic Upgrades](../framework/upgrading.md), additionally check `.deployer`, upgrader plugin, actual application name and persistent data location. After re-executing the deployment script, confirm whether the manually added runtime artifacts are still in the directory.
