# Document Completion and Source Code Verification Records

This is a translated historical record of the Chinese edition’s documentation and source verification work. It does not describe a new source-code change or a new integration test run performed for the English translation.

This document records the outline, delivery and verification of this round and does not include reader navigation. For the reader entrance, see [SUMMARY.md](SUMMARY.md).

## Final Scope and Baseline

- The English documentation center currently has 176 unique navigation pages; the newly added project classification route retains 29 pages.
- Business cases are preferably taken from discussions; in the absence of direct use cases, existing projects, tests or samples of the framework are used. Hosting and tool operations correspond to the actual solutions of hosting and tools.
- Final checked local submissions: framework 590979a3, discussions 9a3376e, hosting 064f7cb, tools 641f216; discussions also include fixes that have not yet been submitted this time. The source code link uses the main branch, so these fixes should be checked against the local working tree and snippet records before committing.
- Have read AGENTS.md, SKILL.md, and README of the repository and related source code directories, and used GitBook skills and official specifications. The document library only retains Chinese README.md.
- The framework working tree is not modified. Code fixes focused on user-authorized discussions that did not execute database scripts, real HTTP requests, cloud service calls, Cake releases, or NuGet pushes.

## Outline and Delivery Portal

| order | content | Delivery and Check Points |
| --- | --- | --- |
| 1 | Architecture and basic concepts | [Architecture](overview/architecture.md), [concept](overview/concepts.md): dependency direction, host, plugin, module, service and configuration responsibilities |
| 2 | Get Started | [Deployment Discussions](get-started/deploy-first-plugin.md), [Business plugin](get-started/first-business-plugin.md): real hosts, projects and running resources, independent virtual applications are no longer retained |
| 3 | plugin framework | [plugin manifest](framework/plugins/plugin-file.md), [Builtins and Services](framework/plugins/builtins-and-services.md), [Host Integration](framework/plugins/hosting.md): Mounting and parsing process in source code |
| 4 | data engine | [First query](framework/data/quickstart.md), [mapping](framework/data/mapping.md), [Data Services](framework/data/services.md), [affairs](framework/data/transactions.md): forum composite key, text navigation, tenant and review |
| 5 | Web | [controller](framework/web/controllers.md), [data service interface](framework/web/data-services.md), [agreement](framework/web/protocols.md): actual routing, request context and asynchronous service entry |
| 6 | security | [Certification](framework/security/authentication.md), [Identity and Credentials](framework/core/security.md), [Privileges](framework/core/security/privileges.md): True Statement, Identity Transformation and Business Permission Boundaries |
| 7 | diagnostics | [Diagnostics](framework/diagnostics.md), [OTLP](framework/diagnostics/otlp.md): indicators, logs, links, export and receive services |
| 8 | Other modules | [AI Integration](framework/intelligences.md), [Machine Learning](framework/learning.md), [Hardware](framework/hardwares.md), [Reporting](framework/reporting.md): existing calls, dependencies and unfinished implementation |
| 9 | news | [Message Queues](framework/messaging.md), [reliable delivery](framework/messaging/reliability.md): production, subscription, confirmation, storage and protocol differences |
| 10 | Automatic upgrade | [Upgrade process](framework/upgrading/workflow.md): configuration, package discovery, deployment, failure recovery and cleanup scope |
| 11 | external extension | [Read according to ability](framework/externals.md): cache, lock, execution, script, spreadsheet, cloud storage and OPC UA |
| 12 | Hosts and tools | [host](hosting/hosting.md)、[deployer](tools/deployer.md)、[packager](tools/packager.md)、[Upgrader](tools/upgrader.md)、[Regular tools](tools/regular.md) |
| 13 | Core and reference | Collection, configuration, IO, serialization, command, event, filtering, service, communication and other real source code examples; synchronization [package index](references/packages.md), [option format](references/option-files.md), [Terminology](references/glossary.md) and FAQ |
| 14 | Verification | Navigation, relative paths, anchors, code sources, type links, GitBook blocks, XML, line breaks, images, and representative page previews |

The above content has been edited and checked for this round. Functions with external dependencies explicitly retain deployment acceptance conditions and do not use static verification to replace actual integration results.

## Parallel Reading Routes

The original capability classification is retained and cross-linked through independent project entries. SUMMARY.md does not mount the same file repeatedly.

