---
description: "Host long-term background tasks, correctly distinguishing between host lifecycle integration, service installation and business readiness."
icon: gears
---

# Background Service Host

The background service host is used for long-term work such as message consumption, scheduled tasks, and synchronization processing. The business is started and stopped through the plugin [Workers](../framework/core/components/worker.md) integration, and there is no need to write the loop directly into the host entry.

## Platform Lifecycle

The current entry uses `Application.Daemon("zongsoft.daemon", ...)` and passes in `host=daemon` and `site=daemon`. Integrate Windows Service or systemd depending on the build platform.

Source: [hosting/daemon/Program.cs](https://github.com/Zongsoft/hosting/blob/main/daemon/Program.cs#L9) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
static void Main(string[] args)
{
	#if WINDOWS
	Zongsoft.Plugins.Hosting.Application
		.Daemon("zongsoft.daemon", [.. args, "host=daemon", "site=daemon"], builder =>
		{
			builder.Services.AddWindowsService(options => options.ServiceName = builder.Environment.ApplicationName);
		}).Run();
	#elif LINUX
	Zongsoft.Plugins.Hosting.Application
		.Daemon("zongsoft.daemon", [.. args, "host=daemon", "site=daemon"], builder =>
		{
			builder.Services.AddSystemd();
		}).Run();
	#else
	Zongsoft.Plugins.Hosting.Application
		.Daemon("zongsoft.daemon", [.. args, "host=daemon", "site=daemon"])
		.Run();
	#endif
}
```
{% endcode %}

These calls make the host respond to the system service lifecycle and do not automatically install the service in the operating system. Service registration, startup account, working directory, automatic startup and recovery strategy are completed by the installation package or operation and maintenance configuration.

## Integration Order

1. Verify the business configuration and handler in [Terminal Host](terminal.md).
2. Prepare the daemon release directory and check the Main, daemon scheme and business plugins.
3. Start the deployed program in the foreground and confirm that there is no logic that relies on terminal input.
4. Register the service through the installation process that conforms to the target system, and check the actual running account and path.
5. Verify business readiness, recovery after stopping and restarting, rather than just checking that the service status is Running.

## How Workers Should Run

The startup phase completes the necessary initialization, the continuous loop should respond to cancellation, and avoid unbounded creation of background tasks. When external services are temporarily unavailable, there should be a clear reconnection or failure strategy; exceptions cannot be swallowed and the process appears to be healthy but is no longer processing work.

In the stop phase, reception is stopped first, and then in-transit tasks are processed within the allowed time. Message consumption needs to be combined with [Confirm with idempotent](../framework/messaging/reliability.md), and scheduled jobs need to be combined with [Scheduling lifecycle](../framework/externals/execution.md).

## Installation and Account

The Windows directory provides `install.cmd` and `uninstall.cmd`; the Linux installation package uses [Packaging Tool](../tools/packager.md) to generate service files and lifecycle scripts. Check the service name, program path and data directory before execution.

{% hint style="warning" %}
🚨 Installation/uninstallation will modify the system service configuration. The environment variables, user directories, certificates, and network permissions seen by the service account may differ from those seen by the interactive account and must be accessed using actual service authentication-dependent access.
{% endhint %}

## Common Faults

When starting and exiting, check entry, runtime, assembly and configuration loading errors. When the process is alive but not working, check whether the worker is mounted, whether the startup fails, and whether the message subscription or job storage is normal. Only when there is an error in the service environment, focus on comparing the working directory, account and environment configuration.

Logs and persistent data should have independent paths and permissions. Especially when using [Full upgrade](../framework/upgrading.md), in-application data cannot be regarded as retained content by default.

Source references: [Backend host](https://github.com/Zongsoft/hosting/tree/main/daemon).
