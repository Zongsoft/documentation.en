---
description: "Understand object-relational mapping, entities, navigation, data shapes, accessors and database-driven collaboration."
icon: diagram-project
---

# Object Relationships and Data Access

The business code operates order, customer, and detail objects; the relational database stores tables, rows, columns, and associated keys. Object-relational mapping (ORM) is responsible for connecting these two representations. [Zongsoft.Data](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Data) uses explicit mapping and data schema to allow the business to describe "which entities, fields and relationships are accessed", which are then executed by the engine and driver.

## Four Levels

**model**is the object type in the code, which can be an ordinary attribute class or use the model mechanism supported by the framework.**mapping**describes entities and tables, attributes and columns, navigation and associated keys.**data schema**describes the member range of a certain operation.**drive** converts expressions into commands supported by the target database.

Take order details as an example: `CustomerId` of the mapping declaration order is associated with `CustomerId` of the customer; a detailed query uses `OrderId, Amount, Customer{Name}, Lines:20{ProductId, Quantity}`; another list query only requires `OrderId, Amount`. The same map can support different shapes without having to rewrite relationships for each query.

## Entities and Names <a id="entity"></a>

`container` in the map provides the namespace for entities and commands. For example, Discussions.Forum in the Discussions mapping is the entity qualified name, and the entity alias points to the Discussions_Forum physical table. Accessor names, mapping container names, and database names are not the same concepts, even though projects often name them with the same text.

Generic queries need to locate entities according to the model; dynamic or cross-module scenarios can explicitly pass the entity qualified name. First check [mapping](mapping.md), and then confirm the model and entity names used in the code.

## Navigation and Relationship Cardinality <a id="navigation"></a>

navigation property expresses "accessing another object along a relationship". The cardinality of a relationship indicates how many objects can be associated: `?` means zero or one, `!` means there must be one, and `*` means a set. They affect how connections and objects are assembled, and cannot be guessed based on C# property names alone.

Single value navigation is read through a join, and collection navigation forms independent slave queries and is populated during result assembly. Requesting a nested object graph does not guarantee that only one SQL will be executed; the deeper the hierarchy and the larger the collection, the more database round-trips and data may be returned. Fields and limits should be selected according to the actual needs of the page.

## Data Schema and Database Structure <a id="schema"></a>

The Schema here refers to the data shape of the operation, not the database table creation script, nor the GraphQL server protocol. It is syntactically similar to a selection set, but has its own parsing rules. `.mapping` is stable structure metadata, and the two have different responsibilities.

{% hint style="info" %}
💡 Deploying `.mapping` will not automatically create tables, migrate databases, or populate initialization data. The mapping, connection, driver and real database structure must be prepared separately for the first integration; [Constant query without table](quickstart.md) can be used only when verifying the assembly.
{% endhint %}

## Conditions and Operand <a id="condition"></a>

Conditions answer "which rows participate in the operation", operand answers "how the values in the expression are evaluated". For example, `Condition.Equal("OrderId", orderId)` limits orders, and `Operand.Field("Quantity") + 1` represents field operations in the database.

Giving the condition value to the engine for binding helps avoid manually spelling SQL. Field names and accessible members must still be controlled by the application; parameterization does not automatically implement field permissions or tenant isolation.

## Accessors and Data Services <a id="service"></a>

[accessor](data-access.md) provides general data operations. [Data Services](services.md) organizes model-related business entrances, writability, verification, authorization and filtering on it. Web controllers can use data services without having to define the same access flow repeatedly at each endpoint.

Database transactions handle submission and rollback within the same transaction scope, and business authorization handles who can operate which data. These responsibilities are not automatically fulfilled by introducing an ORM. In particular, database writing and message sending cannot be assumed to be completed atomically with just one local transaction.

## Recommended Learning Order

Complete [First query](quickstart.md) first, then learn [mapping](mapping.md) and [Data Schemas](schema.md), and then enter [Query](querying.md), [write](writing.md), [affairs](transactions.md) and [Data Services](services.md). See [Drivers](drivers.md) for driver differences.
