---
description: "Filtering and exporting configurations for metrics and tracking are described in terms of framework package options."
icon: stethoscope
---

# Diagnostics

Zongsoft.Diagnostics connects Core diagnostics options to the actual exporter. Discussions does not independently define diagnostics solutions, which are selected and deployed by the host when used; this page uses the framework package configuration as a reference.

## Indicator Export Configuration

Source: [framework/Zongsoft.Diagnostics/src/Zongsoft.Diagnostics.option](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Diagnostics/src/Zongsoft.Diagnostics.option#L6) (excerpt; see source for context).

{% code title="Zongsoft.Diagnostics.option" %}
```xml
<meters>
	<filter>*</filter>

	<exporters>
		<exporter exporter.name="telemetry" driver="telemetry"
		          settings="server=http://localhost:4317;protocol=grpc;processorType=batch;timeout=30s;interval=15s;" />

		<exporter exporter.name="prometheus" driver="prometheus"
		          settings="urls=http://127.0.0.1:9464,http://localhost:9464;path=/metrics;totalSuffix=true;cacheDuration=300ms;" />
	</exporters>
</meters>
```
{% endcode %}

This is the meters section from the original configuration: the wildcard filter selects the source, telemetry points to the local OTLP gRPC sink, and prometheus provides the pull endpoint. The port belongs to this configuration and should not be treated as a fixed port for all hosts. Before running, you should choose the required export method according to your own environment.

## Tracking and Logging Are Not the Same Configuration Object

The same file also has a traces section that configures OTLP and Zipkin. Log output is also organized by [Core log system](core/diagnostics.md); enabling indicator export does not mean that application logs will also be sent to the same target.

## From Business Measurement to Backend

See [Telemetry](core/diagnostics/telemetry.md) for the actual authentication indicators of the framework security module. First confirm that the business actually records measurement values, and then check filtering and exporting; no output may mean that there is no matching source, or it may be a failure of the transmission endpoint, protocol, or backend.

The persistent business statistics such as the number of posts in Discussions are different from the telemetry Counter: the former requires database transaction maintenance, while the latter is used to observe the running process. Don't replace each other.

## Integration Order and Cost

First select a clear signal and source, then set up batches, intervals, timeouts and receivers, then verify failure behavior and resource release. Wildcard sources are suitable for initial verification, and formal deployments should evaluate throughput, tag cardinality, and sensitive attributes.

The self-built receiver reads [OTLP Integration](diagnostics/otlp.md), and the source code is based on [Configurator](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Diagnostics/src/Configuration/Configurator.cs).
