---
description: "Connect the Core security contract, persistent identity permissions, web interface and verification code to establish a complete security call chain."
icon: shield-halved
---

# Security

Zongsoft's security capabilities are divided into public contracts, default persistence implementations, and optional web interfaces. [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) defines models such as identity, credentials, and permissions; Zongsoft.Security implements user, role, member, and permission services and saves them through the data engine; Web and Captcha plugins provide HTTP interfaces and human-computer challenges respectively.

## Let’s First Distinguish Three Issues

**Authentication**confirms who the requester is, such as verifying identity and secrets and issuing credentials.**Authorize**determines whether the subject can perform operations on a resource.**Human-machine challenge** reduces automated abuse, it is neither a proof of identity nor a replacement for authorization.

For example, after a user successfully logs in, he may still not have the permission to delete orders; having the permission to read orders does not mean that he can read orders from other tenants. The complete rule needs to be combined with operation authorization and data scope, see [Authentication and Authorization](security/authentication.md).

## Deployment Components

{% code title="Security.deploy(fragment)" %}
```ini
[plugins zongsoft data]
nuget:Zongsoft.Data

[plugins zongsoft security]
nuget:Zongsoft.Security

[plugins zongsoft security web]
nuget:Zongsoft.Security.Web
```
{% endcode %}

Append these entries to the existing web host, and separately deploy the target database driver, configure the Security connection, initialize the corresponding database structure, and cache the credentials required. Pure backend use does not require the Web child plugin. The deployer skips some Zongsoft transitive dependencies by default, so the deployment plan must be composed explicitly.

Security manifest mounts modules, authenticators, authorizers, and user, role, member, and permission services. Applications typically use them through public security contracts and service APIs, without constructing persistence implementations within controllers or directly modifying permission tables.

## Initialization Sequence

1. Check [Data driven and connected](data/connections.md), use the test database.
2. Prepare [Security database script](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Security/database) against the target database, keeping it consistent with the mapped version.
3. Deploy Security manifests, options, mappings and dependencies, configure credential caching and authentication policies.
4. Verify a successful login, a failed login, an authorized and a denied operation, and then verify renewals and exits.

{% hint style="warning" %}
🚨 The default configuration of the package identity allows password length to be 0 and strength to be None. They are framework defaults, not production policies. Passwords, authentication, credential expiration, and administrative roles should be clear to the application before deployment.
{% endhint %}

## Configuration Entry

The main configurations are located at `/Security/Identity`, `/Security/Authentication`, `/Security/Authorization`. Authentication configuration includes deadlines, failed attempt windows, and scenario deadlines; authorization configuration includes management roles. The configuration file must be entered into the application by pressing [Option matching rules](../references/option-files.md), and you cannot only edit files that have not been loaded.

Verification code independent integration, see [CAPTCHAs and Confirmation Tokens](security/captcha.md). See [Core permissions](core/security/privileges.md) for the basic permission model.

Implementation basis: [security plugin assembly](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Zongsoft.Security.plugin), [Default options](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Zongsoft.Security.option).
