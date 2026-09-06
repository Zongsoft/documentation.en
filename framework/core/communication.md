---
description: "Responsibilities and main types of the Zongsoft.Communication namespace."
icon: radio
---

# Zongsoft.Communication

`Zongsoft.Communication` provides basic abstractions such as communication channels, listeners, transceivers, requesters, and protocol packet parsing. It is not bound to a specific transmission protocol, and the specific network implementation is usually undertaken by modules such as `Zongsoft.Net`.

## Main Responsibilities

* Define communication abstractions such as `IChannel`, `IListener`, `IReceiver`, `IRequester`, etc.
* Provides channel base classes, channel collections, channel selectors, and notifier models.
* Provides extension point for protocol package parsing such as `IPacketizer`.
* Provide a unified design entry for the network layer or custom communication protocols.

## Type

<table data-view="cards">
	<thead>
		<tr>
			<th></th>
			<th></th>
			<th data-hidden data-card-target data-type="content-ref">Page</th>
		</tr>
	</thead>
	<tbody>
		<tr><td><strong>General Communication</strong></td><td>Transceivers, listeners, channels and protocol packet parsing.</td><td><a href="communication/general.md">general.md</a></td></tr>
		<tr><td><strong>Request/Response</strong></td><td>Multi-response requests, response correlation, and ZeroMQ implementation.</td><td><a href="communication/request-response.md">request-response.md</a></td></tr>
		<tr><td><strong>Notifier</strong></td><td>Notifies the initiator abstraction and current implementation status.</td><td><a href="communication/notifier.md">notifier.md</a></td></tr>
		<tr><td><strong>Transmitter</strong></td><td>Template notification senders, descriptors and GUI configuration scenarios.</td><td><a href="communication/transmitter.md">transmitter.md</a></td></tr>
	</tbody>
</table>

## Related Resources

* [Communication source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Communication)
* [Zongsoft.Net source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Net)
