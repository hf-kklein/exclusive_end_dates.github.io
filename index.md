# End Date(Time)s Should be Exclusive: Here's Why

When modeling time periods/frames/slices/ranges, their end should always be exclusive. This page collects good reasons for this convention.

## What are you talking about?

If you have a time frame, it is usually defined by a start and an end datetime. If your time frame starts at January 1st, then

```
start = "2021-01-01"
```

Inclusive start dates are common sense.

But often enough, it's unclear how to model the end of a time span.
If the time frame ends at the end of January, then the end date should be

```
end = "2021-02-01"
```

That the end date, the first moment in February, is not inside the time frame ("January") is referred to as **"_exclusive_ end date"**.

The opposite (modeling the end of January as `end = "2021-01-31"`) is an inclusive end date. But this comes with a lot of problems.

## Convention: `Duration = End - Start`

It's just convenient to use exclusive end dates, because

```
length_of_time_frame = end - start
```

| Convention               | Exclusive Result                      | Naive Inclusive Result               | Inclusive Workaround            |
|--------------------------|---------------------------------------|--------------------------------------|---------------------------------|
| `duration = end - start` | `2021-02-01 - 2021-01-01 = 31 days`✔️ | `2021-01-31 - 2021-01-01 = 30 days`❌ | `duration = end - start + ??`😒 |

When using inclusive end dates (which you shouldn't), you'll have to build workarounds, that for example, add or subtract single days, seconds, microseconds or ticks, when all you want is just the duration of a time slice.
This is just not convenient.

Basically, all programming languages follow this convention.

### Python

```python
from datetime import date

start = date(2021, 1, 1)
end = date(2021, 2, 1)
duration = end - start
print(f"The time between {start} and {end} is {duration.days} days")
# The time between 2021-01-01 and 2021-02-01 is 31 days
```

(_Pandas allows [to specify](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.date_range.html) if `date_range` start and end are open/exclusive or closed/inclusive._)

### C#

```c#
var start = new DateTime(2021, 1, 1, 0, 0, 0, DateTimeKind.Utc);
var end = new DateTime(2021, 2, 1, 0, 0, 0, DateTimeKind.Utc);
var duration = end-start;
Console.Out.WriteLine($"The time between {start:O} and {end:O} is {duration.TotalDays} days");
// The time between 2021-01-01T00:00:00.0000000Z and 2021-02-01T00:00:00.0000000Z is 31 days
```

### Go

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	start := time.Date(2021, 1, 1, 0, 0, 0, 0, time.UTC)
	end := time.Date(2021, 2, 1, 0, 0, 0, 0, time.UTC)
	fmt.Printf("The time between %v and %v is %v days", start, end, end.Sub(start).Hours()/24)
	// The time between 2021-01-01 00:00:00 +0000 UTC and 2021-02-01 00:00:00 +0000 UTC is 31 days
}
```

### TypeScript

```ts
const start = new Date("2021-01-01T00:00:00Z");
const end = new Date("2021-02-01T00:00:00Z");
var durationMilliSeconds = end.getTime() - start.getTime();
console.log(
  `The time between ${start} and ${end} is ${
    durationMilliSeconds / 1000 / 60 / 60 / 24
  } days`
);
// "The time between Fri Jan 01 2021 01:00:00 GMT+0100 (Mitteleuropäische Normalzeit) and Mon Feb 01 2021 01:00:00 GMT+0100 (Mitteleuropäische Normalzeit) is 31 days"
```

### SQL

```sql
SELECT date('2021-02-01')-date('2021-01-01');
-- 31
```

## Avoiding the Need for Manual Workarounds when Connecting Systems/Languages with Different Temporal Resolutions

If one language has the concept of dates without a time (like e.g. Java or Python) and you're connecting it to a system that only understands datetimes/timestamps (like e.g. C# (prior to `DateOnly`) or Typescript), then, with inclusive end dates there's the need to explicitly think about the conversion.

While an inclusive start date `start = 2021-01-01` naturally translates to a datetime `2021-01-01T00:00:00Z` the inclusive `end = 2021-01-31` will be interpreted as `2021-01-31T00:00:00Z` by default which is obviously wrong. An inclusive end date requires the developer to think about edge cases, while the exclusive `end = 2021-02-01` = `2021-02-01T00:00:00Z` requires no workarounds or adaptions at all. You'll find tons of edge cases to think about when using inclusive end dates, including nasty ones like: "What happens on days with switches of Daylight Saving Time (DST)? Do I need to add 23/25 hours after the UTC conversion or 24 without or maybe another second in years with a leap second? So just avoid it.

Generally, the different temporal resolution between systems/languages gives rise to uncertainties.
Even if you provided an inclusive end datetime with seconds, what if milli and microseconds are supported?
With inclusive end dates, there'll always be a little gap between two adjacent time slices whose width is determined by the resolution of the datetime type in the respective language or system.
You don't even need multiple systems for these edge cases to become relevant, they can already occur in the integration of the application with the underlying database/ORM.

## How to model 1 day / 0 day time slices?
When modeling time slices with the duration "1 unit" (be it 1day, 1second, 1 millisecond...) you'll have problems with inclusive ends.
Think of a contract which starts on `2024-01-01`.
Its duration is 1 day, so the inclusive end would be `2024-01-01`, too.

How can you differentiate between contracts with a duration of 1 day and 0 days?
Would a 0-day duration lead to `end` < `start` time slices, just so that you can still apply the odd `duration+1` logic from above?
You can literally see, how annoying it is, when the time slice does not obey the rule `end - start = duration`.
Just don't do it. Use exclusive ends.

## What about the Users and their Habits?
In spoken language, it's unusual to use midnight of the next day when speaking about end dates.
So for users it might be helpful to display end dates inclusively like `2021-01-31` or even `2021-01-31T23:59:59` but those kinds of helpful "guides to the eye" should be restricted to only happen in the UI.
The real data underneath should not work with inclusive end dates.

## Further Reading

- ["Exclusive end dates" on QED Code](http://qedcode.com/content/exclusive-end-dates.html)
