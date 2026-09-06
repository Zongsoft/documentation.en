---
description: "Organize ASP .NET Core applications with plugins to connect controllers, data services, request pipelines, and protocol extensions."
icon: globe
---

# Web Fundamentals

Zongsoft.Web provides general controller, binding, formatting, routing, credential authentication and file access capabilities; Zongsoft.Plugins.Web integrates these capabilities into the plugin host and is responsible for Web part discovery and application lifecycle. The business controller is placed in the plugin, and the host is responsible for hosting and assembly.

## First Understand What the Request Goes Through

A request is first processed by the host middleware, and then matched with the controller and operation; model binding converts the path, query, request header and request body into parameters, the controller calls the business service, and the formatter outputs the response. Authentication establishes the identity of the caller, and authorization determines whether an operation is allowed. Both have their own responsibilities.

The plugin web portal will initialize the application context and application initializer, then add CORS, localization, method coverage, routing, authentication, authorization, response compression and static files, and finally map the controller and Hub. Check relative order when extending middleware rather than reinstalling the entire pipeline.

## Choose How to Use It

Reference Zongsoft.Web when you only need the MVC auxiliary capabilities of the framework; use the Plugins.Web host when you need to discover the business controller through the manifest. Ready-made launcher see [Web Host](../hosting/web.md).

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

The project needs to reference `Zongsoft.Plugins.Web`, and the runtime directory also needs a basic plugin list and business plugins. This entry itself does not generate a business API or database structure.

## Learning Path

- [Deploy Controller Plugins](web/controllers.md): From class library, manifest to one HTTP request.
- [Requests and Data Service APIs](web/data-services.md): Connect the data service to the CRUD interface and understand binding, paging and capability switching.
- [OpenAPI and gRPC](web/protocols.md): View interface documentation, register protocol services and check endpoints.
- [Security](security.md): Configure credentials, authorization and verification codes.

## Request Status and Shared Services

Web application context can bridge the current request body and session. After the request ends, you cannot continue to rely on the objects in the request; the background task should explicitly receive the required business data instead of retaining the request context. Common plugin services registered through service features default to shared instances, see [Service ownership](core/services/locating.md) for details.

{% hint style="warning" %}
🚨 Current plugin hosts include a permissive default CORS policy. Before the business goes online, it should be adjusted according to the actual source and credential method; the presence of authentication and authorization middleware does not mean that all endpoints are automatically protected. Management endpoints such as plugins, modules, files, and diagnostics should also be included in the access policy.
{% endhint %}

## Troubleshooting Ideas

First confirm that the host is listening and the network is reachable, then confirm that the plugin is loaded, the assembly is added as a web part, the routing template matches, and finally the service resolution and business dependencies are checked. 404 should usually be investigated through routing and discovery; authentication failure and service denial should be investigated along security and service links.

Implementation basis: [Web portal](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins.Web/src/Application.cs), [Web builder](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins.Web/src/WebApplicationBuilder.cs), [Web basic source code](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Web/src).
