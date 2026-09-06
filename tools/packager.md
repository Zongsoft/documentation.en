---
description: "Make the complete release directory into a Linux installation package, check the entry, systemd and installation lifecycle."
icon: box
---

# Packaging Tool

`dotnet-pack` makes the prepared application directory as `.tar.gz`, `.deb` or `.rpm`, and generates the metadata and lifecycle script required for installation. It will not compile business plugins or automatically complete the runtime directory for you.

## Select Format

| Commands | product | Applicable delivery methods |
| --- | --- | --- |
| `tar` | tar.gz and installation script | Explicitly run the install/uninstall process |
| `deb` | Debian packages | Target system package manager |
| `rpm` | RPM package | Target system package manager |

The tool uses .NET writing format, and the packaging phase does not rely on external packaging commands; verification and installation should still be performed in a suitable target environment. When Windows generates Linux packages, it also checks for inference of executable permissions.

## Make a Package

Discussions are hosted through the host and do not serve as separate executable portals. After installing Zongsoft.Tools.Packager and deploying business plugins, use hosting/web/default/pack.cmd to make a web hosting installation package. The actual commands and variables of the script are retained below; values such as format and edition are determined by the interactive input in front of the script. The complete pack.cmd should be executed in the script directory. This fragment cannot be pasted directly into PowerShell.

Source: [hosting/web/default/pack.cmd](https://github.com/Zongsoft/hosting/blob/main/web/default/pack.cmd#L77) (excerpt; see source for context).

{% code title="pack.cmd" %}
```bat
dotnet-pack %format%              ^
	--name:Zongsoft.Hosting.Web   ^
	--title:Zongsoft.Web          ^
	--edition:%edition%           ^
	--version:%version%           ^
	--compilation:%compilation%   ^
	--framework:%framework%       ^
	--platform:%platform%         ^
	--architecture:%architecture% ^
	--Environment:%environment%   ^
	--ASPNETCORE_ENVIRONMENT:%environment% ^
	--daemon:zongsoft.web         ^
	--daemon-bind:8069            ^
	--daemon-environments:Environment,ASPNETCORE_ENVIRONMENT ^
	--postinstalled:"../../.deploy/%scheme%/nginx/reload-nginx.sh" ^
	--postuninstalled:"../../.deploy/%scheme%/nginx/reload-nginx.sh" ^
	--exclude:**/logs/;bin/$(compilation)/$(framework)/*.staticwebassets.* ^
	../../mime                    ^
	appsettings.json              ^
	web*.config                   ^
	web*.option                   ^
	wwwroot                       ^
	plugins                       ^
	bin/$(compilation)/$(framework):~ ^
	"../../.deploy/%scheme%/nginx/zongsoft.web.conf:/etc/nginx/conf.d/zongsoft.web.conf"
```
{% endcode %}

Here ^ is the cmd line continuation character, %name% is the cmd variable, and $(name) is replaced by the packaging tool. The script also contains nginx installation hooks and should confirm the target environment. `name` should match the actual application entry instead of just filling in the product display name.

## File and Installation Path

Source directories are collected according to command rules when packaging items are not explicitly selected; content can be controlled using files, directories, last-level wildcards, and `source:target` aliases. `--exclude` is used to exclude logs, caches, and test configurations and cannot be assumed to support all cross-level wildcard syntax of deployer.

Root path aliases such as `/etc/nginx/conf.d/zongsoft.web.conf` represent installation to the system path. deb/rpm handles it as the corresponding root path entry, tars it into `.root/` and copies it by the installation script. Update, retention, and deletion policies for application files and machine configurations should be confirmed separately.

## Systemd Entry

When not disabled, the tool uses the specified service file or generates the service. `--daemon:none`, `disable`, and `disabled` can disable service generation; omitting daemon does not mean disabling it.

`--name` participates in host entry inference, `--daemon` can specify the service ID, and `--install-path` specifies the installation directory. The web host should keep the entry `Zongsoft.Hosting.Web` and the service name can be `zongsoft.web` to avoid generating services that start the `Zongsoft.Web.dll` class library.

`--daemon-bind:8080` generates native HTTP address parameters; `--daemon-environments` writes selected variables to the service file. Before secretly entering the service file, you need to clarify the permissions and maintenance methods.

## Lifecycle and Verification

Extension scripts are supported before and after installation and before and after uninstallation. The current generator distinguishes between Debian upgrade actions and remaining RPM instances to avoid mistaking upgrades for final uninstalls; scripts carried by older versions of installed packages may still affect upgrades, so supported legacy paths need to be tested.

{% code title="InspectPackages.sh" %}
```bash
tar -tf application.tar.gz
dpkg-deb --info application.deb
dpkg-deb --contents application.deb
rpm -qip application.rpm
rpm -qlp application.rpm
rpm -qp --scripts application.rpm
```
{% endcode %}

The above selects execution according to the actual generated format and replaces the file name; they check the product and do not perform installation. Focus on checking the entry files, installation paths, dependencies, permissions, configuration tags and service scripts, and then verify the first installation, upgrade and final uninstallation in the disposable target environment.

{% hint style="warning" %}
🚨 The script in the installation package will modify system services and paths. Just because the package can be read does not mean that installation, upgrade, and uninstallation are all secure; do not directly use production applications to verify generation rules.
{% endhint %}

Source references: [packager](https://github.com/Zongsoft/tools/tree/main/packager), [Full options](https://github.com/Zongsoft/tools/blob/main/packager/README.zh-Hans.md).
