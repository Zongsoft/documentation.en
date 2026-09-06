---
description: "Understand epoch and elapsed time in Discussions attachment names."
icon: clock
---

# Timestamp


Timestamp calculates a point in time or elapsed time in the specified epoch. Discussions' attachment uploads use the cumulative number of days in the Millennium epoch as part of the filename, combined with a random suffix.

Source: [src/api/Controllers/FileController.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/api/Controllers/FileController.cs#L79) (excerpt; see source for context).

{% code title="FileController.cs" %}
```csharp
var infos = this.Accessor.Write(this.Request,
							  this.DataService.GetDirectory(id),
							  args => args.FileName = $"{Timestamp.Millennium.Epoch.GetElapsed().Days}-{Randomizer.GenerateString()}", cancellation);
```
{% endcode %}

This code is in UploadAsync: Request comes from the current HTTP request, the directory is determined by the FileService, and cancellation comes from the caller. It does not treat the number of days in the file name as a business primary key or access permission.

## Epoch, Units and Time Zone

Unix and Millennium are different starting points. When passing time values across systems, both the starting point and the unit must be agreed upon; simply passing a number is not enough to determine whether it represents seconds, milliseconds, days, or ticks. Time zone presentation should also be distinguished from persistence time conventions.

## The Difference with Business Audit Time

Audit fields like CreatedTime, ModifiedTime, etc. of Discussions are filled in by DataValidator using DateTime.Now. The attachment name uses the epoch day, which does not mean that the entire module has uniformly adopted Unix timestamps. See [Data Services](../../data/services.md) for audit strategy.

Methods and boundaries can be checked for [Timestamp source code](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/Timestamp.cs) and [Correspondence test](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Common/Timestamp.cs).
