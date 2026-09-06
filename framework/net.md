---
description: "Understand TCP packetization, handler lifecycle, and resource ownership when working with network communication components."
icon: network-wired
---

# Network Communication

`Zongsoft.Net` provides TCP-based clients, servers, channels and packetizers, as well as FTP file system integration. It is suitable for device communication or existing binary protocol integration; if the requirement is persistent messaging and consumer acknowledgments across services, you should read [Message Queues](messaging.md) first.

## Why Does TCP Need Subcontracting?

TCP provides an ordered stream of bytes. What the sender writes once may be read by the receiver multiple times; multiple writes may also be merged into one read. Therefore, "reading a byte" cannot be directly equivalent to "receiving a business message".

The packetizer is responsible for reducing the byte stream into protocol packets. Both parties must agree on the same format and check length, encoding and content.

| mode | behavior | Applicable premise |
| --- | --- | --- |
| `Headed` | Divide packets using a four-byte big-endian length header | The peer also uses the same length header protocol |
| `Headless` | Deliver content as currently readable data | The upper layer resolves the boundary by itself, or the protocol is inherently a flow |

Don't use `Headless` as a shortcut to "omit the length header but still receive the complete message every time". For text line protocols, read across half lines, multiple lines, maximum length, and unfinished lines before disconnecting should be handled.

## How the Client and Server Cooperate

The client configures the target address and receiving handler, and establishes a connection through `ConnectAsync`; the sending path can also be used to implement delayed connections. After the server listens to the endpoint, it establishes a channel for the connection and hands the decoded packet to the handler. Traffic processing and network reading are separate responsibilities: the packetizer is not responsible for authentication, traffic routing, or retries.

The repository provides `TcpClient.Headed` and `TcpClient.Headless` shared entries. Their addresses, handlers, and connection statuses are shared variable states, which are suitable for controlled single-connection scenarios; do not allow multiple business modules to individually rewrite the configuration of the same entry.

💡 When customizing the client, you must not only select the receiving packetizer, but also check whether the sending channel is packaged correctly. The built-in `Headed` client specifically covers the raw byte send path, and you cannot infer that all send overloads will automatically add a length header just by passing in a packetizer to the generic client.

## Memory and Lifecycle

The `System.Buffers.ReadOnlySequence<byte>` delivered in headless mode may reference a pipe buffer, the read of which should be completed during the handler call. If you need to put it into the background queue or save it for a long time, copy the valid content first.

Headless mode uses `System.Buffers.IMemoryOwner<byte>` to express buffer ownership. The framework channel releases the delivered resources after processing is complete; the handler must not continue to access its memory after returning. When writing a custom channel or handler, you need to know which layer releases the buffer to avoid repeated releases or dangling references.

Connection closing, application stopping and business cancellation should also be handled separately. Currently `ConnectAsync` does not accept cancellation tokens; the cancellation parameter on the send or disconnect method cannot be understood as a unified cancellation mechanism for connection establishment.

## Observable Results and Business Confirmation

The completion of sending usually indicates that the data has been handed over to the local transmission path, and the number returned by the broadcast indicates the number of connections successfully sent locally. None of them is proof that the peer business has been processed. When business confirmation is required, request identification, response, timeout and repeated request processing should be added to the protocol.

It is recommended to at least record the reasons for establishing/closing the connection, the amount of bytes sent and received, depacketization failure and processing time; do not completely record sensitive messages in ordinary logs. For listening ports exposed on untrusted networks, explicit connection authentication, packet size limits, and resource quotas also need to be applied.

## Verify Path

First use the repository's [Network example](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Net/samples) to verify the server and client with the loopback address in the two processes; the default example port is `7969`. Then, split writing, continuous multi-packet, abnormal length, and disconnection and reconnection tests were added to confirm the protocol boundaries and then integrate the real device.

Deploying `.plugin` will register related extensions, such as the FTP file system; it will not automatically start a TCP listener for the business. Starting/stopping listening should be undertaken by [Workers](core/components/worker.md) in the host lifecycle or a clear business entry.

Source references: [network components](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Net/src).
