# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
My questions ask for specific facts like prices, hours, and workloads. I expect exact factual retrieval to be highly successful, but allowing 1 failure accounts for edge cases where the embedding model might miss the semantic link.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
This is a structural requirement of our prompt. The prompt explicitly instructs the model to cite sources based on the context provided, so it should be achievable 100% of the time unless the prompt format breaks.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**
There should be a clear distance gap between relevant campus queries and completely unrelated ones. A target of 4 out of 5 is realistic, leaving room for the possibility that one random question might happen to share semantic overlap with campus terminology.

---

## 4. Something about your chunks

At least 4 of 5 sampled chunks are shorter than 400 characters and contain information from only a single document/post.

**Why this target:**
Because the `campus_life` documents are very short, dense posts (averaging around 317 characters), using larger chunks would merge multiple distinct topics into one chunk, making retrieval noisy and imprecise.
---

## 5. Your choice

For at least 4 of my 5 test questions, the named source document actually contains the specific facts used in the answer, rather than just being topically related.

**Why this target:**
Source attribution isn't useful if the model is hallucinating or misattributing facts. Getting 4 out of 5 correct is a realistic target for an LLM that might occasionally synthesize information from its prior knowledge.
---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
