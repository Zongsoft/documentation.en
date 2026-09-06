---
description: "Locate common issues in plugins, configuration, data, messaging, and delivery by observable phenomena."
icon: circle-question
---

# FAQ

## The Package Is Installed, Why Does the Plugin Not Appear?

The package reference is used for compilation, and the manifest and related products need to be entered into the scan directory at runtime. Check the actual process path first, then `.plugin`, dependency declarations, and DLLs. See [minimal deployment](get-started/deploy-first-plugin.md) for complete steps.

## The Plugin Appears, Why Does the First Call Still Fail?

Services may delay establishing connections, and third-party dependencies may be overridden by other packages during deployment. Check [service resolution](framework/core/services/locating.md), the selected connection and the final DLL version; the plugin list only proves part of the loading phase.

## Why Is There No Change After Modifying `.option`?

Check whether the file matches the application name or plugin manifest base name, environment and host/site; then check whether subsequent configuration sources overwrite this key. Don't default to all components supporting hot updates on the fly. See [Options configuration](references/option-files.md) for details.

## Can the Host Portal Write Business Directly?

Application composition and startup configuration are the responsibility of the host. Business rules are recommended to be placed in plugin services and then called by commands, workers or controllers. This makes it easier to reuse and verify between [Terminal and Web](hosting/hosting.md).

## Why Are the Results Different for `Resolve`, `Find`, and `Locate`?

They are respectively biased towards container registration resolution, service matching, name and provider positioning. Redis in "Connection name @Redis" is a provider alias; other providers may not have corresponding aliases. See [Complete rules](framework/core/services/locating.md).

## Why Is the Query Missing a Navigation Field?

Check mappings and schemas. `*` does not mean full expansion of any level; navigation needs to be declared accordingly. `:20` in navigation is a limited quantity, not a top-level page number. See [Data Schemas](framework/data/schema.md) and [Query](framework/data/querying.md).

## Why Does an Asynchronous Query Report an Error in a Loop?

Obtaining an asynchronous sequence does not mean that the query has been completed, execution and reading may occur during enumeration. Error handling, cancellation and resource lifecycle should cover the enumeration scope. See [Data Access Interfaces](framework/data/data-access.md).

## The Message Publishing Returns Successfully, Why Is the Business Not Completed?

The return value may represent a local send, a protocol publish result, or a Broker persistent admission. It is not a general business confirmation. ZeroMQ may also return `null` when there is no online matching subscription; Kafka's explicit confirmation needs to consider background automatic commit. See [reliable delivery](framework/messaging/reliability.md).

## Why Is the Asynchronous Message Handler Not Awaited?

The current synchronous delegate subscription overload receives `System.Action<Message>`. Passing in the asynchronous lambda will form `async void`. You should implement `IHandler<Message>` instead or inherit the asynchronous handler base class, refer to [Message example](framework/messaging.md).

## The Excel Worksheet Exists, Why Can’t the Import Find the Table?

The ClosedXml extractor looks for real Excel Table and model qualified names, such as `__Sales.User__`. A worksheet name or a normal range cannot replace Table. See [spreadsheet extension](framework/externals/documents.md).

## Why Is the Entry Incorrect After Installation or Upgrade?

Distinguish between display name, service name, assembly name and running application name. The packaging tool uses the name inference entry, and the upgrader uses the actual application identity matching release; the web class library `Zongsoft.Web.dll` is not the host entry. See [Pack](tools/packager.md) and [Upgrade recovery](framework/upgrading/workflow.md).

## Does the Existence of `.deployment` Mean the Upgrade Is Successful?

It indicates that a deployment handover is pending. It is also necessary to confirm the exit of old processes, replacement of files, startup of new processes and business health. Full deployment may clear the data and logs in the application directory, and data storage and recovery must be arranged before deployment. See [Automatic Upgrades](framework/upgrading.md).

## Which Path Should Be Read First?

The first contact starts with [Architecture](overview/architecture.md) and [basic concepts](overview/concepts.md), and then completes [Business plugin](get-started/first-business-plugin.md). If you have existing business needs, go to [Reading navigation](README.md) on the homepage to enter the corresponding data, message, Web or external extension topics.
