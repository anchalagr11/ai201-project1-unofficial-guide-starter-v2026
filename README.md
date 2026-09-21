# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

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

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

## Chunking Strategy

**Chunk size:** 400 characters (a ceiling, not a blind window — I split on
paragraph boundaries and pack whole paragraphs up to this limit)

**Overlap:** 0 characters

I picked those values by reading `campus_life` and running the starter chunker over it:
- The posts are 88 short reviews (median 305 characters, longest 549).
- Every post is a title line followed by blank-line-separated paragraphs.
- Each paragraph holds a distinct fact (the good, the bad, laundry cost, noise).
- My questions ask for one fact at a time (e.g., a laundry price, a build year).
- A chunk should hold one fact cleanly, not several diluted together.

**Why 400.** 
The chunk size of 400 made the most sense in case of campus_life:
- 76 of the 88 posts are already under 400 characters, so most posts stay whole: one post, one chunk, with the title and the fact together. That was my Milestone 1 finding and I want to keep it. 
- The 12 posts that go over 400 characters pack 3 to 5 separate facts into one body. If I leave them whole, a question about the laundry price gets a chunk that's mostly about heating and noise. Splitting those on the blank lines gives each fact its own chunk.
- 400 is also the same limit my criterion 4 checks for, so I set the ceiling to match it rather than pick an unrelated number.


<!-- Milestone 3: code lives in chunker.py::split_documents. -->

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_cs_210.txt#0` — produced by: `chunker.py::split_documents`

```
CS 210 Data Structures

I'm a junior and I've done this twice now. Format is lecture with weekly labs; slides go up after class, not before. Assessment: two midterms and a final, all drawn from lecture material rather than the textbook. Midterms are curved, the final is not.

Expect 8 to 10 hours a week outside class.
```

**Chunk 3** — source: `course_math_220_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for MATH 220 Linear Algebra

People keep asking so: 6 to 8 hours a week, almost all of it on problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_the_ridgeway_cafe_followup.txt#0` — produced by: `chunker.py::split_documents`

```
Re: The Ridgeway Café

Adding to what people have said about The Ridgeway Café. The wait figure of 10 to 15 minutes at 12:30 matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: seating is tight; about 40 seats for a building of 900. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_morrow_house.txt#0` — produced by: `chunker.py::split_documents`

```
Morrow House — what it's actually like

Just finished a year in this building. Built 1954, partially renovated 2008. Rooms are singles and doubles, hall bathrooms.

The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.
```

## Sample Answer

**Question:** How much does laundry cost in Aldridge Hall?

**Answer:**

```
(best distance 0.247, cutoff 0.6)

In Aldridge Hall, laundry costs $1.75 for a wash and $1.50 for a dry.

Source: housing_aldridge_hall.txt (also mentioned in housing_aldridge_hall_laundry.txt).

Sources retrieved: housing_aldridge_hall.txt, housing_aldridge_hall_laundry.txt, housing_calder_annexe.txt, housing_innisfree_hall.txt, housing_old_brewhouse.txt
```

The prompt for this answer included four *other* buildings' laundry costs as
context (Innisfree, Old Brewhouse, Calder), and the model still gave Aldridge's
number and cited an Aldridge file. I kept `GROUNDING_INSTRUCTION` in
`generate.py` unchanged: I asked the same laundry question of three different
buildings whose costs all differ, and each answer stayed on the right building's
document, so answers weren't drifting past the sources.

**My relevance cutoff:** 0.6 (top_k = 5)

I ran all five of my questions and the five OUT_OF_SCOPE ones through retrieval
and recorded the best (rank-1) distance for each. The two groups don't overlap
at all: everything my corpus covers lands between 0.25 and 0.37, and everything
from a different world lands between 0.82 and 0.92. The gap between the two runs
from 0.373 to 0.825, with nothing inside it.

I kept 0.6 because it sits almost dead centre in that gap (the midpoint is
0.599). That leaves 0.23 of room above my hardest real question and 0.23 below
my closest off-topic one, so a real question that's a bit harder, or an
off-topic question that shares a bit more vocabulary, would both still be
sorted correctly. A cutoff near 0.3 would start refusing questions I have the
answer to (my worst real question is already 0.373); one near 0.9 would let the
diesel-engine and World-Cup questions through and the model would make something
up.

| Question | In corpus? | Best distance |
|---|---|---|
| How much does laundry cost in Aldridge Hall? | Yes | 0.247 |
| When do you declare a major? | Yes | 0.286 |
| What is the weekly workload for BIOL 160? | Yes | 0.312 |
| What is the printing quota per semester? | Yes | 0.341 |
| When does Halden Hall close? | Yes | 0.373 |
| What is the capital of Mongolia? | No | 0.825 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.844 |
| Who won the 1994 World Cup? | No | 0.886 |
| How do I write a for loop in Rust? | No | 0.891 |
| How do I change the oil in a diesel engine? | No | 0.923 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**

**2.**

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
