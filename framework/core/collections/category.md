---
description: "Category classification tree and usage of CategoryBase, CategoryCollection, CategoryCollectionBase."
icon: list-tree
---

# Category

`Category` represents a nestable classification node. It inherits from `CategoryBase<Category>` and holds a collection of subcategories through the `Categories` attribute. It is often used in tree structures such as menus, function groupings, navigation categories, and module capability groupings.

## Type Relationship

| Type | Description |
| --- | --- |
| `Category` | Specific classification nodes, built-in `CategoryCollection` sub-collection. |
| `CategoryBase<TSelf>` | Classification node base class that provides titles, descriptions, icons, sorting, tags, and resource localization. |
| `CategoryCollection` | The default subcategory collection of `Category` provides the `Add` method for quickly adding categories by name. |
| `CategoryCollectionBase<TCategory>` | Classification collection base class, responsible for parent node maintenance and automatic sorting by `Ordinal`. |

`Category` is also a hierarchical node, so it inherits the capabilities of path, complete path, root node judgment and path search.

## Create a Classification Tree

Source: [framework/Zongsoft.Core/test/Collections/CategoryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/CategoryTest.cs#L119) (excerpt; see source for context).

{% code title="CategoryTest.cs" %}
```csharp
private static Category Initialize()
{
	var root = new Category();

	var file = root.Categories.Add("File");
	var edit = root.Categories.Add("Edit");
	var help = root.Categories.Add("Help");

	file.Categories.Add("Open");
	file.Categories.Add("Close");
	file.Categories.Add("Save");
	file.Categories.Add("SaveAs");
	file.Categories.Add("Recents").Categories.AddRange(
		new Category("Document-1"),
		new Category("Document-2")
	);

	return root;
}
```
{% endcode %}

`Category` constructed without parameters is the root node, its name is `/`, and its full path is also `/`. The names of ordinary nodes cannot contain illegal characters for hierarchical paths such as `/`, `\`, `*`, `?`, `!`, and `@`.

## Path Lookup

`Category` supports finding nodes by hierarchical path. Paths can contain whitespace, relative paths, root paths, and parent jumps.

Source: [framework/Zongsoft.Core/test/Collections/CategoryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/CategoryTest.cs#L36) (excerpt; see source for context).

{% code title="CategoryTest.cs" %}
```csharp
public void TestFind()
{
	var root = Initialize();

	var found = root.Find("File");
	Assert.NotNull(found);
	Assert.Equal("File", found.Name);
	Assert.Equal("/", found.Path);
	Assert.Equal("/File", found.FullPath);

	found = root.Find("Edit");
	Assert.NotNull(found);
	Assert.Equal("Edit", found.Name);
	Assert.Equal("/", found.Path);
	Assert.Equal("/Edit", found.FullPath);

	found = root.Find("Help");
	Assert.NotNull(found);
	Assert.Equal("Help", found.Name);
	Assert.Equal("/", found.Path);
	Assert.Equal("/Help", found.FullPath);

	found = root.Find(" File / Save");
	Assert.NotNull(found);
	Assert.Equal("Save", found.Name);
	Assert.Equal("/File", found.Path);
	Assert.Equal("/File/Save", found.FullPath);

	Assert.NotNull(root.Find(" File/ Recents"));
	Assert.NotNull(root.Find(" File /Recents / Document-1"));

	Assert.NotNull(root.Find(" /File/ Save"));
	Assert.NotNull(root.Find("/ File  /Recents"));
	Assert.NotNull(root.Find(" / File  /  Recents/Document-2"));

	found = root.Find("File").Find(" Open");
	Assert.NotNull(found);
	Assert.Equal("Open", found.Name);
	Assert.Equal("/File", found.Path);
	Assert.Equal("/File/Open", found.FullPath);

	found = root.Find("File").Find("./ Recents");
	Assert.NotNull(found);
	Assert.Equal("Recents", found.Name);
	Assert.Equal("/File", found.Path);
	Assert.Equal("/File/Recents", found.FullPath);

	found = root.Find("File").Find("Recents").Find("Document-2  ");
	Assert.NotNull(found);
	Assert.Equal("Document-2", found.Name);
	Assert.Equal("/File/Recents", found.Path);
	Assert.Equal("/File/Recents/Document-2", found.FullPath);

	found = root.Find("Edit").Find(".. / File / Save");
	Assert.NotNull(found);
	Assert.Equal("Save", found.Name);
	Assert.Equal("/File", found.Path);
	Assert.Equal("/File/Save", found.FullPath);
}
```
{% endcode %}

`Find` returns the matching node, or `null` if not found. White space before and after path segments are ignored.

## Sort

`CategoryBase<TSelf>.Ordinal` represents the sorting order of categories. When you add a node to `CategoryCollectionBase<TCategory>`, the collection is inserted into place as `Ordinal`.

Source: [framework/Zongsoft.Core/test/Collections/CategoryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/CategoryTest.cs#L97) (excerpt; see source for context).

{% code title="CategoryTest.cs" %}
```csharp
public void TestOrdinal()
{
	const int COUNT = 100;

	var root = new Category();

	for(int i = 0; i < COUNT; i++)
	{
		var ordinal = Random.Shared.Next() % COUNT;
		root.Categories.Add(new Category($"A{(i + 1):000}") { Ordinal = ordinal });
	}

	for(int i = 1; i < root.Categories.Count; i++)
	{
		Assert.NotNull(root.Categories[i]);
		Assert.NotNull(root.Categories[i - 1]);
		Assert.True(root.Categories[i].Ordinal >= root.Categories[i - 1].Ordinal);
	}
}
```
{% endcode %}

The test randomly sets the Ordinal of 100 nodes, and then verifies that the ordering value does not decrease item by item. Initialize and TestFind above belong to the same test fixture; their relationship needs to be preserved when reusing code.

## Localize Titles and Descriptions

`CategoryBase<TSelf>` can receive `Zongsoft.Resources.IResource`. When `Title` or `Description` is not set explicitly, the resource key is looked up by full path and node name.

The resource key search sequence for `Title` includes:

* `{path}.Category.Title`
* `{path}.Category`
* `{path}.Title`
* `{path}`
* `{name}.Category.Title`
* `{name}.Category`
* `{name}.Title`
* `{name}`

The resource key search sequence for `Description` includes:

* `{path}.Category.Description`
* `{path}.Description`
* `{name}.Category.Description`
* `{name}.Description`

Property change notifications are triggered when `Title`, `Description`, `Icon`, `Tags` are explicitly set.

## Related Resources

* [Category.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/Category.cs)
* [CategoryBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/CategoryBase.cs)
* [CategoryCollection.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/CategoryCollection.cs)
* [CategoryCollectionBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Collections/CategoryCollectionBase.cs)
* [CategoryTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Collections/CategoryTest.cs)
