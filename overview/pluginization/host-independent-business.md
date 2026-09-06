---
description: "Keep business operations in services so controllers, commands, and background entry points share the same rules."
icon: arrows-split-up-and-left
---

# Reusing Business Capabilities Across Hosts

The forum's moderation originally runs through a Web page: a moderator opens the interface and clicks Approve. If that capability later needs to serve batch commands, scheduled tasks, or partner systems, the most valuable thing to reuse is not the page or the controller, but the business operation: under what conditions approval is allowed and what gets updated after approval.

Services carry business operations; entry points translate external input into a business call and map the result back to their own output conventions. Zongsoft separates hosts, plugin composition, and business services precisely so the same business rules can be reused by different entry points. The key premise is: **a business service must not bind itself to one entry point's input and output, and must not depend on that entry point in reverse**.

![One business capability, multiple entry points](../../.gitbook/assets/zongsoft-plugin-host-independent-business.png)

_Each entry point supplies identity, business scope, and configuration before calling the same approval service. The batch command and background entry in the illustration are proposed usage, not current forum features. The middle bar is a shared design responsibility, not a new framework gateway._

## Read Responsibilities from a Real Call

[ThreadController](https://github.com/Zongsoft/discussions/blob/main/src/api/Controllers/ThreadController.cs) calls [ThreadService.Approve](https://github.com/Zongsoft/discussions/blob/main/src/Services/ThreadService.cs) and maps the Boolean result to HTTP 204 or 404. The controller is thin: it does not re-implement “only moderators may approve” or “only unapproved threads may be approved”; it only decides that a successful approval returns 204 and anything else returns 404. See [Requests and Data Service APIs](../../framework/web/data-services.md).

The separation leaves a result contract worth noticing: `Approve` returning `false` means no update matched the condition. It does not distinguish a missing thread, an already approved thread, or a caller who is not a moderator. If a new command entry needs finer feedback for its operator, consider extending the business result contract — and mind how much permission-related information it exposes. A command should not guess the failure reason.

## The Dependency Direction Makes Reuse Possible

Entry points and services can be separated only when dependencies point the right way: the business library does not reference the Web library; the Web library references the business library. Discussions' domain assembly keeps its own dependency boundary, and [Discussions.Web](https://github.com/Zongsoft/discussions/tree/main/src/api) only adapts. Business services return domain results and never produce HTTP responses; a terminal command should not simulate an HTTP request just to make a service run.

The same direction constrains metadata: services, mappings, and options needed by domain capabilities ship with the domain plugin, while controller and middleware capabilities rely on the Web host. Writing HTTP semantics into a domain service forces every reusing entry point to provide an HTTP environment.

{% hint style="info" %}
Layering does not require adding an interface for every method. Today's controller uses the concrete data service directly. Introduce an interface with an explicit owner only when cross-module consumers need a stable contract or several implementations must be interchangeable. Maintainability comes from clear responsibility and from avoiding pointless forwarding layers.
{% endhint %}

## A New Entry Point Must Supply Its Own Context

A Web entry point already has an authenticated principal, a request environment, and composed plugins. A command or background entry does not inherit these conditions — especially the question of who the current caller is.

| What must be clear | Where a Web entry point gets it | What a new entry point must arrange |
| --- | --- | --- |
| Caller identity | Authentication and authorization | Establish a trusted execution identity with matching permissions; a background job is not a moderator |
| Business scope | Request parameters and service constraints | State the site, target resources, and allowed scope |
| Configuration and services | Host plugin composition | Deploy domain dependencies, configure connections, and verify service resolution |
| Completion and failure | HTTP response | Define command results, task records, and failure handling |
| Cancellation and retry | Request lifecycle | Use the cancellation the entry point supports and verify repeated-execution semantics |

The rows are design responsibilities, not a claim that every current service method accepts a unified context object. When implementing a new entry point, pass parameters explicitly through the existing API or establish an execution context; if the contract must be extended, treat compatibility as part of the change.

## Place State and Work by Lifetime

“Approval state” may mean a thread selected in the UI, an item already processed by a batch, or an officially approved result in the database. These follow the lifetimes of interaction, task execution, and persistent data respectively, and should not all live on one shared module instance.

A continuously listening entry can be managed by a [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) that starts and stops the loop and calls the service for each message; startup-time composition belongs to an [initializer](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationInitializer.cs). Do not replace lifecycle management by starting long-running tasks in a service constructor. Shared services should not keep the thread identifier or caller identity of “this approval”; call data belongs in an explicit execution scope. See [Service Resolution and Ownership](../../framework/core/services/locating.md).

A proposed batch entry that must resume after interruption should persist its progress in task records and recheck permissions and target state on recovery. Progress kept only in process fields is lost when the process exits, and restoring a memory snapshot cannot undo already committed approvals.

## Verify Reuse at Two Levels

The first level targets business results: an ineligible caller is rejected, repeated approval follows the contract, and content-post approval state stays consistent. With the required configuration, identity, and data dependencies, such checks should be able to call the business service directly — no full HTTP pipeline is needed to verify rules.

The second level targets the entry point itself: how a controller maps the result, how a command obtains identity, and how a background task recovers after interruption. New entry points can share the first level of checks but still need their own adapter checks. Only then can you tell whether a change altered business rules or merely altered how the rules are accessed.

HTTP entry points follow the [REST API Design Guidelines](https://github.com/Zongsoft/Guidelines/blob/main/zongsoft.rest-api.guidelines.md), and C# implementations follow the [C# Coding Guidelines](https://github.com/Zongsoft/Guidelines/blob/main/zongsoft.csharp.guidelines.md). Next, read [Designing Extension Points as Collaboration Contracts](extension-contracts.md) to decide whether a new capability should join via a service call, an extension collection, or an event.

## Further Reading

- Choose the host that carries the capability: terminal, background service, and Web differences in [Host Overview](../../hosting/hosting.md).
- How module services resolve and fall back: [Plugin Application Model](../../framework/plugins/application-model.md).
- Service ownership and scopes: [Service Resolution and Ownership](../../framework/core/services/locating.md).
- Capabilities shared by multiple entry points should first find their boundary: [Finding Module Boundaries Through Business Changes](business-boundaries.md).
