---
description: "Understand OPC UA nodes, sessions, subscriptions, and certificates, and plan industrial device integration."
icon: plug
---

# OPC UA Device Protocol

OPC UA provides a unified protocol for data exchange between industrial devices and applications. This page introduces basic concepts such as nodes, sessions, subscriptions, and certificates, as well as the running conditions that need to be prepared before integration.

## Basic Concepts of OPC UA

OPC UA organizes device data in server address space. **NodeId** identifies the node, and the value of the node is also accompanied by type, status code and timestamp; only displaying the value will lose the quality and sampling time information.

**Session**is a session interacting with the server;**Subscription** and monitoring items are used to continuously receive changes. Subscriptions are not permanently valid objects, and their recovery status needs to be verified after disconnection and reconnection, server restart, and certificate changes.

`Zongsoft.Externals.Opc` provides related client, server and read-write subscription adaptations. Plugins make the assembly and SDK available, and the actual endpoints, certificate trusts, session establishment, and worker lifecycle are organized by the application.

## Start with a Local Example

Read [OPC example description](https://github.com/Zongsoft/framework/blob/main/externals/opc/samples/README.zh-Hans.md) first and use the supporting local server/client project. It is recommended to verify the connection, browse, read a known node, subscribe to changes, and then verify recovery after disconnection.

Before writing to the real device, the data type, dimension, allowed range and device status must be clarified. Device-side interlocking and subsequent status confirmation cannot be ignored just because the SDK write method returns success.

## Certificates and Identities

Application certificates are used to establish trusted communications, and user identities determine what operations a session can perform. The two cannot replace each other. The certificate subject, application URI, validity period, and trust directory should be consistent with the actual deployment.

{% hint style="warning" %}
🚨 Do not use sample self-signed certificates, automatically trust all certificates, or sample passwords written on the command line as formal deployment scenarios. The private key and trust directory are protected by the running account; after the trust is changed, the connection behavior of both the client and the server should be verified.
{% endhint %}

diagnostics retains at least node identification, quality status, source timestamps, and error stages, and controls access to sensitive device information. When exiting, subscription distribution should be stopped, the session should be closed, and the client should be released to avoid duplicate subscriptions remaining after restarting.

Source references: [OPC](https://github.com/Zongsoft/framework/tree/main/externals/opc).

## Continue Reading by Project

[Opc](projects/opc.md)
