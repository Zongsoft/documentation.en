---
description: Introduce plugins through verifiable business slices, then manage release combinations, data migration, and the tradeoffs of separate processes.
icon: route
---

# From Incremental Change to Verified Plugin Delivery

A running business system rarely has the option to suspend all requirements for a redesign. When introducing Zongsoft plugins, start with a business path whose responsibility is clear, verify its services, composition, and deployment, then expand gradually. Each step should leave a runnable, observable version with a known response to failure.

This article presents an incremental design method for existing systems seeking clearer module ownership or configurable product capabilities. The forum's file organization is an implementation reference; the migration steps do not describe an ongoing forum migration.

## Choose a Slice with Independent Acceptance Criteria

A useful first slice has explicit input and output, a traceable call chain, and reasonably clear data ownership and dependencies. Exporting, content queries, or a stable business operation can be candidates, depending on the system. A critical write spanning several subsystems usually needs more consistency design. Complexity alone does not make it a good first experiment.

For example, the forum's existing [template model provider](https://github.com/Zongsoft/discussions/blob/main/src/Features/Archiving/UserDataTemplateModelProvider.cs) retrieves data through the user service. A similar capability could retain its entry point and rules while organizing template adaptation and resources as a plugin. Validate results, filtering, and permissions. An export being read-only does not justify bypassing data-scope restrictions.

Before changing structure, establish observable behavior: results for given inputs, permitted identities, failure responses, and approximate resource consumption. Compare each step with this baseline to detect accidental business changes.

## Leave a Runnable Result at Every Step

| Stage | Main change | Evidence needed to proceed |
| --- | --- | --- |
| Extract a business entry point | Existing entry calls a service with explicit responsibility | Original results, permissions, and failure behavior remain verifiable |
| Declare plugin composition | Add manifests, dependencies, and required extensions | Target host loads the plugin and resolves its services |
| Define delivery artifacts | Write deployment rules for configuration and resources | A clean runtime directory completes a real business call |
| Switch the call path | New implementation takes responsibility for a defined scope | Errors, latency, and differences between implementations are observable |
| Retire the old implementation | Remove duplicate entries and obsolete dependencies | Callers have migrated and runtime evidence shows the old path is unused |

Switching is a release decision that may use application routing, configuration, or deployment batches. It does not imply automatic live replacement of loaded assemblies. Choose the mechanism for the target host and delivery setup.

When comparing old and new implementations in parallel, start with side-effect-free reads whose results can be compared. Do not execute both versions of a write merely to compare outcomes. Duplicate notifications, deductions, or files can create business effects that rolling back an assembly cannot undo.

## Validate the Release Combination

![Select versions, assemble artifacts, and verify operation alongside data compatibility](../../.gitbook/assets/zongsoft-plugin-evolutionary-delivery.png)

_After selecting hosts and plugin versions, deliver configuration, mappings, and resources, then verify startup, business calls, and recovery. Data compatibility and migration need their own design; file rollback cannot replace them._

The forum's [domain deployment file](https://github.com/Zongsoft/discussions/blob/main/src/Zongsoft.Discussions.deploy) includes manifest, option, mapping, and assembly rules. Its [Web deployment file](https://github.com/Zongsoft/discussions/blob/main/src/api/Zongsoft.Discussions.Web.deploy) adds the Web manifest and templates. A usable delivery unit must include the metadata and resources required at runtime.

For a product with multiple customer configurations, record the host, plugin versions, drivers, configuration requirements, and migration steps for each supported combination. Release validation should cover the actual minimum configuration and important extension combinations, distinguishing a missing optional plugin from a missing required dependency. There is no need to promise every theoretical permutation of plugins as a supported product.

Publishing a plugin package independently means it can be produced and managed separately. Whether that version can run still depends on compatible hosts, shared dependencies, extension contracts, and data structures. Renaming plugin directories cannot resolve shared-assembly version conflicts within one process.

## Data Changes Set the Recovery Boundary

Rolling back program files can leave databases and content files in a changed state. Before release, determine whether old code tolerates a new mapping field, can read a converted content format, and can cope with external notifications already sent by the new version.

Where appropriate, add compatible fields first, migrate data gradually, switch reads and writes, and remove old structures last. For changes incompatible with old versions, specify a recovery plan and maintenance window. Deployment tools deliver artifacts; they cannot infer data migrations or business compensation. See [Deployment Model](../deployment.md) and [Upgrade Workflow](../../framework/upgrading/workflow.md).

## Decide When a Separate Process Is Worthwhile

Plugins organize code and runtime composition. Process separation introduces fault and communication boundaries. The mechanisms can be combined according to actual needs.

| Requirement | First option to evaluate | Additional cost or limit |
| --- | --- | --- |
| Different products select different capabilities | Combine plugins through deployment configurations | Manage supported version combinations |
| Different entry points share business rules | Separate business services from host adapters | Supply identity, configuration, and runtime dependencies |
| A workload needs independent scaling or fault isolation | Run the capability in a separate host process | Add protocols, timeouts, observability, and recovery |
| Untrusted extensions require permission isolation | Establish a security boundary in deployment and runtime infrastructure | In-process plugins do not provide that guarantee |

Moving a local call onto a network changes failure semantics. A timeout can occur after the remote side has already committed a write. Idempotency, retries, data ownership, and outcome queries need explicit agreement; replacing an interface implementation with a remote client is not sufficient. A separate process is not a mandatory destination for every plugin.

## Judge the Design Through a Release

After the first slice, check whether the next requirement stays mainly within its business area, the runtime directory can be assembled repeatedly, failures can be traced to calls or composition, and the release owner can explain recovery. Those outcomes reveal more than the number of new projects.

Continue with [Your First Business Plugin](../../get-started/first-business-plugin.md), [Deploy Your First Plugin](../../get-started/deploy-first-plugin.md), and [Run and Debug](../../get-started/run-and-debug.md). If ownership is still unclear, revisit [Business Boundaries](business-boundaries.md).

## Further Reading

The Elux author's [article on decomposing large frontend applications](https://www.cnblogs.com/hiisea/p/16914890.html) and [Micro Module Design](https://github.com/hiisea/elux/blob/main/docs/designed/micro-module.md), both in Chinese, provide a starting point for discussing modular evolution. This article develops release combinations, database compatibility, and host-process tradeoffs for Zongsoft server applications. Frontend module loading is not treated as a plugin upgrade guarantee.
