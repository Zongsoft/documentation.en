---
description: "Locate operational problems layer by layer by process, plugin, configuration, service and business call."
icon: bug
---

# Run and Debug

Debugging should be against the actual runtime directory. The source code has been modified, the project has been compiled and the host has loaded the new version. First confirm the process path and final DLL, and then analyze the business behavior.

## Start with the Smallest Observable Result

After completing [Discussions Deployment](deploy-first-plugin.md), first check that the web host starts and the two business manifests are loaded, and then use the forum read-only request in discussions/docs/http/forum.http. Confirm that the DLL/PDB in the run directory is consistent with the local build.

The web scenario first checks the listening address and `/Application`, and then verifies [Business controller](../framework/web/controllers.md) of Discussions. When the connection fails, do not check the controller first; when a 404 error occurs, do not modify the database first.

## Layered Troubleshooting

| level | Check content | Common causes |
| --- | --- | --- |
| process | Entry path, runtime, exit log | Wrong DLL started, runtime missing |
| plugin | Scan directories, manifests and dependencies | Only copy DLL, dependency names are inconsistent |
| Configuration | File base name, environment, host/site | Modified unmatched options file |
| service | Register contracts, aliases, modules/providers | Resolve/Find/Locate usage confusion |
| first call | External address, permissions, dependent DLL | Delayed connection failure, transitive dependency override |
| business results | Data, transactions, confirmations and status | Query shape, idempotent or permission scope does not match |

## Breakpoints and Symbols

Attach a debugger to the actual host process and ensure that DLL, PDB and source code versions are consistent. After modifying the plugin, stop the host, copy the corresponding product, and then restart; the framework does not promise that any loaded assembly can be hot-replaced immediately through file overwriting.

If the host uses a local framework reference, it also needs to match Debug/Release and the target framework. Simply rebuilding a project without its dependencies may result in symbol mismatches or missing methods on the first call.

## Configuration and Logs

Record the application name, content root directory, environment and plugin path, and then check [options file match](../references/option-files.md). When configuring diagnostics, only necessary keys and non-secret information are output, avoiding printing the complete connection string.

Logs should preserve exception types and call chains. Errors in asynchronous query, streaming response or message processing may occur in subsequent enumerations/callbacks, and the error handling scope needs to cover the real execution phase.

## Create Reproducible Steps

Save desensitized input, running versions, configuration differences, and expected/actual results. First check the process and plugin assembly, then execute the forum read-only query, and then add external dependencies and concurrency. Use independent test data when it comes to writing, and clean objects explicitly.

Further reading: [Host deployment](../hosting/deployment.md), [service resolution](../framework/core/services/locating.md), [FAQ](../faq.md).
