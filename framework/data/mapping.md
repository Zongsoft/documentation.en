---
description: "Explanation mapping with Forum composite key, topic body relationship and sequence number configuration of Discussions."
icon: table
---

# Mapping Files


.mapping maps the domain model to the database structure. It declares tables, keys, fields and navigation relationships, and can be delivered with business plugins; it does not create a database itself, nor perform migrations. The mapping for Discussions is placed in a single Zongsoft.Discussions.mapping, and the container name is Discussions.

## The Forum Number Belongs to the Site

The following is the definition of the key and number attributes within the Forum entity. See the source file for peripheral entity declarations and other attributes.

The following is the definition of the key and number attributes within the Forum entity. See the source file for peripheral entity declarations and other attributes.

Source: [src/Zongsoft.Discussions.mapping](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.mapping#L197) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.mapping" %}
```xml
<key>
	<member name="SiteId" />
	<member name="ForumId" />
</key>

<property name="SiteId" type="uint" nullable="false" />
<property name="ForumId" type="ushort" nullable="false" sequence="#(SiteId)" />
```
{% endcode %}

The Forum key consists of SiteId and ForumId. ForumId uses an external serial number divided by SiteId. Forums with the same ForumId on another site cannot be regarded as the same object. The entity qualified name is Discussions.Forum and the physical table name is Discussions_Forum.

## How to Cite the Forum and Text in a Topic

Source: [src/Zongsoft.Discussions.mapping](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.mapping#L278) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.mapping" %}
```xml
<complexProperty name="Forum" port="Forum">
	<link port="SiteId" />
	<link port="ForumId" />
</complexProperty>
```
{% endcode %}

The link.port of the relationship points to the target attribute, and the anchor refers to the current entity attribute. The above relationship is located in the ForumUser entity, see the source file for the complete context; the Forum navigation with the same name also exists in the topic. The composite key relationship should connect both site and forum numbers.

Source: [src/Zongsoft.Discussions.mapping](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.mapping#L330) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.mapping" %}
```xml
<complexProperty name="Post" port="Post" multiplicity="!">
	<link port="PostId" />
</complexProperty>
```
{% endcode %}

The topic's Post is a required body post. ThreadService.OnInsert will include the Post navigation into the writing mode so that the model relationship can be handed over to the engine for processing; simply adding an attribute to the model will not automatically generate a database relationship.

## Scalars, Default Values, and Immutable Fields

Source: [src/Zongsoft.Discussions.mapping](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.mapping#L293) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.mapping" %}
```xml
<property name="ThreadId" type="ulong" nullable="false" sequence="#" />
<property name="SiteId" type="uint" nullable="false" immutable="true" />
<property name="ForumId" type="ushort" nullable="false" />
<property name="Title" type="nvarchar" length="50" nullable="false" />
<property name="Acronym" type="varchar" length="50" nullable="true" />
<property name="Summary" type="nvarchar" length="500" nullable="true" />
<property name="Tags" type="nvarchar" length="100" nullable="true" />
<property name="PostId" type="ulong" nullable="false" />
```
{% endcode %}

String length, nullability and database type should be checked with the model and SQL at the same time. immutable is used to constrain the writing phase and cannot replace user authorization. The serial number mark # requires an external serial number service; the serial number divided by site also depends on the SiteId to be correctly assigned first, see [Sequence Generators](../core/common/sequence.md).

## Four SQL Scripts Are Not Automatically Compatible with Promises

The database directory of Discussions contains MySQL, SQL Server, PostgreSQL, and ClickHouse scripts. The Message in the current mapping also explicitly specifies the ClickHouse driver, so when selecting a different database you must also check the entity driver tags, data source selection, and SQL structure. You cannot just change the connection string and declare the switch complete.

## Expansion and Verification

The multiplicity of complex attributes distinguishes between optional single values, required single values, and sets. When cascading writing is required, immutable and behaviors should be clearly reviewed; the default value must be [map loader](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/src/Metadata/Profiles/MetadataFileResolver.cs). XSD validation checks the format, and the actual driver validation checks the database semantics. The two cannot replace each other.

Discussions currently has no named SQL commands. This capability can continue to check the frame [MetadataCommandScriptorTest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Data/test/MetadataCommandScriptorTest.cs). Do not create another Answer or order statistics command for the forum.

Continue reading: [Data Schemas](schema.md), [Connection Configuration](connections.md), [write](writing.md).
