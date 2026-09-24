# IN5040 Assignment 1 — Working Plan and Tips

Esper 8.2.0. Weather streams from JFK and San Francisco, 2014-2018, 1826 tuples per stream.
This document is a plan, not a solution. No query here answers an assignment question.

Docs: <http://esper.espertech.com/release-8.2.0/reference-esper/html/index.html>
Online tryout: <https://esper-epl-tryout.appspot.com/epltryout/mainform.html>

---

## 1. What has to be delivered

One PDF. Contents, per the assignment text:

| Item | Needs query | Needs output | Needs prose |
|---|---|---|---|
| Q1 | yes | yes | assumptions only |
| Q2 | two queries, sliding and tumbling | yes | yes, two questions |
| Q3 | yes | yes | assumptions only |
| Q4 | yes | yes | assumptions only |
| Q5 | no | no | yes, pattern matching by hand |
| Q6 | yes | yes | assumptions only |
| Q7 | yes | yes | assumptions only |
| Q8 | yes | yes | assumptions only |

Expected result counts, given in the assignment. Use these as your test oracle.

| Question | Expected tuples |
|---|---|
| Q1 | 3 |
| Q3 | 5 |
| Q4 | 7 |
| Q6 | 1 |
| Q7 | 1 |
| Q2, Q8 | not stated |

---

## 2. Suggested order of work

The questions are ordered by difficulty on purpose. Do not jump ahead.

1. **Read first.** Esper reference chapter 2, sections 2.1 to 2.10 and 2.17. This is what the TA points at. It is short and it is the whole mental model.
2. **Q1.** Already done. Use it to confirm the toolchain works.
3. **Q2.** Windows. This is the concept gate for Q3 and Q8.
4. **Q3.** Same idea as Q2, but driven by the tuple timestamp instead of the tuple count.
5. **Q4.** Window plus an aggregate condition.
6. **Q5.** Pen and paper. No code. Do it after Q4 so you already have pattern intuition.
7. **Q6 and Q7.** Patterns, chapter 7.
8. **Q8.** Join of two streams. Do it last and do not touch the sleep.

Budget: Q2 and Q8 will take the longest. Q5 takes fifteen minutes.

---

## 3. The mental model you need before writing EPL

### 3.1 Insert stream and remove stream

Every statement produces two streams of rows.

- **Insert stream**, `newData` in the listener. Rows entering the result.
- **Remove stream**, `oldData`. Rows leaving the result, e.g. when a window expires an event.

The supplied `EventListener` only prints `newData[0]`. Two consequences:

- If a statement outputs several rows at once, **you only see the first one**.
- If a statement ever fires with `newData == null` (remove-stream-only output), it throws a `NullPointerException`.

If either bites you, that is a signal about your query shape, not a bug in your query logic. You are allowed to note it as an assumption, or to loop over `newData` yourself.

### 3.2 A view is a window

In Esper a data window is called a *view*, and it is attached to a stream with `#`.

```
select ... from SomeStream#length(10)
```

No view means "one event at a time, nothing retained". Aggregations without a view aggregate over **all events since statement start**, which is almost never what an assignment question means.

### 3.3 Sliding vs tumbling

| | Sliding | Tumbling (batch) |
|---|---|---|
| Fires | on every arriving event | once per full batch |
| Windows overlap | yes | no |
| Length-based | `#length(N)` | `#length_batch(N)` |
| Time-based | `#time(P)` | `#time_batch(P)` |
| External time | `#ext_timed(expr, P)` | `#ext_timed_batch(expr, P)` |

This table is the answer to most of Q2 and Q3. Learn it cold. Full list: chapter 14, *EPL Reference: Data Windows*.

### 3.4 Tuple-based vs time-based

"Tuple-based" (or length-based) counts events. "Time-based" measures time.

The data here is one tuple per day, so seven tuples and seven days *look* the same. They are not the same mechanism, and Q2 and Q3 are testing exactly that you know the difference. Q2 says tuple-based. Q3 says use the external timestamp.

### 3.5 System time vs external time

Esper's clock by default is wall-clock time of the JVM. Your tuples carry `timestamp`, an epoch value in milliseconds for a date in 2014-2018, and they arrive 5 ms apart.

So `#time(7 days)` measures seven real seconds-of-your-laptop, not seven days of weather. To measure in *data* time you need the externally-timed views, which take the timestamp expression as their first argument:

