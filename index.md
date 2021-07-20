# End Dates Shuold be Exclusive: Here's why

When modelling time periods/frames/slices/ranges their end should always be exclusive. This page collects good reasons for this convention.

## What are you talking about?

If you have a time frame, it is usually defined by a start and an end datetime. If your time frame starts at January 1st, then

```
start = 2021-01-01
```

Inclusive start dates are common sense.
If the time frame ends at the end of January, then the end date should be

```
end = 2021-02-01
```

That the end date, the first moment in February, is not inside the time frame ("January") is refered to as "_exclusive_ end date".

The opposite is an inclusive end date (e.g `end=2021-01-31`). But this comes with a lot of problems.

## Convenience

It's just convenient to use exclusive end dates:

```
length_of_time_frame = end-start
```

| Convention               | Exclusive Result                      | Naive Inclusive Result                | Inclusive Workaround            |
| ------------------------ | ------------------------------------- | ------------------------------------- | ------------------------------- |
| `duration = end - start` | `2021-02-01 - 2021-01-31 = 31 days`✔️ | `2021-01-31 - 2021-01-01 = 30 days`❌ | `duration = end - start + ??`😒 |

Adding or substracting single days, seconds or ticks, when all you want is just the duration of a time slice is not convenient. 

## Everyone else does so
Barely anyone uses inclusive end dates.

### Python

```python

```
### C#
```c#
var start = new DateTime(2021, 1, 1, 0, 0, 0, DateTimeKind.Utc);
var end = new DateTime(2021, 2, 1, 0, 0, 0, DateTimeKind.Utc);
var duration = end-start;
Console.Out.WriteLine($"The time between {start:O} and {end:O} is {duration.TotalDays} days");
// The time between 2021-01-01T00:00:00.0000000Z and 2021-02-01T00:00:00.0000000Z is 31 days
```
([dotnet fiddle](https://dotnetfiddle.net/QpgTQW))