- [data driven](framework/data/drivers.md): MySQL, SQL Server, PostgreSQL, SQLite, DuckDB, ClickHouse, TDengine, InfluxDB, 8 items in total.
- [message item](framework/messaging/projects/README.md): kafka, rabbit, mqtt, zero,.storages, 5 items in total.
- [external projects](framework/externals/projects/README.md): aliyun, amazon, closedxml, etcd, garnet, hangfire, lua, opc, openxml, polly, python, redis, scriban, wechat, a total of 14 items.

externals/velopack and externals/grapecity are no longer included according to the maintainer's request. The corresponding project page, navigation and package index have been removed. The original "Device Protocol and Desktop Update" is adjusted to [OPC UA Device Protocol](framework/externals/integration.md), and the plug icon is set; the spreadsheet topics remain ClosedXml and OpenXml, and the Reporting public contract remains.

## Example Sources and Subsequent Maintenance

[discussions-examples.json](.gitbook/discussions-examples.json) records 297 excerpts, source files and line ranges in 121 pages. Excerpts from C#, XML, HTTP, etc. are checked piece by piece against the local corresponding source code; the installation, build, and deployment commands are operational guidelines written around real projects, and the protocol diagram explains the public contract and does not pretend to be an existing integration of Discussions.

When adjacent source code working trees exist, execute node.gitbook/verify-examples.mjs from this document repository to recheck whether the source record, current source code, and the code in the text are consistent. See [verify-examples.mjs](.gitbook/verify-examples.mjs) for tool source code. After the source code changes, you should first understand the behavior before modifying the text and excerpt records; do not update the line number just to pass the check.

## Illustrations and Reading Styles

- Use imagegen skill and the built-in image_gen tool to regenerate the homepage, entry, plugin, data cover, and add new messages and external extension images, a total of 6 2172 × 724 banners.
- The images are placed in.gitbook/assets, the original 4 SVGs have been replaced, and the text and card resource links have been synchronized. [Generate records](.gitbook/assets/image-generation.json) retains prompt words and file lists.
- Uniform warm white background, low saturation cyan and modular composition; no embedded API text that is difficult to maintain. Exact relationships continue using spreadsheet, body, and Mermaid.
- Improve readability with sections, concept links, code titles, hint blocks, and a healthy dose of 💡 / 🚨; express common type descriptions separately from implementation boundaries.

## Discussions Fix

| question | Fixed behavior |
| --- | --- |
| Missing direct dependency on configuration interface | Explicitly declare Configuration.Abstractions, support default NuGet and local framework references |
| Wrong HTTP path in web package | Pack 5 real request files from docs/http; topic request contains ForumId with nested Post.Content |
| Avatar and photo uploaded | The controller uses the UploadAsync callback to save the path and pass the request to cancel |
| Caller SiteId replaces current site | The query conditions are combined with the current identity site by AND, and the original conditions are retained. |
| Packaging results and asynchronous enumeration are lost before querying | OnFiltered wraps the final result, retaining model type, asynchronous reading, paging events and early release |
| Unreviewed text and inconsistent types | Anonymous/non-author results are desensitized; both clearing and reading the text use embedded types to avoid empty paths or repeated reading of files. |
| Moderator details are lost before authorization. | The original text is associated with the internal weak reference table of the current instance; details will be restored after moderator authorization is completed, and the review status will not be modified. |
| Topic cascading text and direct replies missed review | The two entrances uniformly query the forum rules, and the server determines Approved; the topic is consistent with the text, and is forced into the writing mode. |
| HTTP asynchronous entry bypasses synchronous business hooks | Complete relevant asynchronous paths for 8 services, pass cancellation, use asynchronous accessors and transaction completion interfaces |
| Files left behind after long body failure | Topics, posts, feedback, and private messages are unified to handle content conversion and compensation for new file failures; short text replacement restores embedded logos |
| File name collision and duplicate text conversion | Use random suffixes for messages/feedback; feedback will no longer write the content that has been converted into a path again. |
| The main record of the private message fails and the recipient is still written. | Return directly when the main record has not been written to avoid continuing to create a receiving relationship |
| Repeatedly implement Core’s enumeration capabilities | 5 queries are switched to Core's FirstOrDefault; the filter adaptation layer reuses Pageable.Filter and Enumerable, and the business layer iterative implementation is removed |

