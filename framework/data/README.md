---
description: "Design goals, capabilities, and starting points for the Zongsoft.Data data engine."
icon: database
---

# Data Engine

![Data Engine](../../.gitbook/assets/zongsoft-data-cover.png)

`Zongsoft.Data` is an ORM data access framework with a GraphQL-like style. Data schemas, mapping files, condition expressions, and database drivers describe the structure of data access, supporting complex queries, navigation, filtering, paging, grouping, aggregation, and writes without hand-written SQL.

Its design separates data access into four stable layers instead of replacing SQL with another SQL-like string language:

* Model layer: Business code uses POCO types or anonymous objects.
* Metadata layer: `.mapping` file describes entities, fields, inheritance and navigation relationships.
* Access layer: `IDataAccess` exposes unified query, write, aggregation, import and execution interfaces.
* Driver layer: Each database driver converts unified expressions into corresponding database syntax and executes them.

Start with [Object Relationships and Data Access](concepts.md), then follow [Your First Query](quickstart.md) through the Discussions table setup, connection configuration, and forum query. For organizing business rules, see [Data Services](services.md).

## Features

* Strict POCO objects are supported.
* Support read/write separation.
* Support table inheritance related operations.
* Supports isolation of mapping files by business module.
* Use a data schema to describe query and write shapes.
* Provides multiple database drivers.

## Core Concepts

<table data-view="cards">
	<thead>
		<tr>
			<th></th>
			<th></th>
			<th data-hidden data-card-target data-type="content-ref">Page</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><strong>Data Schemas</strong></td>
			<td>Describes the shape of the data queried or written.</td>
			<td><a href="schema.md">schema.md</a></td>
		</tr>
		<tr>
			<td><strong>Mapping Files</strong></td>
			<td>Describe entities, tables, fields, and relationships.</td>
			<td><a href="mapping.md">mapping.md</a></td>
		</tr>
		<tr>
			<td><strong>Connection Configuration</strong></td>
			<td>Configure data sources, read/write separation and drivers.</td>
			<td><a href="connections.md">connections.md</a></td>
		</tr>
		<tr>
			<td><strong>Data Access Interfaces</strong></td>
			<td>Execute queries, writes, aggregations and commands.</td>
			<td><a href="data-access.md">data-access.md</a></td>
		</tr>
		<tr>
			<td><strong>Conditions and Operands</strong></td>
			<td>Express filter conditions and field operations.</td>
			<td><a href="conditions-and-operands.md">conditions-and-operands.md</a></td>
		</tr>
		<tr>
			<td><strong>Queries and Navigation</strong></td>
			<td>Read the object graph using `schema`, paging and sorting.</td>
			<td><a href="querying.md">querying.md</a></td>
		</tr>
		<tr>
			<td><strong>Write Operations</strong></td>
			<td>Add, update, delete and upsert.</td>
			<td><a href="writing.md">writing.md</a></td>
		</tr>
		<tr>
			<td><strong>Drivers</strong></td>
			<td>Select, deploy and extend database drivers.</td>
			<td><a href="drivers.md">drivers.md</a></td>
		</tr>
	</tbody>
</table>

## Choose a Starting Point

{% tabs %}
{% tab title="I want to query data" %}
Let’s look at [Data Schemas](schema.md) and [Queries and Navigation](querying.md) first. These two pages explain how to represent fields, navigation properties, paging, and sorting.
{% endtab %}

{% tab title="I want to write data" %}
Let’s look at [Write Operations](writing.md) and [Conditions and Operands](conditions-and-operands.md) first. These two pages explain add, update, delete, upsert, and field operations.
{% endtab %}

{% tab title="I want to access the database" %}
Let’s look at [Connection Configuration](connections.md) and [Drivers](drivers.md) first. These two pages explain connection names, read/write separation, driver plugins, and deployment checks.
{% endtab %}
{% endtabs %}

## Browse by Driver

Common drivers include SQL Server, MySQL, SQLite, DuckDB, PostgreSQL, TDengine, ClickHouse, and InfluxDB. Drivers are usually deployed as independent plugins, and the data driver and connection settings driver are mounted to the plugin tree.

When a database has been selected, you can directly enter the [MySQL](drivers/mysql.md), [SQL Server](drivers/mssql.md), [PostgreSQL](drivers/postgres.md), [SQLite](drivers/sqlite.md), [DuckDB](drivers/duckdb.md), [ClickHouse](drivers/clickhouse.md), [TDengine](drivers/tdengine.md) or [InfluxDB](drivers/influx.md) project page.

See [Package and Module Index](../../references/packages.md) for the complete package name and [Plugin Manifests and Loading](../plugins/plugin-file.md) for the deployment method.

{% content-ref url="drivers.md" %}
[drivers.md](drivers.md)
{% endcontent-ref %}

## Typical Usage

Source: [src/Module.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Module.cs#L52) (excerpt; see source for context).

{% code title="Module.cs" %}
```csharp
public IDataAccess Accessor => _accessor ??= this.Services.ResolveRequired<IDataAccessProvider>().GetAccessor(this.Name);
```
{% endcode %}

In this call:

* `this.Services.ResolveRequired<IDataAccessProvider>()` resolves the data-access provider from the module’s services.
* `GetAccessor(this.Name)` obtains the accessor for the current module name; the shown source belongs to the Discussions module.
* `??=` caches the accessor for subsequent calls.

For query conditions, selected fields, navigation, sorting, and paging, continue with [Data Access Interfaces](data-access.md) and [Queries and Navigation](querying.md).

## Next Steps

{% content-ref url="schema.md" %}
[schema.md](schema.md)
{% endcontent-ref %}

{% content-ref url="data-access.md" %}
[data-access.md](data-access.md)
{% endcontent-ref %}

{% content-ref url="writing.md" %}
[writing.md](writing.md)
{% endcontent-ref %}

## Related Resources

* [Zongsoft.Data source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data)
* [Zongsoft.Data Chinese README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/README.zh-Hans.md)
* [Zongsoft.Data English README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/README.md)
* [Zongsoft.Data NuGet package](https://www.nuget.org/packages/Zongsoft.Data)