```
select ... from SomeStream#ext_timed(someTimestampField, 30 seconds)
```

`Assignment1.java` already sets `configuration.getCommon().getTimeSource().setTimeUnit(TimeUnit.MILLISECONDS)` for this reason. The hint comment in the file says `ext_timed`.

---

## 4. Per-question plan

For each question: what it is testing, what to decide, and the doc section to open. No queries.

### Q1 — filter
Testing: the basic select/filter model, sections 2.2 and 2.4.
Decide: nothing much. Note that the assignment asks for the *date*, and `timestamp` is an epoch long. Decide whether you output the raw long or format it, and state that choice as an assumption.
Status: done, 3 tuples. Good.

### Q2 — sliding vs tumbling, same question
Testing: that you understand why the two window types give different answers.
Decide:
- Which view types. One sliding, one tumbling, both tuple-based.
- How to express "the seven-day window's average temperature is above 82". A filter on individual tuples is not the same as a condition on the window's aggregate. Think about where that condition belongs: `where` runs per row, `having` runs on the aggregate.
- How to get `Start_date` and `End_date` out of a window. Look at the `first()` and `last()` aggregation functions, chapter 13.
Write-up: the sliding query considers every overlapping 7-tuple group; the tumbling one considers only disjoint groups. Answer *which* gives the higher maximum and *why* in terms of how many candidate windows each one evaluates. One or two sentences is enough, but the reason must be about window overlap, not about luck.
Doc: chapter 14 (data windows), chapter 13 (`first`, `last`), section 2.6 to 2.10.

### Q3 — weeks by external timestamp
Testing: external time and batching together.
Decide:
- The definition given is "week 1 is tuples 1-7, week 2 is tuples 8-14". That is a *tumbling* definition. But the question insists on the external timestamp. Reconcile these two before writing anything.
- Three or more inches of precipitation across the week is an aggregate over the batch.
- Note the data has no gaps, so a seven-day external-time batch and a seven-tuple batch coincide here. Say so as an assumption; it shows you know why it matters.
Doc: `#ext_timed_batch` in chapter 14.

### Q4 — three-day period, rain plus wind
Testing: a window with two conditions of different kinds.
Decide:
- Window length 3, sliding or tumbling. "Any three days in a row" is the phrase to reason about. Compare against the 7-tuple answer to make sure you pick the one that yields 7 tuples.
- "average wind speed exceeding 19 mph" — is that every day in the window, or the window's average? Read the sentence again and commit to one reading. Write it down as an assumption either way.
- "includes rain" — only somewhere in the three days, not every day. That is an aggregate condition, not a per-row filter.
- **How you define rain matters.** See section 6.3 below before you decide.
Doc: chapter 13 aggregation functions, `having` in chapter 5.

### Q5 — pattern semantics, on paper
Sequence: `A1 C1 A2 B1 D1 A3 B2 C2 B3 C3 C4 A4 B4`

Testing: whether you understand that `every` restarts a subexpression, and that where you put it changes how many sub-expressions are alive at once.

Method that works. For each of the four patterns, walk the sequence left to right and maintain a list of *active subexpression instances*. At each event ask: does this event advance an active instance, complete one, or start a new one? Write the list out at each step. Then read off the matches.

Reference points from chapter 7:
- `every A -> B` — each A starts its own search for a B.
- `A -> every B` — one A only, then every subsequent B matches.
- `every A -> every B` — the combinations multiply.
- `every (A -> B)` — the whole pair restarts, so a new A is only looked for after the previous pair completed.

Sanity check: the four answers should not all be the same length. If two of yours come out identical, re-walk them. Your answer should list the concrete matches, e.g. `(A2, B1)`, not just describe the semantics. You can verify on the online tryout, but do it by hand first — this is an exam-style question.

### Q6 — 40 degree swing within seven days
Testing: a followed-by pattern with a correlated filter and a time guard.
Decide:
- Two tagged events in a pattern. The second one's filter refers to the first one's property. Syntax for that is `b=Type(prop > a.prop)`.
- "40 degrees or more difference" is absolute. Think about whether one comparison covers both directions or whether you need `or`.
- "not more than seven days apart" — seven days of *data* time, not wall clock. `timer:within` uses engine time, which here is wall clock. So think about whether the constraint belongs in the guard or in the filter expression as arithmetic on `timestamp`. This is the trap in this question.
- `every` placement determines whether you get one match or many. The expected answer is 1 tuple, which is a strong hint about the right shape.
Doc: chapter 7, especially followed-by, tagged events, and guards.

