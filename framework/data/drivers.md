---
description: "Select, deploy and extend Zongsoft.Data database driver."
icon: hard-drive
---

# Browse by Driver

The driver is responsible for converting unified data expressions into specific database syntax and executing commands. A complete driver usually consists of two parts: the connection settings driver and the data driver.

## Browse by Driver

When the database has been determined, you can enter it from the corresponding project page. Each page focuses on explaining the package name, driver key, deployment connection and special verification; topics such as [Query](querying.md), [write](writing.md), [mapping](mapping.md) continue to explain the usage of each driver sharing.

| drive | source code directory | NuGet package |
| --- | --- | --- |
| [MySQL](drivers/mysql.md) | `mysql` | `Zongsoft.Data.MySql` |
| [SQL Server](drivers/mssql.md) | `mssql` | `Zongsoft.Data.MsSql` |
| [PostgreSQL](drivers/postgres.md) | `postgres` | `Zongsoft.Data.PostgreSql` |
| [SQLite](drivers/sqlite.md) | `sqlite` | `Zongsoft.Data.SQLite` |
| [DuckDB](drivers/duckdb.md) | `duckdb` | `Zongsoft.Data.DuckDB` |
| [ClickHouse](drivers/clickhouse.md) | `clickhouse` | `Zongsoft.Data.ClickHouse` |
| [TDengine](drivers/tdengine.md) | `tdengine` | `Zongsoft.Data.TDengine` |
| [InfluxDB](drivers/influx.md) | `influx` | `Zongsoft.Data.Influx` |

## Plugin Registration

The driver is deployed as a plugin. Taking MySQL as an example, the plugin will depend on `Zongsoft.Data` and register objects with two extension points:

Source: [framework/Zongsoft.Data/drivers/mysql/src/Zongsoft.Data.MySql.plugin](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/mysql/src/Zongsoft.Data.MySql.plugin#L19) (excerpt; see source for context).

{% code title="Zongsoft.Data.MySql.plugin" %}
```xml
<extension path="/Workbench/Configuration/ConnectionSettings/Drivers">
	<object name="MySql" value="{static:Zongsoft.Data.MySql.Configuration.MySqlConnectionSettingsDriver.Instance, Zongsoft.Data.MySql}" />
</extension>
```
{% endcode %}

`driver="MySql"` in the connection configuration must be consistent with the object name here.

## Deployment Check

Before using a database, check three things:

- There are corresponding drivers `.plugin` and `.dll` in the plugin directory.
- The driver plugin declares dependence on `Zongsoft.Data`.
- The `driver` name of the connection configuration is consistent with the plugin registration name.

If the accessor cannot be created, first check whether the connection configuration path, default connection name and driver plugin are loaded.

## Driving Responsibilities

Drivers usually need to implement:

- Connection string parsing.
- Data source created.
- SQL or SQL-like expression generation.
- Query, insert, update, delete, upsert and aggregation execution.
- Data import.
- Functions, paging, return values, and parameter handling in the database dialect.

Business code should not directly rely on specific driver types. Keep modules replaceable by selecting drivers through connection configuration and data access interfaces.

## What to Check Before Choosing a Driver

What drives the unification is the application call and mapping process; the query semantics, concurrency model and transaction capabilities of the database still differ. First clarify whether the business is transaction processing, time series collection or local analysis, and then check the actual support scope of the target driver and database. Don't assume that just because both systems accept SQL, they are directly interchangeable.

It is recommended to use a minimal model that can represent the business to verify the following items:

| Check items | Why it affects business |
| --- | --- |
| String, identifier case and null value comparison | The same conditions may match different records on different databases |
| Amount accuracy, time accuracy and time zone | May affect settlement, sorting and time range filtering |
| Auto-increment, return value and upsert operations | Affects whether the model obtains the correct identification and final value after writing |
| Navigation queries and pagination | To check actual SQL, number of results, and execution cost |
| Transactions, concurrent updates and failure rollback | The unified transaction entry cannot create transactions that are not supported by the database. |
| Bulk import, default values and constraints | The behavior of skipping fields, ignoring conflicts and failed batches must be clear |

You can start by using [First query](quickstart.md) to verify the host link; then verify these business semantics in an independent test library. The connection can be opened and the constant query is successful, which only shows the basic connectivity.

## Two Implementations That Require Special Understanding

### DuckDB Import Path

Currently [DuckDB importer](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/duckdb/src/DuckDBImporter.cs) is a regular type. First create a temporary table within the connection, append data in batches according to selected fields, and then execute `INSERT ... SELECT` again to write to the target table. This uses the bulk append capability but also allows the imported fields to be only a subset of the physical table fields and retains the default values of unspecified target columns.

When the database custom type is mapped to `System.Data.DbType.Object`, the importer uses parameterized row-by-row insertion instead. Therefore, the field types of the same batch of data may also change the execution path, and the throughput cannot be estimated based only on the "batch import" name.

The import joins the current data transaction by default; when no external transaction is obtained, the independent connection uses internal transactions to protect the current batch. Enabling the Ignore Constraints option may skip conflicting rows, and a successful import call does not mean that every input becomes a new record. See [Transactions and Consistency](transactions.md) for transaction background.

### InfluxDB Version and Transaction Boundaries

Currently [Influx driver](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/influx/src/InfluxDriver.cs) uses InfluxDB3.Client, targets InfluxDB 3, and declares the `TransactionSuppressed` attribute. Do not understand it as a local transaction participant of the relational database, and do not directly apply the configuration and query language of the old version of InfluxDB into the current driver.

{% hint style="warning" %}
🚨 Businesses that rely on "multiple writes all succeed or all are revoked" must confirm this capability before selecting a driver. The unified interface facilitates code reuse but does not automatically complete transactions, constraints, or data model semantics that are missing from the target database.
{% endhint %}

## Related Resources

| drive | Source | README | NuGet |
| --- | --- | --- | --- |
| SQL Server | [mssql](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/mssql) | [README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/mssql/README.md) | [Zongsoft.Data.MsSql](https://www.nuget.org/packages/Zongsoft.Data.MsSql) |
| MySQL | [mysql](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/mysql) | [README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/mysql/README.md) | [Zongsoft.Data.MySql](https://www.nuget.org/packages/Zongsoft.Data.MySql) |
| PostgreSQL | [postgres](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/postgres) | [README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/postgres/README.md) | [Zongsoft.Data.PostgreSql](https://www.nuget.org/packages/Zongsoft.Data.PostgreSql) |
| SQLite | [sqlite](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/sqlite) | [README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/sqlite/README.md) | [Zongsoft.Data.SQLite](https://www.nuget.org/packages/Zongsoft.Data.SQLite) |
| DuckDB | [duckdb](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/duckdb) | [README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/duckdb/README.md) | [Zongsoft.Data.DuckDB](https://www.nuget.org/packages/Zongsoft.Data.DuckDB) |
| ClickHouse | [clickhouse](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/clickhouse) | [README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/clickhouse/README.md) | [Zongsoft.Data.ClickHouse](https://www.nuget.org/packages/Zongsoft.Data.ClickHouse) |
| TDengine | [tdengine](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/tdengine) | [README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/tdengine/README.md) | [Zongsoft.Data.TDengine](https://www.nuget.org/packages/Zongsoft.Data.TDengine) |
| InfluxDB | [influx](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data/drivers/influx) | [README](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/drivers/influx/README.md) | [Zongsoft.Data.Influx](https://www.nuget.org/packages/Zongsoft.Data.Influx) |
