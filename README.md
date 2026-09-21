# The Unofficial Guide

Manas Maddi — corpus: `campus_life`

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This system answers questions about campus life at a made-up university, using
a corpus of ~88 short posts covering course workloads and exams, dining hall
wait times, housing (laundry, noise, room layout), and admin policies like
add/drop deadlines, parking permits, and pass/fail rules. It's built for
questions with a specific, checkable answer — "how long is the wait at Kestrel
Commons at lunch" rather than "which dining hall is best."

## Chunking Strategy

**Chunk size:** one document = one chunk (no character-count splitting)
**Overlap:** none

Every file in `campus_life` is a short, single-topic post — the longest is 554
characters and the average is about 320, both well under the starter's default
800-character `CHUNK_SIZE`. When I ran the original `fallback_split` in
Milestone 1, it produced 88 documents and 88 chunks with zero actual splitting,
so a fixed-size window strategy wasn't doing anything to begin with — it was
silently equivalent to "one chunk per document." I rewrote `split_documents` in
`chunker.py` to make that explicit instead of implicit: each `Document` becomes
exactly one `Chunk`, keeping the post's full thought intact rather than
manufacturing an arbitrary split that would never trigger on this corpus
anyway.

## Sample Chunks

`python app.py chunks -n 5` — chunks produced by `chunker.py::split_documents`.

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_biol_160.txt#0` — produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_hist_118_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_pellew_dining_hall_followup.txt#0` — produced by: `chunker.py::split_documents`

```
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_innisfree_hall.txt#0` — produced by: `chunker.py::split_documents`

```
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

## Sample Answer

**Question:** What percentage of the CS 210 grade do the labs count for?

**Answer:**

```
The labs count for 10% of the CS 210 grade.

Sources:
- `course_cs_210_exams.txt`
- `course_cs_210.txt`

Sources retrieved: course_cs_210.txt, course_cs_210_exams.txt, course_cs_340_exams.txt, course_phys_130.txt, course_stat_150_exams.txt
```

**My relevance cutoff:** 0.6 (kept the starter default)

I ran my 5 in-scope test questions and the 5 `OUT_OF_SCOPE` questions through
`python app.py retrieve` and recorded the best (rank-1) distance for each. The
two groups didn't just have a gap — they had a canyon: every in-scope question
came back under 0.37, and every out-of-scope question came back above 0.82,
with nothing in between. Since 0.6 sits comfortably in the middle of that gap
and correctly passed all 5 in-scope questions while refusing all 5 out-of-scope
ones, I didn't move it off the starter's default — there was no evidence it
needed adjusting for this corpus. Top-k stayed at the default of 5 too: the
correct chunk landed at rank 1 for every in-scope question I checked, well
separated from the next-closest (unrelated) result.

| Question | In corpus? | Best distance |
|---|---|---|
| Do unused dining dollars roll over from spring to autumn? | yes | 0.192 |
| What shows up on a transcript after dropping a course past week two? | yes | 0.304 |
| What percentage of the CS 210 grade do labs count for? | yes | 0.368 |
| How quickly do west lot parking permits sell out in August? | yes | 0.189 |
| Best time to do laundry in Aldridge Hall to avoid a wait? | yes | 0.355 |
| What is the capital of Mongolia? | no | 0.825 |
| How do I change the oil in a diesel engine? | no | 0.934 |
| Who won the 1994 World Cup? | no | 0.886 |
| Recommended dosage of ibuprofen for a headache? | no | 0.844 |
| How do I write a for loop in Rust? | no | 0.896 |

## How I Used AI

**1.** I wrote my own first draft of `split_documents` in `chunker.py` for
Milestone 3, then asked Claude to look it over instead of asking it to write
the function. It caught three real bugs I hadn't seen: `chunk_size`/`overlap`
were referenced but never defined anywhere in the function, `chunks =
list[Chunk]` assigned the *type* `list[Chunk]` to the variable instead of an
empty list (so `.append()` didn't exist on it), and I was calling
`chunks.append(text=..., source=..., ...)` with keyword arguments when
`append()` only takes one positional value — I needed to build a `Chunk(...)`
object first. I fixed each one by hand rather than pasting in a rewritten
function, and had it run `python chunker.py` afterward to confirm it produced
88 chunks (one per document) instead of erroring out.

**2.** For Milestone 4 I asked Claude to run my five test questions plus the
five `OUT_OF_SCOPE` questions through `python app.py retrieve` and report the
best distance for each, rather than have it guess at what a reasonable cutoff
would be. It came back with two cleanly separated groups (in-scope questions
all under 0.37, out-of-scope all over 0.82) and suggested keeping the starter's
default `THRESHOLD = 0.6` since it already sat in that gap. I didn't just take
the number — I checked the actual retrieval output myself (the `admin_*.txt`
and `course_cs_210*.txt` files it named are real and are the correct sources
for those questions) before writing that reasoning into the README.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