### Q7 — three-day escalation
Testing: chaining three tagged events with conditions relative to the previous one.
Decide:
- Three tags, two followed-by steps, each step comparing to the tag before it.
- "consecutive days" must be enforced. The pattern operator `->` means "sometime later", not "the very next event". Decide how you force adjacency: a timestamp condition between successive tags, or a different construct.
- `#length(3)` with `prev()` is an alternative route. `prev(n, field)` reads the n-th previous event *within a data window*. Consider which formulation you find clearer, and be ready to justify it orally.
Doc: chapter 7 for the pattern route, chapter 10 `prev`/`prior` for the window route. Note `prior` works without a data window, `prev` does not.

### Q8 — join across the two streams
Testing: joins, and the fact that a join needs retained state on both sides.
Decide:
- Both streams need a view. A join between two streams with no window has nothing to match against.
- Week definition is the same tumbling 7-tuple definition as Q3. Both sides need it.
- Join condition: which weeks of one stream pair with which weeks of the other. Think about what makes two batches "the same week". Timestamp equality of the batch boundary is one route.
- Three conditions: temperature difference over 35, rain at JFK, no rain at SF. Each is an aggregate over its own batch.
- **Leave `Thread.sleep` in.** Without it, one file's thread can race far ahead of the other, and the two streams' batches stop lining up. The assignment says this explicitly.
Doc: chapter 5, join section. Read the part about why each stream needs a data window.

---

## 5. Workflow

```
make all          # compile
make run3         # run query_3.epl
make run3 > out/q3.txt 2>&1   # capture output for the PDF
make clean
```

Practical loop:

1. Write the query in the `.epl` file. **No trailing semicolon.**
2. `make all && make runX`.
3. Count output lines. Compare with the expected count.
4. When correct, save the output to a file. Do not retype results into the PDF by hand.

Speed. `SLEEP_TIME = 5` ms times 1826 tuples is roughly 9 seconds per stream. That is tolerable. You may lower it or comment out the sleep while iterating on Q1 to Q7, but restore it for Q8.

Add a `run5`-style target to the Makefile only if you need it. There is no `run5` because Q5 has no query.

Suggested extra targets:

```
out:
	mkdir -p out
```

Keep every run's output. You will want to diff after a change.

Compile errors from Esper are verbose but they point at a column number in your EPL. Read the first line of the exception, not the stack trace.

---

## 6. Traps in this specific dataset and codebase

### 6.1 The CSV files are CRLF
Every line ends `\r\n`, and `StringTokenizer` puts the `\r` inside the **last** field. So `weather` is `"0\r"`, not `"0"`.

Consequence: an equality test against a literal will silently never match. If you filter on `weather`, either use a pattern match rather than equality, or trim the value. Verify with a throwaway `select weather from jfk` and look closely at the output.

### 6.2 Timestamps are epoch milliseconds, local midnight
`LocalDate.parse(...).atStartOfDay(ZoneId.systemDefault()).toEpochSecond() * 1000`.
So one day is exactly `86400000` except across DST boundaries, where it is not. Keep that in mind if you write day arithmetic on `timestamp` for Q6 or Q7. The dataset spans five years, so it crosses DST twenty times.

### 6.3 `weather` is not a boolean rain flag
It is a NOAA weather-type code string: concatenated digits, `0` when nothing was recorded. JFK shows values like `0`, `1`, `12`, `123`, `1238`, `18`. SF similar. A `1` anywhere means fog was recorded, not rain.

`precipitation` is inches and is the reliable quantity. San Francisco has 278 days with precipitation above zero out of 1826. Define "rain" as `precipitation > 0` and say so in your assumptions. If you define it off `weather`, say that too and be ready to defend it.

### 6.4 Both streams share one Java class
`WeatherTuple` is registered twice, as event type `jfk` and as event type `san_francisco`. Stream names in EPL are `jfk` and `san_francisco`. That is why Q8 can join them without a schema.

