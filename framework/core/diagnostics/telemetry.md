---
description: "Understand meters, gauges, labels and exports from real authentication metrics from the framework security module."
icon: chart-line
---

# Telemetry

Telemetry metrics summarize multiple operations into an observable quantity or distribution, while tracing records the boundaries a call passes through. Discussions has not yet defined its own metric group; the framework security module it relies on already records counts for the authentication process, and this page uses that implementation as an example.

## Create Module Meter

Source: [framework/Zongsoft.Security/src/Module.Meter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Module.Meter.cs#L61) (excerpt; see source for context).

{% code title="Module.Meter.cs" %}
```csharp
internal Diagnostor()
{
	#if NET8_0_OR_GREATER
	_meter = Current.Services.ResolveRequired<IMeterFactory>().Create(NAME);
	#else
	_meter = new Meter(NAME, Current.Version.ToString());
	#endif

	this.Authentication = new(_meter);
}
```
{% endcode %}

.NET 8 and above obtain the IMeterFactory from the module container; earlier targets create the Meter directly. The meter name comes from the security module constant, and the lifecycle follows the module diagnostics meter. It does not create a new one every time you log in.

## Define Certification Instrument

Source: [framework/Zongsoft.Security/src/Module.Meter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Module.Meter.cs#L74) (excerpt; see source for context).

{% code title="Module.Meter.cs" %}
```csharp
	public sealed class AuthenticationMeter(Meter meter)
	{
		#region 常量定义
		private const string METER = nameof(Diagnostor.Authentication);
		#endregion

		#region 公共字段
		public readonly Counter<long> Authenticated = meter.CreateCounter<long>($"{METER}.{nameof(Authenticated)}");
		public readonly Counter<long> Authenticating = meter.CreateCounter<long>($"{METER}.{nameof(Authenticating)}");
		#endregion
	}
}
```
{% endcode %}

Authenticated and Authenticating are two Counters, used at different stages. Filtering, dashboards and alarms can be configured only after the instrument name is stable; the name in the protocol cannot be changed just because the Chinese display name is the same.

## Record Measurements During Real Events

Source: [framework/Zongsoft.Security/src/Module.Events.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Module.Events.cs#L114) (excerpt; see source for context).

{% code title="Module.Events.cs" %}
```csharp
private static bool OnAuthenticating(Privileges.AuthenticatingEventArgs args)
{
	Current.Meter.Authentication.Authenticating.Add(1,
		new KeyValuePair<string, object>(nameof(args.Scheme), args.Scheme),
		new KeyValuePair<string, object>(nameof(args.Scenario), args.Scenario));

	return true;
}
```
{% endcode %}

The measurement is an authentication attempt and the label distinguishes Scheme from Scenario. Tags should have a controlled scope; using each user number or request number as a metric tag will result in high cardinality, and individual traces are better suited to logging or tracing.

## Acquisition and Export Are Subsequent Steps

Creating a Meter and recording a Counter does not mean that the external backend has received the data. You also need to select sources, configure exporters and transport endpoints, and actually produce measurements. See [diagnostics extension](../../diagnostics.md) for the actual configuration.

| signal | Questions suitable to be answered |
| --- | --- |
| Log | What happened in a certain operation and what is the exception call chain? |
| indicator | How many times in a period of time, what is the distribution, and whether there is abnormal growth |
| Track | Which components does a request go through and how long does each period take? |

The three can be related, but the complete business process cannot be deduced from just one indicator count. To monitor posts or file storage, if you want to join Discussions, you should first determine the business success point, failure classification, and transaction boundaries. You cannot directly rename the security module's counts to forum statistics.

## Validation Scope

Verify meter creation, event firing, tag values, filter matches, and export results; then check for cancellations, backend unavailability, and process exits. When using a self-built OTLP server, you need to check the conversion restrictions, see [protocol integration](../../diagnostics/otlp.md).
