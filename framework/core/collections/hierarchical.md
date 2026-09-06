---
description: "Hierarchical hierarchical nodes, node collections, path search and hierarchical expressions."
icon: code-branch
---

# Hierarchical

`Hierarchical` related types provide tree nodes, node collections, path search and expression parsing capabilities. Models such as `Category`, configuration tree, option tree, etc. that need to "locate nodes by path" can reuse this infrastructure.

## Type Relationship

| Type | Description |
| --- | --- |
| `HierarchicalNode` | Non-generic hierarchical node base class that provides name, path, full path, and illegal character constraints. |
| `HierarchicalNode<TNode>` | Generic hierarchical node base class, providing parent node, child node collection and `Find` search logic. |
| `HierarchicalNodeCollection<TNode>` | A collection of child nodes based on the name key, ignoring case, prohibiting adding root nodes. |
| `HierarchicalNodeUtility` | Provides `IsRoot` and other hierarchical node extension methods. |
| `HierarchicalExpression` | Represents the parsing result of "hierarchical path + member access expression". |
| `HierarchicalExpressionParser` | Parse hierarchical expression text. |

## Hierarchy Node

Hierarchical nodes use `/` as the path separator. The root node name is `/`, `Path` of the root node is an empty string, and `FullPath` is `/`.

Source: [framework/Zongsoft.Core/test/Collections/CategoryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/CategoryTest.cs#L11) (excerpt; see source for context).

{% code title="CategoryTest.cs" %}
```csharp
public void TestName()
{
	var root = Initialize();

	Assert.True(root.IsRoot());
	Assert.Equal("/", root.Name);
	Assert.Equal(string.Empty, root.Path);
	Assert.Equal("/", root.FullPath);

	var category = new Category();
	Assert.True(root.IsRoot());

	Assert.IsType<ArgumentException>(Record.Exception(() => root.Categories.Add(category)));
	Assert.IsType<ArgumentException>(Record.Exception(() => new Category("/")));
	Assert.IsType<ArgumentException>(Record.Exception(() => new Category("ABC\\")));
	Assert.IsType<ArgumentException>(Record.Exception(() => new Category("ABC/DEF")));
	Assert.IsType<ArgumentNullException>(Record.Exception(() => new Category(string.Empty)));
	Assert.IsType<ArgumentNullException>(Record.Exception(() => new Category(" ")));
	Assert.IsType<ArgumentNullException>(Record.Exception(() => new Category("\t")));
	Assert.IsType<ArgumentNullException>(Record.Exception(() => new Category("\n")));
	Assert.IsType<ArgumentNullException>(Record.Exception(() => new Category("\r")));
	Assert.IsType<ArgumentNullException>(Record.Exception(() => new Category(Environment.NewLine)));
}
```
{% endcode %}

The node name cannot be empty and cannot contain path separators and characters reserved for hierarchical expressions.

## Pathfinding

`HierarchicalNode<TNode>.Find` supports the following path forms:

| path | Description |
| --- | --- |
| `/A/B` | Start searching from the root node. |
| `A/B` | Start searching from the child nodes of the current node. |
| `./A` | Start searching from the current node. |
| `../A` | Start searching from the parent node. |
| empty string | Return the current node. |

For the complete creation and search process, see [Real test examples for Category](category.md#path-lookup), where the same File / Edit / Help classification tree is reused.

Whitespace at both ends of path segments is ignored. Returns `null` when the node is not found.

## Hierarchical Expression

`HierarchicalExpression` is used to parse "node path + member accessor". The expression consists of two parts: path and optional member access.

Source: [framework/Zongsoft.Core/test/Collections/HierarchicalExpressionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/HierarchicalExpressionTest.cs#L16) (excerpt; see source for context).

{% code title="HierarchicalExpressionTest.cs" %}
```csharp
var TEXT = @"/";
var expression = HierarchicalExpressionParser.Parse(TEXT);

Assert.NotNull(expression);
Assert.Null(expression.Accessor);
Assert.Equal(PathAnchor.Root, expression.Anchor);
Assert.Equal("/", expression.Path);
Assert.True(expression.Segments == null || expression.Segments.Length == 0);
```
{% endcode %}

Common formats include:

* `/root/node@property`
* `../sibling/node@property`
* `child/node[index]`
* `@property1.property2`

Member access is partially parsed by `Zongsoft.Reflection.Expressions`, so property, indexer, and chained member access can be expressed.

## Related Resources

* [HierarchicalExpression.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/HierarchicalExpression.cs)
* [HierarchicalExpressionParser.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/HierarchicalExpressionParser.cs)
* [HierarchicalNode.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/HierarchicalNode.cs)
* [HierarchicalNode&lt;T&gt;.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/HierarchicalNode%601.cs)
* [HierarchicalNodeCollection.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/HierarchicalNodeCollection.cs)
* [HierarchicalNodeUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/HierarchicalNodeUtility.cs)
* [HierarchicalExpressionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/HierarchicalExpressionTest.cs)