### 6.5 Threads are independent
The two files are read by two threads with no coordination. Their interleaving is not deterministic. This only matters for Q8 and it is exactly why the sleep must stay.

### 6.6 Output timestamp is not the data timestamp
The listener prints `System.currentTimeMillis()` first, then the tuple. The leading number in your output is wall clock, not weather data. Do not mistake it for `timestamp` when you paste results into the PDF.

---

## 7. EPL syntax cookbook, generic

Neutral examples only. Nothing here is a solution to any question. Adapt the shapes yourself.

**Filter and project**

```
select id, value as v from SensorStream where value > 100
```

**Sliding length window with an aggregate**

```
select avg(value) from SensorStream#length(10)
```

**Tumbling length window**

```
select avg(value) from SensorStream#length_batch(10)
```

**Condition on the aggregate, not on the row**

```
select avg(value) from SensorStream#length_batch(10) having avg(value) > 50
```

**Boundaries of the current window**

```
select first(ts) as w_start, last(ts) as w_end from SensorStream#length_batch(10)
```

**External time window, sliding and tumbling**

```
select sum(value) from SensorStream#ext_timed(ts, 30 sec)
select sum(value) from SensorStream#ext_timed_batch(ts, 30 sec)
```

**Reach back inside a window**

```
select value, prev(1, value) as previous from SensorStream#length(3)
```

`prior(1, value)` does the same by arrival order and needs no window.

**Pattern: followed-by with a correlated filter**

```
select a.id, b.id from pattern [every a=SensorEvent -> b=SensorEvent(value > a.value)]
```

**Pattern: guard**

```
select * from pattern [every a=SensorEvent -> b=AlarmEvent where timer:within(10 sec)]
```

`timer:within` uses engine time. In this assignment engine time is wall clock, not data time.

**Join of two windowed streams**

```
select a.id, b.id
from StreamOne#length(5) as a, StreamTwo#length(5) as b
where a.key = b.key
```

**Aliasing for the required output columns**

```
select first(ts) as Week_start, last(ts) as Week_end, sum(p) as Precipitation from ...
```

The assignment names the output attributes. Alias to exactly those names. It costs nothing and it makes the PDF readable.

---

## 8. Verification checklist

Before you put a question in the PDF:

- [ ] Output row count matches the expected count, where one is given.
- [ ] Output columns are named exactly as the assignment asks.
- [ ] Dates are readable, or you state that you output epoch millis.
- [ ] No trailing semicolon in the `.epl` file.
- [ ] You can say in one sentence *why* this window or pattern type and not the other one.
- [ ] Assumptions written down: rain definition, window type choice, day arithmetic, timezone.
- [ ] Output was captured from a run, not retyped.

For Q2 and Q8, where no count is given, cross-check differently: shrink the window, run, and confirm the result count moves in the direction you predict. If it does not, your query is not doing what you think.

---

## 9. Turning this into exam preparation

The oral exam will ask about concepts, not about your exact EPL. For each question, be able to answer:

- Why a window at all? What would happen without one?
- Sliding or tumbling, and what changes in the result set?
- System time or event time, and what breaks if you pick the wrong one?
- Why does this need a pattern rather than a window, or the other way round?
- Q8: why does a join need state on both sides, and what happens when the two streams are not synchronised?

Q5 is a pure concept question. If you can walk the `every` variants confidently on paper, you can answer most pattern questions on the spot.

Also be ready for: what is the difference between a DSMS and a DBMS, what is load shedding, what does it mean that a query is continuous, and why stream joins are bounded by windows. Those are standard IN5040 territory and this assignment is the concrete instance of all of them.

---

## 10. Reading list, in order

1. Chapter 2, sections 2.1 to 2.10 — the processing model. Read all of it.
2. Chapter 2, section 2.17 — basic EPL patterns.
3. Chapter 14 — data windows. Skim the table, read `length`, `length_batch`, `time`, `time_batch`, `ext_timed`, `ext_timed_batch`.
4. Chapter 7 — patterns. Read the `every` section twice.
5. Chapter 5 — clauses. The join section and the `output` clause.
6. Chapter 13 — aggregation functions. `first`, `last`, `window`.
7. Chapter 10 — `prev`, `prior`, `prevwindow`.

Chapter 8, *Match Recognize*, is an alternative to patterns for Q7. Optional, but worth a look if the pattern route fights you.
