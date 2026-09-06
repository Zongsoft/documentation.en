---
description: "The Amazon project's capability scope, deployment products, integration entry points, and usage boundaries."
icon: plug
---

# Amazon

Integrate Amazon S3 and compatible object stores through the framework file system. Suitable for applications that use Bucket and object Key to manage file resources.

| Item | value |
| --- | --- |
| source code directory | `externals/amazon` |
| main bag | `Zongsoft.Externals.Amazon` |
| Companion theme | [Cloud Services and Callbacks](../cloud.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals amazon]
nuget:Zongsoft.Externals.Amazon
```
{% endcode %}

The connection settings are at `/Externals/Amazon/ConnectionSettings` and the drive key is `amazon.s3`. The file system identifier is `zfs.s3`. For example, assets in `zfs.s3:/assets/manuals/start.pdf` are Buckets.

## Integration Steps

1. Prepare test buckets, connection endpoints, zones, and authorized credentials.
2. Check the existing objects according to the read-only example in the topic, and then verify that the application actually needs to enumerate, read or write.
3. Releases the stream produced by each call and confirms the write result with final object information.

For specific configuration, calling examples and related basic concepts, see [Cloud Services and Callbacks](../cloud.md).

## Project Boundaries

{% hint style="info" %}
💡 Custom endpoints use path addressing. Compatibility with S3 does not mean that all vendors behave the same; directories are just object key prefixes, and the rename, append, and random write semantics of local files should not be directly applied to object storage.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Cloud Services and Callbacks](../cloud.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/amazon)
