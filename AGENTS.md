# Zongsoft English Documentation Collaboration Rules

This repository contains the English GitBook documentation for the Zongsoft framework, hosts, and toolchain. Follow these project rules together with the official GitBook skill:

https://gitbook.com/docs/skill.md

## Basic Constraints

- Use CRLF line endings for text files. `.cmd` files must use CRLF; `.sh` files use LF as specified by `.gitattributes`.
- Indent code with tabs. Do not use two or more consecutive blank lines in code. Indent navigation levels in `SUMMARY.md` with spaces.
- Do not modify unrelated files, reorganize entire existing documents, or make unrelated bulk style changes.
- This is an independent English documentation repository. Other projects' bilingual README conventions do not apply. Keep only an English `README.md` at the root and in topic directories. Do not create README files in other languages or add language selectors. Write prose in English; preserve terminology, package names, type names, commands, and filenames.
- `SUMMARY.md` and internal links must point to the corresponding `README.md`. When linking to bilingual documentation in other repositories, preserve the filenames actually used there.
- Write for Zongsoft users: explain how and when to use a feature and what to watch for. Avoid turning documentation into line-by-line source commentary.

## GitBook Authoring Rules

- Before editing an existing page, read `SUMMARY.md` to understand its navigation position, sibling names, and relative links.
- Preserve valid GitBook Markdown, including frontmatter, custom blocks, relative links, and asset references.
- Add a `description` and an appropriate `icon` in frontmatter when creating an ordinary page.
- Update `SUMMARY.md` when adding pages. Never list the same Markdown file twice.
- Use relative internal links, such as `[Deployment Tool](tools/deployer.md)`. Update references when moving files.
- Put images and downloadable assets in `.gitbook/assets/`, using paths relative to each page.
- Use GitBook card tables for navigation and topic entrances. Prefer paragraphs, lists, hints, tabs, steppers, and content references for explanations.
- Where practical, wrap code blocks in GitBook code blocks with meaningful titles.

{% code title="GitBookCodeBlock.md" %}
````markdown
{% code title="Program.cs" %}
```csharp
builder.Services.AddHostedService<Worker>();
```
{% endcode %}
````
{% endcode %}

## .NET Type Links

When prose uses an inline-code .NET class, namespace, interface, struct, enum, or other built-in type whose name does not start with `System.`, add links to its official documentation and source.dot.net source.

- When Microsoft Learn documentation is available, link the type name to Learn and follow it with an italicized Source link.
- Otherwise, link the type name directly to source.dot.net.
- Links are optional for foundational names beginning with `System.`, but may be useful when the type is central to the page.
- Prefer related pages in this repository for Zongsoft types. When source links are needed, use the corresponding GitHub file or directory.
- Prefer official documentation for third-party types, with a GitHub source link where appropriate.
- These rules apply mainly to inline code in prose; do not link every type inside code blocks.

{% code title="DotnetTypeLinks.md" %}
```markdown
[`Microsoft.Extensions.Caching.Memory.MemoryCache`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.memorycache) _[Source](https://source.dot.net/#Microsoft.Extensions.Caching.Memory/MemoryCache.cs)_

[`IChangeToken`](https://source.dot.net/#Microsoft.Extensions.Primitives/IChangeToken.cs)
```
{% endcode %}

## Content Scope

- Cover the Zongsoft framework, plugin framework, data engine, hosts, tools, and reference formats. Add content to existing sections instead of creating a competing classification.
- Overview pages explain design goals and boundaries; getting-started pages provide executable paths; framework guides explain abstractions and typical usage; reference pages provide formats, package indexes, and terminology.
- For complex types, component models, extension mechanisms, and design patterns, explain design principles and intent as well as scenarios, prerequisites, scope, and limitations.
- .NET SDK 8, 9, and 10 may all occur. Verify requirements against the relevant project files, package documentation, or source; do not state unverified version requirements.
- Connect deployment, plugins, mappings, options, and drivers to existing pages under `framework/plugins/`, `framework/data/`, `hosting/`, `tools/`, and `references/`.
- Keep examples short and focused on the API shape. Use tabs for nesting in C#, XML, YAML, and text directory examples.
- Use meaningful example titles, such as `SelectUsers.cs`, `Zongsoft.Data.plugin`, and `samples.option`.
- Use GitBook hints for boundaries: `info` for notes, `warning` for risks and limitations, and `danger` only for destructive or irreversible operations.
- Use tables for type relationships, package indexes, and configuration matrices; cards for navigation; tabs for alternatives; and steppers for ordered tutorials.
- Do not present behavior as fact without confirmation in source, packages, or existing documentation. Mark inference with wording such as “usually,” “recommended,” or “depends on configuration.”

## Validation Checklist

- Frontmatter is valid YAML, and GitBook blocks have matching closing tags.
- `SUMMARY.md` matches added, moved, and removed pages.
- Relative links and asset paths resolve from the referring page.
- Code fences have language identifiers, and major examples have code titles.
- Inline .NET type links follow these rules, especially Microsoft extension types whose names do not start with `System.`.
- Preserve existing user changes outside the authorized task.
- Keep verbatim source excerpts consistent with `.gitbook/discussions-examples.json`; their original comments and literals may remain in Chinese.