Additional review: Cancel the CancellationToken alias in the 7 service files and refer to System.Threading uniformly; explicitly use Models.Thread where the model has the same name. Preserved text conversion and failed file compensation are business rules of Discussions and cannot be overridden by database transactions. New validation covers paging notification subscription/unsubscription, paging suppression status, and asynchronous filtering cancellation and release.

Discussions' Core dependency has been upgraded to 7.59.0. When checking NuGet on 2026-09-06, the latest is still 7.58.0, and the latter's paging filter passes the wrong current element to the callback; local 7.59.0 has been fixed. Currently, the verification is based on the local framework reference. The verification results of the old package path cannot be used to the latest modification before release. The construction method has been synchronized to the Chinese and English README of Discussions and the library preparation environment page.

The regression entry is discussions/test/Zongsoft.Discussions.Regression.csproj, which is a console checker executed using dotnet run, not a dotnet test project. The inspection covers isolation identity, site conditions, synchronous/asynchronous, body inline/external, audit policy, failure compensation, early cancellation, iteration release and uploading empty requests.

There are still different lifecycles for database transactions, object storage and business state. Failure compensation for new files is not a substitute for process crash recovery, old file recycling, or cascading and concurrent acceptance of real databases.

## Known Differences in Framework Source Code

These differences have been explained on the corresponding pages, and the framework has not been modified:

- There is a difference between the current named data provider registration and Core's default accessor resolution contract; Discussions uses module accessors.
- The XSD is inconsistent with some of the loader's defaults; runtime behavior cannot be inferred from just the format definition.
- Kafka automatic commit and automatic site recording affect manual confirmation guarantees; Redis lock lease and fencing are not equivalent to the full consensus protocol, and the demo file counter is not an atomic business commit.
- There are differences between the ZeroMQ README and the public base class's duplicate subscription compatibility check. The text is subject to the current implementation.
- The Scriban optional variable argument is still accessed directly from Count; real variable dictionary calling is used.
- Learning's TextFileLoader.Settings.Populate is different from Core's virtual method signature. Pipeline.Build also has combination defects, and the Web portal has skeleton content; no training tutorial that claims to be complete and runnable is provided.
- Part of the positioning/rendering implementation of Reporting has not yet been completed; the default scheme in the upgrade list and options must also be distinguished based on the actual file.

## Final Verification

- All 176 navigation pages exist and are not repeated. The collection range is consistent with the driver/message/external project directory; the page icons are complete.
- The relative paths and reference anchor points within the entire library pass the check. The GitHub source code paths in the 718 navigation texts correspond to adjacent repositories; it is not claimed that all external websites are reachable online.
- The simple key value of frontmatter, GitBook block pairing, code fence, language and title check are passed; the .NET type abbreviated in the text supplements the official documentation and source code link, and the framework type and attribute name of the same name have been distinguished.
- 297 registration excerpts passed the source file, line range and text consistency check; 38 XML segments passed syntax parsing in the temporary root element and were not equivalent to the database run verification of all mappings.
- Six pictures checked. Home page, data entry, MySQL, Kafka, distributed lock, and data service have been checked for local content rendering and narrow screen data entry; there are no missing pictures or horizontal overflow of the entire page. This preview does not simulate full GitBook custom rendering, and GitBook is not published online.
- Discussions passed 275 offline regression checks with local Core 7.59.0 references; the net8.0, net9.0, and net10.0 builds of the API project all passed with zero errors and zero warnings. Core 7.59.0 is yet to be released by NuGet, and the current default package restore path has not yet been verified.
- After rebuilding the net10.0 output of local Core and Web according to the new README command, 275 regressions passed again. Core itself has 4 existing warnings (obsolete PasswordUtility and unused category parameter), Web has no warnings; the framework working tree has not been modified.
- Local dotnet pack generates a web package, verifies.deploy, web plugin and 5 HTTP files, and rechecks the theme request content in the package. Packaging prompts that the existing package lacks README; the package has not been pushed.
- The text involved in this modification uses CRLF uniformly; the code uses Tab, and cleans two or more consecutive blank lines, and SUMMARY retains the space level. `.cmd` must use CRLF, `.sh` follows `.gitattributes` and uses LF. Documentation and discussions pass git diff --check.

This round does not connect to the real database, Broker, cloud service, model service or device, and does not perform a deployment restart. The operational acceptance of the external integration is guided by the actual project entry, prerequisites and check items given on each page.
