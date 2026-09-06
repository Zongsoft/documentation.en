---
description: "Responsibilities and main types of the Zongsoft.Common namespace."
icon: wrench
---

# Zongsoft.Common

`Zongsoft.Common` places the most common tool types and extension methods in the core class library, including basic capabilities such as type conversion, randomness, verification, sequence, time, string, URI and type alias.

## Main Responsibilities

* Provides enhanced versions of common tools such as `Convert`, `Randomizer`, `EnumUtility`.
* Provides extension methods such as string, array, date and time, type, URI, etc.
* Provides checksum, sequence number, alias set, bit vector and annotation tools.
* Reduce scattered dependencies on duplicate gadget classes for upper-level modules.

## Type

<table data-view="cards">
	<thead>
		<tr>
			<th></th>
			<th></th>
			<th data-hidden data-card-target data-type="content-ref">Page</th>
		</tr>
	</thead>
	<tbody>
		<tr><td><strong>AggregateExceptionUtility</strong></td><td>Specific exception handling in aggregate exceptions.</td><td><a href="common/aggregate-exception-utility.md">aggregate-exception-utility.md</a></td></tr>
		<tr><td><strong>AnnotationUtility</strong></td><td>Read member classification, display name and description annotation.</td><td><a href="common/annotation-utility.md">annotation-utility.md</a></td></tr>
		<tr><td><strong>BitVector</strong></td><td>32-bit and 64-bit bit flags.</td><td><a href="common/bit-vector.md">bit-vector.md</a></td></tr>
		<tr><td><strong>Buffer</strong></td><td>Memory leasing, encoding, decoding and binary reading.</td><td><a href="common/buffer.md">buffer.md</a></td></tr>
		<tr><td><strong>Checksum</strong></td><td>Hash check value calculation, parsing and verification.</td><td><a href="common/checksum.md">checksum.md</a></td></tr>
		<tr><td><strong>Convert</strong></td><td>Type conversion and hexadecimal conversion.</td><td><a href="common/convert.md">convert.md</a></td></tr>
		<tr><td><strong>EnumUtility</strong></td><td>Enumeration item metadata, aliases, and descriptions.</td><td><a href="common/enum-utility.md">enum-utility.md</a></td></tr>
		<tr><td><strong>HierarchyVector32</strong></td><td>Four-level hierarchical coding and parent-child relationship judgments.</td><td><a href="common/hierarchy-vector32.md">hierarchy-vector32.md</a></td></tr>
		<tr><td><strong>Locker</strong></td><td>Synchronous and asynchronous mutex locks.</td><td><a href="common/locker.md">locker.md</a></td></tr>
		<tr><td><strong>Notification</strong></td><td>Change tokens and immediately expired tokens.</td><td><a href="common/notification.md">notification.md</a></td></tr>
		<tr><td><strong>OperationException</strong></td><td>Operation exception with reason code.</td><td><a href="common/operation-exception.md">operation-exception.md</a></td></tr>
		<tr><td><strong>Predication</strong></td><td>Asynchronous conditional assertions, assertion base classes, and composite collections.</td><td><a href="common/predication.md">predication.md</a></td></tr>
		<tr><td><strong>Randomizer</strong></td><td>Random bytes, numbers and string generation.</td><td><a href="common/randomizer.md">randomizer.md</a></td></tr>
		<tr><td><strong>Sequence</strong></td><td>Serial number interface and number segment extender.</td><td><a href="common/sequence.md">sequence.md</a></td></tr>
		<tr><td><strong>Timer</strong></td><td>Periodic task timer.</td><td><a href="common/timer.md">timer.md</a></td></tr>
		<tr><td><strong>Timestamp</strong></td><td>Timestamp conversion.</td><td><a href="common/timestamp.md">timestamp.md</a></td></tr>
		<tr><td><strong>TypeAlias</strong></td><td>Type alias resolution and generation.</td><td><a href="common/type-alias.md">type-alias.md</a></td></tr>
		<tr><td><strong>Validators</strong></td><td>Synchronous and asynchronous data validity verification interface.</td><td><a href="common/validator.md">validator.md</a></td></tr>
		<tr><td><strong>Common Extensions</strong></td><td>Array, string, time, type, and URI extensions.</td><td><a href="common/extensions.md">extensions.md</a></td></tr>
	</tbody>
</table>

## Related Resources

* [Common source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Common)
