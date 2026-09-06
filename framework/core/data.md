---
description: "Responsibilities of the Zongsoft.Data namespace and its subnamespaces in the core class library."
icon: database
---

# Zongsoft.Data

[`Zongsoft.Data`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data) defines the basic abstraction of data access, data service model, conditions, operand, mode, paging, sorting, transactions, data dictionary and data metadata in the core class library. The complete data engine implementation is continued to be extended by the `Zongsoft.Data` module.

## Main Responsibilities

* Define data access and data service abstractions such as `IDataAccess`, `IDataService`, `IDataSearcher`, etc.
* Express query conditions, operand, paging, sorting, return values, data schema and data types.
* Provides [`Transaction`](data/transactions.md) ambient transaction objects and transaction participation registration models.
* Provides data service events, data manipulation options, model descriptions, and data dictionaries.
* Provides data metadata abstraction for use by mapping file and data engine implementations.

## Sub Namespace

| namespace | Description |
| --- | --- |
| `Zongsoft.Data.Archiving` | Data archiving related abstractions. |
| `Zongsoft.Data.Metadata` | Data metadata models such as entities, attributes, associations, commands and parameters. |
| [`Zongsoft.Data.Transactions`](data/transactions.md) | Transaction state, transaction information, registration context, and transaction participant abstractions. |

## Related Resources

* [Data source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Data)
* [Data Engine](../data/README.md)
* [Zongsoft.Data NuGet package](https://www.nuget.org/packages/Zongsoft.Data)
