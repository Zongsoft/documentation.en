---
description: "The Aliyun project's capability scope, deployment products, integration entrance and usage boundaries."
icon: plug
---

# Aliyun

Integrate Alibaba Cloud OSS, messaging, SMS and voice, mobile push and other capabilities. Different services have independent resources, templates and permission conditions, and are configured according to the actual capabilities used.

| Item | value |
| --- | --- |
| source code directory | `externals/aliyun` |
| main bag | `Zongsoft.Externals.Aliyun` |
| Companion theme | [Cloud Services and Callbacks](../cloud.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals aliyun]
nuget:Zongsoft.Externals.Aliyun
```
{% endcode %}

The configuration is located at `/Externals/Aliyun`. General settings select Service Center and Network, named Certificate holds access credentials; here Certificate is the credential model of the adapter. OSS integrates through `zfs.oss` file system.

The callback needs to be deployed separately `Zongsoft.Externals.Aliyun.Gateway`, and the main package will not automatically provide an HTTP callback entry.

## Integration Steps

1. Select a service, configure dedicated resources and corresponding credentials; the object storage first verifies that the information of the existing object is read.
2. SMS, voice and push notifications need to check the template, application and target type, and then verify the platform return value and business receipt.
3. Registers the target handler when receiving a callback and verifies the original request's signature, duplicate notifications, and protocol-required responses.

For specific configuration, calling examples and related basic concepts, see [Cloud Services and Callbacks](../cloud.md).

## Project Boundaries

{% hint style="info" %}
💡 The name in the default callback path is the handler dispatch key. The gateway does not automatically complete signature verification, decryption and replay protection for all services; Aliyun's handler collection assembly also needs to expose the Handlers attribute according to the actual plugin node.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Cloud Services and Callbacks](../cloud.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/aliyun)
