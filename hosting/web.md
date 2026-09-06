---
description: "Start the plugin architecture HTTP host, organize the site configuration, and verify the controller and runtime environment."
icon: globe
---

# Web Host

Web hosting hosts controllers, authentication and authorization, and HTTP protocol extensions. The business interface is provided by the plugin, and the host entrance is responsible for establishing the running pipeline and site context. See [Web Fundamentals](../framework/web.md) for frame capabilities.

## Default Site

The current repository actually provides `web/default`. The management end, merchant end, client end, callback gateway, etc. are examples of site divisions that can be established based on business, and are not complete products that already exist.

The current entry is passed in `host=web`, `site=default`, `daemon=zongsoft.web`, redirect the root path to `/Application` and then run the application:

Source: [hosting/web/default/Program.cs](https://github.com/Zongsoft/hosting/blob/main/web/default/Program.cs#L12) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
static void Main(string[] args)
{
	var app = Zongsoft.Web.Application.Web([..args, "host=web", "site=default", "daemon=zongsoft.web"]);

	//如果要启用私有部署模式则打开下行代码注释
	//app.Configuration["Deployment"] = "private";

	app.Map("/", ctx => { ctx.Response.Redirect("/Application"); return Task.CompletedTask; });
	app.Run();
}
```
{% endcode %}

When adding a new site, you should also adjust the `site` and corresponding options in the entry, build/deployment script, and you cannot just copy the directory and then modify the display title.

## Build and Deploy

The project references the output of the adjacent framework by default. The current web project also needs to check its Release reference path. When the plugin reports `Zongsoft.Web` assembly version mismatch, first check the framework output and final DLL, and then redeploy the host.

The runtime directory requires the host's own dependency files, `appsettings.json`, plugin list and resources. Simply copying the business controller DLL is not a substitute for full plugin deployment. See [Deploy Controller Plugins](../framework/web/controllers.md) for a complete example of a stand-alone probe controller.

## Authenticate from HTTP

First confirm the actual listening address from the startup log, and then request `/Application`. `/Modules` and `/Events` can be used to view the corresponding description; access is still affected by the actual deployment and authorization configuration.

{% code title="ProbeApplication.ps1" %}
```powershell
curl.exe --include http://127.0.0.1:8069/Application
```
{% endcode %}

`8069` is a local example port and should be replaced with the actual listening value. For connection failures, 404, 401/403, and 500, prioritize checking the monitoring network, route loading, identity permissions, and server-side exceptions respectively. Do not redeploy all plugins in all situations.

Repository `web/.http` contains the HttpYac request definition, which can also be translated into other HTTP client calls by its method, path, headers, and body. Credentials should be obtained from the current environment, do not copy the default test account or secret to the main example.

## Publishing and Agency

When packaging Linux, the entry name should be `Zongsoft.Hosting.Web` and the service name can be `zongsoft.web`; do not let the generated service point to the class library `Zongsoft.Web.dll`. For the generation rules of service files, see [Packaging Tool](../tools/packager.md).

Reverse proxy environments should check external URLs, forward header trust, HTTPS, request sizes, and timeouts. After enabling OpenAPI or Dashboard, you need to check the corresponding access control; CORS configuration does not replace authentication.

The static file directory prompt caused by empty `wwwroot` does not necessarily affect the pure API service. Do not add irrelevant placeholder services to eliminate the prompt. This should be judged by whether the application actually serves static content.

Related topics: [data service interface](../framework/web/data-services.md), [OpenAPI/gRPC](../framework/web/protocols.md), [Authentication and authorization](../framework/security/authentication.md).

Source references: [Default website](https://github.com/Zongsoft/hosting/tree/main/web/default).
