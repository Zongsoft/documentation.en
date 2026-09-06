---
description: "Prepare development dependencies using repository Podman manifests, identifying ports, mounts, and persistence boundaries."
icon: boxes-stacked
---

# Container Environments

The hosting repository's Podman manifest is primarily used to prepare development and verify dependencies. The list includes service ports, initialization scripts, and local mounting paths. You should check according to your own workspace and data storage requirements before use; they are not equivalent to a production cluster that has undergone complete operation and maintenance design.

## Existing List

| Checklist | Pod name | Purpose |
| --- | --- | --- |
| `zongsoft.pod-host.yaml` | `zongsoft` | Development/build host environment and workspace mounting |
| `zongsoft.pod-redis.yaml` | `zongsoft.caching` | Redis |
| `zongsoft.pod-etcd.yaml` | `zongsoft.distributed` | etcd, map 2379 |
| `zongsoft.pod-mysql.yaml` | `zongsoft.data` | MySQL, mapping 3306 |
| `zongsoft.pod-postgres.yaml` | `zongsoft.data` | PostgreSQL, mapping 5432 |
| `zongsoft.pod-rustfs.yaml` | `zongsoft.io` | S3 compatible storage |

The MySQL and PostgreSQL manifests use the same Pod name and should not be treated as two environments that can be started independently at the same time without checking. When coexistence is required, resource naming and network organization should be specified by the environment scheme.

## Check Before Start

First check whether the hostPath in the list exists, whether the initialization SQL comes from the expected repository, whether the port is occupied, and then check the credentials and data volume. Database initial mount does not equal database data directory persistence; the current etcd inventory does not have persistent volumes and should not hold data that needs to be retained across rebuilds.

`zongsoft.pod(start).cmd` and `zongsoft.pod(stop).cmd` will ask for service selection. Read the corresponding script before executing to confirm the operation target; stopping the container, deleting the Pod and deleting the data volume are different operations and cannot be understood as a lossless restart.

## Where Can I Find the Address?

Hosts running on Windows typically access services through a port mapped to the host, such as `127.0.0.1:2379`. `localhost` in the container points to itself; for cross-Pod access, you need to confirm the network, name resolution, and service listening address. Reachability cannot be guaranteed based on only one Pod name.

{% code title="InspectPods.ps1" %}
```powershell
podman ps --all --pod
podman pod ps
```
{% endcode %}

These read-only commands are used to view status. Process Running does not mean that the database has been initialized. You should continue to check the corresponding container log or service readiness detection before starting the plugin that depends on it.

## Windows and WSL

The Windows Podman operating environment relies on corresponding virtualization and network configuration. When an address is unreachable, first distinguish port mapping, DNS, proxy, firewall and WSL network, and then refer to the environment description of the hosting README. Don't think of switching mirrored/NAT as a fixed fix for all problems.

When mounting source code, you must also consider the path format, permissions, case, and newline characters of the host and container. The script executable bit and correct LF under Linux will also affect the operation.

## Relationship to Application Delivery

Developing containers is used to reproduce dependencies, and application image delivery requires additional arrangements for configuration injection, data volumes, health checks, logs, and version releases. Thin containers usually do not include systemd, and it cannot be directly assumed that the service restart path of [out-of-process upgrader](../framework/upgrading/workflow.md) is available.

Source references: [Podman manifests and scripts](https://github.com/Zongsoft/hosting), [Environmental Statement](https://github.com/Zongsoft/hosting/blob/main/README.zh-Hans.md).
