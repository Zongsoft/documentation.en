---
description: "Concatenate plugins, data, permissions, files and archives with real source code from the Discussions community forum."
icon: seedling
---

# User Stories


This document uses the [Discussions](https://github.com/Zongsoft/Zongsoft.Discussions) community forum module as the business case. The local source code is located in the same level as the discussions repository. The business code in the page is directly excerpted from this repository. When peripheral code is omitted, the files and methods will be noted; the deployment command is written around this real project, and no additional demo application will be built.

## From the Business Understanding Framework

Discussions organizes sites, forums, topics, posts, private messages, user profiles, and attachments into models. The model describes the data, the service implements the business actions, the web controller adapts the request, and the plugin manifest hosts the integration of these objects. It is a business class library and web plugin, and does not come with a production host, database connection or complete operation and maintenance solution.

| reading task | real entrance | Correspondence guide |
| --- | --- | --- |
| Understand how to assemble business modules | Module.cs、Zongsoft.Discussions.plugin | [Business plugin](get-started/first-business-plugin.md) |
| Query forum topics and paginate | ForumService.GetPinnedThreads | [Queries and Navigation](framework/data/querying.md) |
| Create topics and maintain statistics | ThreadService.OnInsert | [Transactions and Consistency](framework/data/transactions.md) |
| Vote and recount votes | PostService.Upvote、SetPostVotes | [Write Operations](framework/data/writing.md) |
| Constrain site and audit visibility | DataValidator、ThreadFilter、PostFilter | [Data Services](framework/data/services.md) |
| Convert authentication identity | UserChallenger、UserIdentity | [Authentication and Authorization](framework/security/authentication.md) |
| Save long text and attachments | Utility、FileController | [file system](framework/core/io.md) |
| Export user and site information | UserDataTemplateModelProvider、user-list.xlsx | [spreadsheets and templates](framework/externals/documents.md) |

## Business Relationship

Sites are business isolation scopes; forum numbers are assigned within sites, so forum relationships cannot use just ForumId. The topic points to the main post through PostId, and replies and comments are also carried by the Post model. The text of the private message is saved by Message, and the recipient and read status are saved by UserMessage; the "message" here is the business data in the site, and it cannot be judged that the project uses the message queue.

Check also [model](https://github.com/Zongsoft/Zongsoft.Discussions/tree/main/src/Models), [mapping](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.mapping) and [database script](https://github.com/Zongsoft/Zongsoft.Discussions/tree/main/database) when reading the fields. The presence of four SQL scripts does not mean that each database combination has passed operational acceptance.

## Paradigm Boundaries

This, data, schema, options and cancellation in the code belong to the original method context and cannot be directly pasted into the blank Program.cs. First understand its host, identity, mapping, configuration and service dependencies, and then run the corresponding business path. The source code link uses the main branch; when this fix has not yet been submitted, the local discussions working tree shall prevail.

{% hint style="info" %}
💡 The current project does not cover all the capabilities of the framework. When there is no direct use case, use existing projects, tests or samples in the sibling framework repository repository, and clearly mark the source; do not describe the caching, queue, scheduling or AI functions in the framework examples as existing implementations in the forum.
{% endhint %}

## Local Check Sequence

First [Build business library](get-started/first-business-plugin.md), then [Deploy to real host](get-started/deploy-first-plugin.md), then check the database mapping, identity and file directory, and finally verify your isolation environment using the request in the repository docs/http. Do not directly execute the remote address or authentication information saved there.

Archive templates, SQL, and HTTP requests all provide real business context, but should not be brought into production as-is. In particular, database scripts may rebuild objects and should be reviewed and used in temporary databases before execution.
