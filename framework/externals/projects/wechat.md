---
description: "Wechat project's capability scope, deployment products, integration entrance and usage boundaries."
icon: plug
---

# Wechat

Provide WeChat accounts, public platforms, third-party platforms and payment adaptation. Application identity, user identity, platform authorization and merchant permissions need to be handled separately.

| Item | value |
| --- | --- |
| source code directory | `externals/wechat` |
| main bag | `Zongsoft.Externals.Wechat` |
| Companion theme | [Cloud Services and Callbacks](../cloud.md) |

## Deployment and Usage Portal

Append the main package to the existing host's [Deployment checklist](../../../references/deploy-files.md), and retain the plugin list, assembly and ancillary running resources in the package:

{% code title="Application.deploy (append fragment)" %}
```ini
[plugins zongsoft externals wechat]
nuget:Zongsoft.Externals.Wechat
```
{% endcode %}

Configure accounts, certificates and services according to the package options. The main package provides client capabilities, and the HTTP API and callback entry are independent products.

Related products include `Zongsoft.Externals.Wechat.Web` and `Zongsoft.Externals.Wechat.Gateway`, which can be combined according to actual needs.

## Integration Steps

1. First determine the WeChat product used and the identity of the corresponding application or merchant.
2. Check the authorization, token saving and refresh processes, and then verify the required client operations.
3. Register the handler when receiving the callback to verify the signature, decryption, duplicate notification and response content required by the target protocol.

For specific configuration, calling examples and related basic concepts, see [Cloud Services and Callbacks](../cloud.md).

## Project Boundaries

{% hint style="info" %}
💡 Default Gateway's POST dispatch is not equal to complete platform address verification or automatic signature verification. Source verification and idempotent check should be completed before business execution, and the gateway routing name cannot be used as the basis for authentication.
{% endhint %}

## Further Reading

[Browse extensions by project](README.md) · [Cloud Services and Callbacks](../cloud.md) · [source code and project description](https://github.com/Zongsoft/framework/tree/main/externals/wechat)
