---
description: "Install Zongsoft related NuGet packages and global tools."
icon: download
---

# Install Packages

Zongsoft framework components are mainly distributed through NuGet packages, and the toolchain is distributed through .NET global tools.

## Compile Reference and Run Deployment

`dotnet add package` allows the project to compile and use the corresponding API; the plugin host also requires runtime artifacts such as `.plugin`, `.option`, mapping and dependencies. Business modules usually only reference the required contracts, and the specific driver is selected by the host deployment plan. For example, a module that only uses the data access contract does not have to directly reference all database drivers.

Team projects should have fixed compatible versions. When different versions of packages are deployed together, the final DLL and ancillary resources must match; successful NuGet restoration cannot be regarded as successful plugin deployment.

## Install Framework Package

Install the corresponding package according to your application needs. For example, plugin-based applications often use:

```bash
dotnet add package Zongsoft.Core
dotnet add package Zongsoft.Plugins
```

If web capabilities are required:

```bash
dotnet add package Zongsoft.Web
dotnet add package Zongsoft.Plugins.Web
```

If data access is required:

```bash
dotnet add package Zongsoft.Data
dotnet add package Zongsoft.Data.MySql
```

Available drivers include SQL Server, MySQL, SQLite, DuckDB, PostgreSQL, InfluxDB, TDengine and ClickHouse, etc. See [Package and Module Index](../references/packages.md) for details.

## Install Deployment Tools

`dotnet-deploy` is used to deploy plugins and side files based on the `.deploy` file.

```bash
dotnet tool install -g Zongsoft.Tools.Deployer
```

Update installed tools:

```bash
dotnet tool update -g Zongsoft.Tools.Deployer
```

## Install Packaging Tools

`dotnet-pack` is used to make `.tar.gz`, `.deb`, and `.rpm` installation packages.

```bash
dotnet tool install -g Zongsoft.Tools.Packager
```

## Check Tool

```bash
dotnet tool list -g
```

Once you've confirmed that the tools you need are included in the list, continue reading [Choose a Host](hosting.md).

## When the Tool Cannot Be Found

First check whether `dotnet tool list -g` and the tool directory are in PATH, then open a new terminal and try again. Global installation belongs to the current account, and system services or other accounts may not be able to find the same path. Offline or private source environments also require pre-arranged package caching and access permissions.

When making a plugin application, you only need to install the deployer first, and the Linux installation package and automatic upgrade tool will be prepared at the corresponding stage. Don't start all containers just to run an example with no external dependencies.
