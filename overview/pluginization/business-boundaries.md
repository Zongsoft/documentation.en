---
description: Use changes to forum moderation, content, and statistics to decide which capabilities belong together and which dependencies deserve a separate boundary.
icon: cubes
---

# Finding Module Boundaries Through Business Changes

A forum introduces a moderation rule that affects thread state, content visibility, and queries. If implementing it requires coordination across separately owned model, service, and controller libraries, the first question is who owns the rule. Classifying files does not necessarily give business changes a clear owner.

With Zongsoft, start with a complete business operation to find a boundary, then choose assemblies, plugin manifests, and deployment packages to support it. This article uses the existing forum module as evidence. Its decomposition recommendations are design methods, not a directory structure mandated by the framework.

![Forum ownership and a proposed search extension](../../.gitbook/assets/zongsoft-plugin-business-boundaries.png)

_Moderation, content, and statistics share business ownership, supported by composition and delivery metadata. Search indexing is a candidate extension. Containers represent maintenance responsibility, not database or process isolation._

## Trace the Rules Behind a Change

Approving a thread does more than set a Boolean field. [ThreadService.Approve](https://github.com/Zongsoft/discussions/blob/main/src/Services/ThreadService.cs) combines the thread identifier, its unapproved state, and moderator eligibility into an update condition, and also updates the approval state of its content post. Separate owners for thread approval and content approval would have to maintain this operation's semantics together.

Creating a thread also involves content storage and author statistics. The current service performs data insertion and statistics updates inside a transaction, while content-handling logic deals with failures between file storage and database operations. See [Transactions and Consistency](../../framework/data/transactions.md). These relationships offer concrete evidence for boundaries: which data changes together, which failures need joint handling, and which rules need a single explanation.

Review several recent requirements and record the affected rules, data, and entry points. Parts that repeatedly change together and require joint acceptance are candidates for one business boundary. Grouping every type containing “user” into a shared module can instead mix login identity, forum author statistics, and customer records.

## Four Boundaries, Four Decisions

Zongsoft offers different organizational mechanisms at different stages. Understand their responsibilities before asking one split to accomplish too many goals.

| Boundary | Decision it supports | Forum example |
| --- | --- | --- |
| Application module | Business identity, module services, and events | [Module](https://github.com/Zongsoft/discussions/blob/main/src/Module.cs) defines Discussions and its accessor |
| Assembly | Which types compile together and which dependencies they can reference | The domain library and [Discussions.Web](https://github.com/Zongsoft/discussions/tree/main/src/api) compile separately |
| Plugin | Which assemblies load and where objects contribute capabilities | Domain and Web manifests declare their composition separately |
| Deployment configuration | Which versions, settings, and resources ship | `.deploy` files describe artifacts needed in the runtime directory |

There is no automatic one-to-one relationship between an [application module](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/IApplicationModule.cs) and a plugin. The forum's Web manifest depends on its domain plugin; together they provide access to the forum. Separating the Web assembly can constrain protocol dependencies without creating a separate “Web business module.” See [Modules, Services, and Providers](../concepts.md#module-service-provider).

## Ownership Includes Rules and Delivery Resources

A module's maintenance responsibility should cover the models, services, mappings, options, and resources needed to implement its rules. Adding a C# property while omitting the mapping and database scripts can leave a successfully compiled module unusable. Delivery tests whether the boundary is complete.

This does not require one assembly for every file. The forum's [Web deployment file](https://github.com/Zongsoft/discussions/blob/main/src/api/Zongsoft.Discussions.Web.deploy) delivers its controller plugin and spreadsheet templates, while the [domain deployment file](https://github.com/Zongsoft/discussions/blob/main/src/Zongsoft.Discussions.deploy) delivers the domain manifest, options, and mappings. Maintainers need to know how these form a usable capability; separately published files still need compatibility.

Layering remains useful inside a boundary. Services own moderation rules, controllers adapt HTTP, and data access handles persistence. With those responsibilities clear, a moderation change can primarily affect the service and its validation cases. Continue with [Reusing Business Capabilities Across Hosts](host-independent-business.md).

## Test a Proposed Split

Suppose the product needs an external search index for the forum. This is a proposed capability, not an existing forum feature. Its query technology and rebuild process make it a candidate for a separate plugin. First decide acceptable indexing delay, how quickly hidden threads must disappear, how deletion propagates, and how failures recover.

If hiding content must take effect immediately, stale index entries cannot be the sole authority for visibility. The application could verify business state before returning results or implement an update mechanism that meets the required timing. The forum owns visibility rules; the index plugin owns the search implementation. Separating them does not define their agreement.

Conversely, splitting thread approval and content-post approval into separate plugins is questionable without distinct usage scenarios, failure handling, and release schedules. Internal type separation may be sufficient until independent composition provides a real benefit.

## Reinforce Service Domains with Engineering Constraints

Module service containers organize lookup. They cannot prevent code in the same process from referencing another module's implementation or accessing its database. A public-contract-only collaboration policy also needs project-reference review, interface review, and tests. The [Core library](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) carries general framework contracts; the product team should assign ownership for product-specific cross-module contracts instead of placing every shared type in an ever-growing common library.

{% hint style="info" %}
Boundaries can change as understanding improves. A short-lived application with few rules and one deployment configuration may benefit from a simple structure. Add boundaries when recurring changes, team coordination, or separate delivery create a concrete need.
{% endhint %}

When reviewing a candidate module, describe its rules, public entry points, behavior when dependencies are missing, and the artifacts that must be validated together when replacing it. A list of directory names alone needs further work. The next article examines [reusing these entry points across hosts](host-independent-business.md).

## Further Reading

Elux's [Micro Modules](https://github.com/hiisea/elux/blob/main/docs/designed/micro-module.md) and its author's [exploration of business modularity](https://www.cnblogs.com/hiisea/p/16624472.html) discuss organizing code and resources around business capabilities. This article develops the topic through forum moderation, Zongsoft module services, and plugin delivery. The frameworks' runtime mechanisms should be understood separately. The linked Elux articles are in Chinese.
