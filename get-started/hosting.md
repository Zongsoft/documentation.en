---
description: "Choose the appropriate Zongsoft host based on debugging, service running, and Web API scenarios."
icon: server
---

# Choose a Host

Zongsoft's host is located in [`Zongsoft/hosting`](https://github.com/Zongsoft/hosting) repository. The host is only responsible for starting the runtime and hosting plugins, and should not write business code directly.

## Host Type

### Terminal Host

The terminal host runs through a console and is suitable for interactive debugging, command line reproduction, and observing the behavior of plugin-based applications.

source code directory:

```text
hosting/terminal
```

### Background Service Host

The background service host is suitable for deployment as a Windows Service or Linux systemd service for long-running background applications.

source code directory:

```text
hosting/daemon
```

### Web Host

Web hosting is based on ASP .NET and is suitable for Web API, authentication and authorization, HTTP pipeline, site configuration and interface debugging.

source code directory:

```text
hosting/web/default
```

## How to Choose

When developing and debugging plugins, give priority to using the terminal host; when you need to verify the HTTP interface, use the web host; when preparing to deploy as a background service, use the background service host.

## Next Steps

After selecting the host, continue reading [Deploy Your First Plugin](deploy-first-plugin.md) to learn how to deploy the plugin to the host's `plugins/` directory.

## Reuse Conditions

Shared business contracts can be reused in different hosts, and controllers, middleware and interactive commands require corresponding host capabilities. When building business services, try to put protocol adaptation at the edge, so that terminal commands and HTTP controllers call the same service.

Existing hosting combines multiple infrastructures by default. If you are learning for the first time, you can use the independent minimal host in the tutorial. When you need a ready-made running solution, read the construction and deployment instructions of [Terminal](../hosting/terminal.md), [Background service](../hosting/daemon.md) and [Web](../hosting/web.md) respectively.
