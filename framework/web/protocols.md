---
description: "Enable OpenAPI/Scalar and gRPC for plugin web applications, distinguishing between document generation, service registration, and actual protocol endpoints."
icon: network-wired
---

# OpenAPI and GRPC

OpenAPI describes the HTTP interface for easy browsing, debugging, and client generation; gRPC calls services according to protocol definitions, usually using Protocol Buffers. The two are different integration methods, and are also provided by optional plugins. They do not need to be installed for ordinary MVC applications.

{% tabs %}
{% tab title="OpenAPI and Scalar" %}
## Deploy Documentation Plugin

Append to the existing web host deployment manifest:

{% code title="OpenAPI.deploy(fragment)" %}
```ini
[plugins zongsoft web openapi]
nuget:Zongsoft.Web.OpenApi
```
{% endcode %}

Preserve in-package manifests, options, and dependencies. The initializer registers the document endpoint and Scalar interface. The default endpoints are `/openapi/v1.json`, `/openapi/v1.yaml` (yml is also accepted), and `/scalar`.

The document is generated based on the discovered controller and operand data; attribute routing directly participates in the generation, and conventional routing also requires the actual running of the routing table. There is no real mapped endpoint, and it cannot be turned into a callable interface by writing a document configuration.

## Configuration and Verification

The configuration section is `/web/openapi` and can declare the server, environment, authentication scheme and public request headers. The authentication scheme appears in the document and only describes how the client should carry the certificate. It does not automatically install authentication or authorization on the server.

First open the JSON document, confirm the business path, method and parameter source, and then use Scalar to initiate a real request. When encountering inconsistencies between documents and routes, check attribute routes and controller descriptors without modifying the business logic first.

{% hint style="warning" %}
🚨 The current initializer enables Scalar authentication information persistence and generates a sample Credential value; this random value is not a valid server-issued login credential. Real credentials should not be saved on shared devices, and the visibility of documents and management interfaces should be limited by the application.
{% endhint %}
{% endtab %}

{% tab title="gRPC" %}
## Deploy Protocol Host Extension

{% code title="Grpc.deploy(fragment)" %}
```ini
[plugins zongsoft web grpc]
nuget:Zongsoft.Web.Grpc
```
{% endcode %}

This plugin registers the gRPC server and reflection service and does not define the `.proto` of the application. The business needs to provide the generated service base class implementation, register the specific type through the framework service system, and assign the `gRPC` label.

Discussions There is currently no gRPC service implementation. The framework diagnostics protocol service registers OTLP metric receivers through the static Metrics member of Listener:

Source: [framework/Zongsoft.Diagnostics/protocols/server/src/Listener.Metrics.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Diagnostics/protocols/server/src/Listener.Metrics.cs#L51) (excerpt; see source for context).

{% code title="Listener.Metrics.cs" %}
```csharp
[Service(Tags = "gRPC", Members = nameof(Metrics))]
partial class Listener
{
	#region 单例字段
	public static readonly MetricsProcessor Metrics = new();
```
{% endcode %}

MetricsProcessor inherits MetricsService.MetricsServiceBase in the same file and implements the Export method. Members here means registering static members; it and direct registration service type are the two entrances supported by the service system. See [OTLP Integration](../diagnostics/otlp.md) for the complete call chain.

The initializer reads the type under the tag and maps the service. Merely inheriting to generate a base class or simply copying a DLL is not enough to accomplish endpoint discovery.

## Agreement and Cancellation

Hosts and proxies must support the required HTTP/2, TLS, message sizes, and deadlines. Service implementations should pass call cancellation to downstream operations; a client timeout does not mean that the performed write operation is undone. The actual call should be made using a client that matches the protocol, gRPC methods cannot be authenticated with normal JSON requests.

{% hint style="warning" %}
🚨 The current initializer also maps gRPC reflection, which can expose service metadata. The access policy should be clearly defined during deployment, and the protocol management plane should not be made public by default.
{% endhint %}
{% endtab %}
{% endtabs %}

Implementation basis: [OpenAPI initialization](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Web/openapi/WebInitializer.cs), [Documentation endpoint](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Web/openapi/WebExtension.cs), [gRPC initialization](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Web/grpc/GrpcInitializer.cs). For an example of OTLP protocol integration, see [diagnostics protocol](../diagnostics/otlp.md).
