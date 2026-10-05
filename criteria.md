code criteria.md# Acceptance criteria — The Unofficial Guide

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
I chose 4 out of 5 because the campus life corpus contins many short documents
covering related campus topics, so I expect retrieval to work well but not be
perfect. Allowing one miss gives me a realistic way to ideentify where semantic  
search may confuse similar topics without makng the target tooo easy.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
I chose every answer becoz source attributionn is a core requirement of a
grounded RAG system. If the system gives an answer from retrieved documents,
it should always be able to identify whch document supported that answer.

---

## 3. The relevance gate stops out-of-corpus questions

For at least 4 of my 5 out-of-scope questions, the relevance gate refuses the
question before a model call is made

At least 4 of 5 sampled chunks should read as complete thoughts and contain
enough context to understand the information without needing the previous or
next chunk.

**Why this target:**

I chose 4 out of 5 because semantic search may occasionally retrieve a document
that appears related even when the question is outside the corpus. The system
should reject most unsupported questions rather than passing weak evidence to
the model and risking an unsupported answer.
---

## 4. Something about your chunks

At least 4 of 5 sampled chunks should read as complete thoughts and contain
enough context to understand the information without needing the previous or
next chunk.



**Why this target:**

The campus_life documents are relatively short, and the starter produced 88
chunks from 88 documents, so many documents are already close to a useful
standalone size. I chose 4 of 5 because I want most chunks to preserve complete
ideas while allowing for an occasional document that may require splitting.

---

## 5. Your choice

For at least 4 of my 5 test questions, the source document named in the answer
must contain information that directly supports the answer given.

**Why this target:**

Simply displaying a filename does not prove that an answer is grounded. I chose
4 of 5 because I want the citation to be meaningful and verifiable, while
allowing one case where retrieval or generation may select a source that is
related but not strong enough to fully support the answer

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
