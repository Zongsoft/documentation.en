---
description: "Collect hardware information of the current operating environment and understand the stability and usage scope of the device identification."
icon: microchip
---

# Hardware Information

`Zongsoft.Hardwares` implements the hardware acquisition contract in the core library `Zongsoft.IO.Hardwares`. It is suitable for device information display, diagnostics and auxiliary identification in the application authorization process; the collection results depend on the operating system, permissions and actual operating environment.

## Information Model

`IHardware` describes a single item of hardware. Common fields include `Code`, `Name`, `Type`, `Model`, and `Serie`. Whether a single device has unique characteristics should be judged by `HasUnique(out string)`. Don't mistake `HardwareProfile.Identifier` for a property that every device has.

`HardwareProfile` is used to combine device information and produce a summary ID. Single device, one-time collection result and machine portrait are three different levels: adding a network card, migrating a virtual machine or changing visible devices may affect the portrait input.

## Deployment and Collection

{% code title=".deploy" %}
```ini
[plugins zongsoft hardwares]
nuget:Zongsoft.Hardwares
```
{% endcode %}

Discussions There is no hardware acquisition use case. This page uses the real test of HardwareCollectorTest in the framework: reading the device through the collector, verifying that the collection and elements are not empty, and not outputting the real device identification. Plugin applications can also resolve collectors through core hardware contracts.

Source: [framework/Zongsoft.Hardwares/test/HardwareCollectorTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Hardwares/test/HardwareCollectorTest.cs#L11) (excerpt; see source for context).

{% code title="HardwareCollectorTest.cs" %}
```csharp
public void TestCollect()
{
	var hardwares = HardwareCollector.Instance.Collect();

	Assert.NotNull(hardwares);
	Assert.DoesNotContain(hardwares, hardware => hardware == null);
}
```
{% endcode %}

The framework samples/Program.cs uses the collection results to construct a HardwareProfile and prints the portrait and device details; this is suitable for local viewing, and the output should not be saved to the public log as it is. When a portrait is needed, the device collection collected this time can be used to construct `HardwareProfile`. It is recommended to first fix and record the device filtering rules adopted by the application, and then discuss whether the identification meets business requirements.

## Platforms and Async Boundaries

Currently, the platform collector is selected according to Windows, Linux, and macOS, and network device information is supplemented; other platforms only use the available network information path. What the container sees is usually the visible environment of the container, which cannot default to the complete hardware of the physical host.

`CollectAsync` checks the cancellation item by item during the collection enumeration process and asynchronously yields execution. It does not transform all underlying system queries into interruptible asynchronous calls; time-consuming platform queries may not respond to cancellation immediately. When it needs to be used in the request link, the call frequency should be controlled and caching should be considered.

{% hint style="warning" %}
🚨 Hardware fingerprints are not trusted credentials. Virtualization, device replacement, permission changes, and forged system information may all affect the results; the authorization system should also be designed with rebinding, failure recovery, and other authentication mechanisms.
{% endhint %}

## Troubleshooting and Usage Suggestions

When the field is empty, first use the same account to check whether the operating system exposes this information, and then compare the permission differences between the local, service account, and container. Don't treat "missing fields" as if the machine is illegal, and don't make up replacement serial numbers without proof.

If you need to report device portraits, you should collect only the fields required for the purpose, and control access, storage time, and desensitization methods. The diagnostics page gives priority to displaying the collection time, type and missing reasons, and the complete identification is retained in the management process with corresponding permissions.

Source references: [Collection implementation](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Hardwares/src), [core hardware contract](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/IO/Hardwares).
