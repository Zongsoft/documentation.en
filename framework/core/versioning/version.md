---
description: "The semantic version and numerical version number in the Zongsoft.Versioning namespace."
icon: code-commit
---

# Zongsoft.Versioning

`Zongsoft.Versioning` provides the version representation model in Zongsoft.Core. It separates "semantic version used for publishing, display and compatibility judgment" and "numeric version number used for sorting, range query and persistence" to prevent the same type from taking on both textual semantics and integer encoding responsibilities at the same time.

## When to Use

`Version` represents the semantic version, in the format `major.minor.patch-label+extra`. It is suitable for describing public versions of applications, modules, protocols or packages, such as `1.2.3`, `1.2.3-alpha.1`, `1.2.3-alpha.1+build.5`.

`Version.Number` represents a four-segment numerical version number, including four `ushort` fields: `Major`, `Minor`, `Patch`, and `Revision`. It is suitable for saving to databases, configuration items, or persistent fields that require stable ordering, and can also work with older integer version values.

Discussions does not directly operate this version type; the following uses Core's VersionTest and VersionNumberTest to retain test input and output assertions. They validate version rules and do not represent the current Discussions package version.

## Semantic Version

`Version` is a reference type and must be constructed with the nonnegative `Major`, `Minor`, and `Patch`. `Label` and `Extra` will remove leading and trailing blanks; blank values will be treated as having no labels or additional information.

Source: [framework/Zongsoft.Core/test/Versioning/VersionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Versioning/VersionTest.cs#L94) (excerpt; see source for context).

{% code title="VersionTest.cs" %}
```csharp
public void TestFormat()
{
	var version = new Version(1, 2, 3, "alpha.1", "build.5");

	Assert.Equal("1.2.3-alpha.1", version.ToString());
	Assert.Equal("1.2.3-alpha.1", version.ToString("N"));
	Assert.Equal("1.2.3-alpha.1+build.5", version.ToString("F"));
	Assert.Equal("1.2.3", version.ToString("V"));
	Assert.Equal("alpha.1", version.ToString("R"));
	Assert.Equal("build.5", version.ToString("M"));
	Assert.Equal("1.2.3.0", version.ToString("x.y.z.r"));
}
```
{% endcode %}

Parsing requires exactly three segments of the numeric part and follows the basic constraints of the semantic version: numeric fields cannot have leading zeros; labels and additional information consist of dot-delimited identifiers, and identifiers can only contain letters, numbers, and hyphens; purely numeric identifiers in labels cannot have leading zeros. Extra information allows for something like `build.01`.

When comparing, first compare `Major`, `Minor`, and `Patch`. When the numbers are the same, the stable version without `Label` is higher than the pre-release version with `Label`; labels are compared by point segments, numeric identifiers are compared by numeric values, numeric identifiers are lower than non-numeric identifiers, and text comparisons ignore case. `Extra` is only output as construction or additional information and does not participate in equality and size comparisons.

| member | Description |
| --- | --- |
| `Major`、`Minor`、`Patch` | A semantic version of a three-segment number. |
| `Label` | Pre-release tag, corresponding to `-alpha.1` in the semantic version. |
| `Extra` | Additional information, corresponding to `+build.5` in the semantic version. |
| `HasLabel()` / `HasExtra()` | Determine and read optional tags or additional information. |
| `Parse(...)` / `TryParse(...)` | Parse `Version` from text. |
| `CompareTo(...)` | Compare by semantic version priority. |
| `ToString(format)` | Output different forms of version text according to format characters. |

### Format

`Version.ToString()` is equivalent to `ToString("N")` by default, and will output the digital version and pre-release tags, but not `Extra`. If you need to preserve build information, `ToString("F")` should be used.

| Format | output |
| --- | --- |
| `N` | Normalized version: `1.2.3` or `1.2.3-alpha.1`. |
| `V` | Digital version only: `1.2.3`. |
| `F` | Full version: `1.2.3-alpha.1+build.5`. |
| `R` / `L` | Tags only. |
| `M` / `E` | Additional information only. |
| `x`、`y`、`z` | Output the major version, minor version and revision number respectively. |
| `r` | The reserved fourth segment is currently always `0`. |

{% hint style="warning" %}
The JSON converter for `Version` will write out `ToString("F")`, so `Extra` will be retained; the type converter uses the default `ToString()` when converting to a string, and will not output `Extra`.
{% endhint %}

## Numeric Version Number

`Version.Number` is a value type, retaining the usage scenarios of the old four-segment version value. It can be parsed from `1.2`, `1.2.3`, `1.2.3.4`; the missing `Patch` or `Revision` will be replaced by `0`, but single-segment versions and version segments exceeding the `ushort` range are not supported.

Source: [framework/Zongsoft.Core/test/Versioning/VersionNumberTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Versioning/VersionNumberTest.cs#L172) (excerpt; see source for context).

{% code title="VersionNumberTest.cs" %}
```csharp
public void TestNumericConversion()
{
	var version = new Version.Number(1, 2, 3, 4);
	const ulong Packed = 0x0001_0002_0003_0004UL;

	Assert.Equal(Packed, (ulong)version);
	Assert.Equal((long)Packed, (long)version);
	Assert.Equal(version, (Version.Number)Packed);
	Assert.Equal(version, (Version.Number)(long)Packed);

	version = new Version.Number(ushort.MaxValue, ushort.MaxValue, ushort.MaxValue, ushort.MaxValue);
	Assert.Equal(ulong.MaxValue, (ulong)version);
	Assert.Equal(-1L, (long)version);
	Assert.Equal(version, (Version.Number)(-1L));
}
```
{% endcode %}

The integer form is packed into 64-bit unsigned integers in sequence `Major`, `Minor`, `Patch`, `Revision` and four `ushort`, so the numerical sorting, version comparison and persistence results can be consistent. `Version.Number` also supports implicit conversion with `long`, `ulong`, and `System.Version` to facilitate compatibility with old data and .NET standard version types.

| Ability | Description |
| --- | --- |
| `IsZero` | Determine whether it is `0.0.0.0`. |
| `Parse(...)` / `TryParse(...)` | Parse from two, three or four paragraphs of numeric text. |
| comparison operator | Support `==`, `!=`, `>`, `>=`, `<`, `<=`. |
| integer conversion | Can be implicitly converted to and restored from `ulong`, `long`. |
| `System.Version` conversion | Can be interchanged with `System.Version`; when there is no revision number, the three segments `System.Version` are output. |
| JSON conversion | Writes out string form; supports string version number when reading, and also supports 64-bit integers. |

{% hint style="warning" %}
`Version.Number` The maximum value of each segment is `65535`. If a release requires a pre-release tag, build metadata, or a number field outside of this range, `Version` or a specialized release model should be used instead of cramming this information into four numbers.
{% endhint %}

## Migration Recommendations

If the old code only compares four-segment numbers like `1.2.3.4`, or relies on integer persistence, you usually only need to change the namespace and type to `Zongsoft.Versioning.Version.Number`.

The above TestNumericConversion is an integer round-trip compatibility use case; when migrating across old versions, use existing persistent values to verify each recovery result before deciding whether to adjust the field type.

If the old field originally saved a user-oriented release version and needs to express `alpha`, `beta`, `preview`, `rc` or build information, it is recommended to use `Version` instead, and clarify whether `Extra` needs to be written into the serialization result.

## Reference Implementation

* [Version.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Versioning/Version.cs)
* [Version.Number.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Versioning/Version.Number.cs)
* [VersionTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Versioning/VersionTest.cs)
* [VersionNumberTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Versioning/VersionNumberTest.cs)
