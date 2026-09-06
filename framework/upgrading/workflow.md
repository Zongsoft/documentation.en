---
description: "Configure upgrade discovery and deployment handover, verify release status, and locate upgrade failures by stage."
icon: stairs
---

# Upgrade Integration and Recovery

Complete an upgrade in a disposable application directory before integrating it into production. Prepare the old version of the application, the new version of the complete runtime directory, the deployer matching the platform, and the independent package storage; do not directly use the development source code directory as the upgrade target.

## 1. Prepare Runtime Artifacts

Deploy `Zongsoft.Upgrading.Upgrader` in the application to be upgraded, and put the deployer that matches the platform and architecture into `{application-directory}/.deployer/`. The Windows file name is `Zongsoft.Upgrading.Deployer.exe`; other supported platforms are deployed according to the corresponding released products.

The deployer is a standalone Native AOT program and simply copying the in-app upgrader DLL is not enough to work. The startup process of Linux also relies on `systemd-run`; ordinary thin containers may not have this environment. Container deployment should first decide whether to replace the image with external orchestration or use self-upgrade in an environment with corresponding process management capabilities.

## 2. Configure the Discovery Channel

Discussions has no separate upgrade-client configuration. The following example uses the options shipped with the framework upgrader. They select the Web package manager by default and also declare a File channel. Adjust the addresses for the deployment environment of the application being upgraded.

Source: [framework/upgrading/upgrader/Zongsoft.Upgrading.Upgrader.option](https://github.com/Zongsoft/framework/blob/main/upgrading/upgrader/Zongsoft.Upgrading.Upgrader.option#L3) (excerpt; see source for context).

{% code title="Zongsoft.Upgrading.Upgrader.option" %}
```xml
<options>
	<option path="/Upgrading">
		<connectionSettings default="Web">
			<connectionSetting connectionSetting.name="Web"
			                   value="url=http://127.0.0.1:8069/Upgrading/Upgrader;timeout=30s" />

			<connectionSetting connectionSetting.name="File"
			                   value="url=zfs.s3:/upgrading/releases/" />
		</connectionSettings>
	</option>
</options>
```
{% endcode %}

`.option` is currently included in the package and `Web` is selected by default. Don't judge deployment defaults solely by the `default="File"` example in the old README. See [Option Configuration Files](../../references/option-files.md) for the name and coverage rules of the application configuration file.

The plugin registers the upgrader in the startup [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs), and the default period is ten minutes; when the period is greater than or equal to five minutes, a check will be scheduled about ten seconds after startup. Therefore, before installing and starting the plugin, you should prepare the correct release source and shutdown strategy.

## 3. Manage Packages and Release Status

After creating the ZIP and manifest, Web publishing proceeds in the order of "Import manifest → Upload package → Mark published". The server recalculates the size and checksum from the storage content after uploading. Package storage and metadata database are two types of resources that need to be backed up together and ensure consistent references.

The Web module requires the data engine, a database driver, an initialized upgrade database schema, and the storage provider configured under `/Upgrading/Settings`. The bundled manifest uses the SQLite path. If you change drivers, update the connection, database initialization, and plugin dependencies together.

The real controller method to discover the entry is as follows. name and edition come from the route, platform, architecture and additional parameters come from the request; use the actual host publishing identity when calling, instead of treating the Discussions plugin package name as the host name:

Source: [framework/upgrading/web/Controllers/UpgraderController.cs](https://github.com/Zongsoft/framework/blob/main/upgrading/web/Controllers/UpgraderController.cs#L48) (excerpt; see source for context).

{% code title="UpgraderController.cs" %}
```csharp
public async Task<IActionResult> GetAsync(string name, string edition, [FromQuery]Platform platform, [FromQuery]Architecture architecture, CancellationToken cancellation = default)
{
	if(string.IsNullOrWhiteSpace(name))
		throw new BadHttpRequestException($"The '{nameof(name)}' parameter is required.", StatusCodes.Status400BadRequest);

	var parameters = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);

	foreach(var pair in this.Request.Query)
		parameters[pair.Key] = pair.Value;

	foreach(var header in this.Request.Headers)
		parameters[header.Key] = header.Value;

	using var stream = new MemoryStream();
	await Release.SaveAsync(stream, Upgrader.GetAsync(name, edition, platform, architecture, parameters, cancellation), cancellation);
	return this.File(stream.ToArray(), "application/manifest+xml");
}
```
{% endcode %}

The response uses the shared release protocol’s XML format. A release must be visible, published, and not deprecated, with a matching identity and version range, an available package path, and a positive package size. If an evaluator is specified, the release must also pass its evaluation.

The evaluator is suitable for deciding whether a release is suitable for a specific instance, such as a staged rollout group. Request parameters and headers are input only, and hardware fields are not reliable credentials; authentication and publishing permissions should be handled by independent mechanisms. The evaluator should be deterministic and free of side effects wherever possible, so as to avoid modifying the business status in one version check.

## 4. Understand Preparation and Deployment

The in-app `UpgradeAsync` is responsible for downloading, verifying by metadata, decompressing and writing `.deployment`. Returning `true` may also mean that a description has been found to be deployed, and it cannot be considered that all files have just been re-validated.

`Deploy` checks the existence of the deployer and closes the current host after starting the program. The out-of-process deployer waits for the host to exit, reads the description file exclusively, processes the file in full or incremental mode, and then executes the corresponding launcher.

| Application/Platform | Current Restart Path |
| --- | --- |
| Windows Web | IIS application pool recycling path |
| Windows background service | `sc start` |
| Linux/FreeBSD Services | `systemctl start` |
| Terminal and universal host | Start process directly |

The deployment account must have corresponding permissions, and the host type and startup parameters must be correct. Just because a platform can run .NET does not mean that every system service management method on it exists.

## 5. Check Results and Recovery

| Symptom | Check First |
| --- | --- |
| No release found | Actual application name, distribution name, version, platform/architecture, release status, evaluator |
| Download failed | Package URL, storage provider, credentials, timeout, file size and checksum |
| Already have `.deployment` but do not exit | Whether `.deployer` exists, process startup log, file lock |
| Exited with incomplete file | Clean/copy errors, disk space, file usage, executor logs |
| File updated but service not restored | Launcher, service name, working directory, permissions and actual process logs |

Full deployment will clear the in-application logs, and log output or snapshots should be saved outside the application directory during upgrade acceptance. Finally, confirm the end of `.deployment` lifecycle, `.version` content and the health status of the new process at the same time.

After failure, first save a copy of the description file, manifest, decompression directory and log to determine which stage it has stopped at. Do not directly delete all handover information in order to allow the [worker](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/IWorker.cs) to run again; if the file has been partially replaced, you should restore it to a consistent state based on the prepared recovery package and data backup, and then try again.

Source references: [client](https://github.com/Zongsoft/framework/tree/main/upgrading/upgrader), [Deployment order](https://github.com/Zongsoft/framework/blob/main/upgrading/deployer/Deployer.Deploy.cs), [Web manager](https://github.com/Zongsoft/framework/tree/main/upgrading/web).
