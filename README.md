# The Unofficial Guide

*Dev Rajpuriya — Corpus: `campus_life`**

# Unit 1

## What This Does

The Unofficial Guide is a retrieval-augmented generation (RAG) system built
over the `campus_life` corpus. The corpus contains 88 documents covering
student-life topics such as housing, dining, courses, registration, parking,
laundry, and campus services.

When a user asks a question, the system searches the corpus for the most
relevant chunks and gives those chunks to the language model as context. The
model answers using the retrieved documents and names the source it used. If
the retrieved information is not relevant enough, the system refuses to answer
instead of guessing.

## Chunking Strategy

**Chunk size:** Soft maximum of about 500 characters

**Overlap:** One paragraph

The `campus_life` corpus contains mostly short posts. With the starter
800-character fixed-window chunker, the corpus produced 88 chunks from 88
documents. This showed that most documents were short enough that they were
not being split at all.

Most whole-document chunks were readable, but some longer posts contained
several different ideas in the same chunk. For example, a housing post could
contain room information, air conditioning, laundry, and noise together.

I replaced the fixed character-window strategy with paragraph-aware chunking.
The new strategy keeps paragraphs intact instead of cutting through sentences,
uses a soft target of about 500 characters, and keeps one paragraph of overlap
between neighboring chunks when a document has to be split.

After the change, the corpus produced 90 chunks with an average length of 311
characters. The shortest chunk was 178 characters and the longest was 461
characters. This was only two more chunks than the starter, so the documents
were not over-split, while the longest chunks became smaller and more focused.

## Sample Chunks

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer
window — through the end of week six — but a drop after week two shows as a W
on your transcript. Nothing anywhere on the registrar's site says this plainly,
and students find out from each other.

---

**Chunk 2** — source: `course_biol_160_exams.txt#0` — produced by: `chunker.py::split_documents`

BIOL 160 Cell Biology — assessment

Four unit tests and a cumulative final. Not curved.

The unit tests come fast, roughly every three weeks; falling behind once is very
hard to recover from.

---

**Chunk 3** — source: `course_math_220_exams.txt#0` — produced by: `chunker.py::split_documents`

MATH 220 Linear Algebra — assessment

Two midterms and a cumulative final. Curved to a B- median.

The problem sets are the course; the lectures make sense afterwards rather than
during.

---

**Chunk 4** — source: `dining_the_ridgeway_cafe.txt#0` — produced by: `chunker.py::split_documents`

The Ridgeway Café

Second-year here. Wait times: 10 to 15 minutes at 12:30, none after 2:00. The
thing worth going for is the only place on campus with real espresso. The thing
to know is that seating is tight; about 40 seats for a building of 900.

Hours are 7:00am to 4:00pm weekdays only. Costs declining balance only, no meal
swipes.

---

**Chunk 5** — source: `housing_morrow_house.txt#0` — produced by: `chunker.py::split_documents`

Morrow House — what it's actually like

Just finished a year in this building. Built 1954, partially renovated 2008.
Rooms are singles and doubles, hall bathrooms.

The good: cheapest housing tier by about $900 a year, and the singles are real
singles.

The bad: known damp problem on the ground floor; two rooms were taken offline
in 2024.

Laundry costs $1.50 wash, $1.25 dry, coin or card. On noise: loud until about
1am on weekends, no enforced quiet hours.

## Sample Answer

**Question:**

When is the best time to do laundry in Aldridge Hall?

**Answer:**

The best time to do laundry in Aldridge Hall is Tuesday or Wednesday morning.

Source: `housing_aldridge_hall_laundry.txt` (also mentioned in
`housing_aldridge_hall.txt` regarding laundry details, though the specific
timing comes from `housing_aldridge_hall_laundry.txt`).

**My relevance cutoff:** `0.6`

**Top-k:** `5`

I kept top-k at 5 because the correct source appeared as the first retrieval
result for all five of my in-corpus test questions. Increasing it was not
necessary for these tests.

I tested all five questions that the corpus should answer and all five
out-of-scope questions. Lower distance means the retrieved document is more
semantically similar to the question.

| Question | In corpus? | Best distance |
|---|---|---:|
| How are juniors and seniors prioritized in the housing lottery? | Yes | 0.2050 |
| How quickly do student parking permits for the west lots usually sell out? | Yes | 0.1856 |
| What must happen with my adviser before I can register for classes? | Yes | 0.4315 |
| When is the best time to do laundry in Aldridge Hall? | Yes | 0.3021 |
| How long are wait times at Halden Hall even at noon? | Yes | 0.2118 |
| What is the capital of Mongolia? | No | 0.8246 |
| How do I change the oil in a diesel engine? | No | 0.9340 |
| Who won the 1994 World Cup? | No | 0.8859 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8442 |
| How do I write a for loop in Rust? | No | 0.8960 |

The highest best-distance among the five questions the corpus should answer was
0.4315. The lowest best-distance among the five out-of-scope questions was
0.8246. This created a large gap between the two groups.

Because the existing cutoff of 0.6 falls safely inside that gap, I kept it
rather than changing it without evidence. With the 0.6 cutoff, all five
in-corpus questions passed the gate and all five out-of-scope questions were
refused.

For example, asking:

**Question:** Who won the 1994 World Cup?

returned:

> I don't have enough information about that.

I also inspected the grounding instruction sent to the model. It tells the
model to use only the provided documents, not guess when the documents do not
cover the question, and name the source document. The generated Aldridge Hall
answer followed those instructions, so I kept the existing grounding
instruction.

## How I Used AI

**1. Chunking strategy**

I used an AI assistant to help me understand the starter chunking logic and
compare alternative chunking strategies for the short `campus_life` documents.
The suggested approach was paragraph-aware chunking with a soft 500-character
target and one-paragraph overlap instead of fixed 800-character windows.

I implemented the approach and verified it against the actual corpus rather
than accepting the suggestion automatically. The starter produced 88 chunks
with a longest chunk of 549 characters. My implementation produced 90 chunks
with a longest chunk of 461 characters. I also manually reviewed five generated
chunks to confirm they remained understandable on their own.

**2. Relevance cutoff**

I used an AI assistant to help interpret the retrieval-distance results after
running five in-corpus questions and five out-of-scope questions.

The in-corpus distances ranged from 0.1856 to 0.4315, while the out-of-scope
distances ranged from 0.8246 to 0.9340. Based on this separation, I kept the
existing 0.6 relevance cutoff rather than changing it without evidence. I then
tested an unrelated World Cup question and confirmed that the relevance gate
refused it before a model call was made.

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
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks are understandable on their own | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Cited source actually supports the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

### Evidence from the before run

**Criterion 1 — retrieval contained the answer**

For the housing lottery question, retrieval returned:

`admin_housing_lottery.txt`

Best distance: `0.2050`

The generated answer was:

> Juniors and seniors are ordered by accumulated credit hours first, with ties broken randomly.
>
> Source: admin_housing_lottery.txt

The other four test questions also retrieved a document containing the correct answer.

**Criterion 2 — every answer named a source**

Example:
> You need to have your adviser hold lifted before you can register.


and compare them against the acceptance criteria I had already written. I used
the actual outputs from `run_eval.py` to make the final MET or MISSED decisions
rather than changing the original targets.

I also used the assistant to compare the top-k 5 and top-k 3 runs. I kept the
top-k 3 change because the correct source remained available for all five test
questions and all five out-of-scope questions continued to be rejected.

> Source: advising_registration.txt

All generated answers in the three runs named at least one source document.

**Criterion 3 — gate stopped out-of-scope questions**

The relevance gate refused all five out-of-scope questions:

- Capital of Mongolia — distance 0.825
- Diesel engine oil change — distance 0.934
- 1994 World Cup — distance 0.886
- Ibuprofen dosage — distance 0.844
- Rust for loop — distance 0.896

Result: `5 of 5 refused`

**Criterion 4 — sampled chunks were understandable on their own**

The five chunks inspected in Unit 1 remained complete standalone thoughts.
All five could answer a question without requiring the previous or next chunk.

Result: `5 of 5`

Produced by: `chunker.py::split_documents`

**Criterion 5 — cited source supported the answer**

Example:

> The best time to do laundry in Aldridge Hall is Tuesday or Wednesday morning.
>
> Source: housing_aldridge_hall_laundry.txt

The cited document directly contains the statement that the best time is
Tuesday or Wednesday morning.

All five test questions had citations that supported the generated answer.

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
| 1 | Retrieved chunks contain the answer | MET | All 5 test questions retrieved a chunk containing the needed answer in all three runs, exceeding the 4-of-5 target. |
| 2 | Every answer names a source | MET | All generated answers named at least one source document in all three runs. |
| 3 | The relevance gate stops out-of-corpus questions | MET | The gate refused all 5 out-of-scope questions, exceeding the 4-of-5 target. |
| 4 | Sampled chunks are understandable on their own | MET | All 5 sampled chunks remained readable as standalone thoughts, exceeding the 4-of-5 target. |
| 5 | The cited source actually supports the answer | MET | For all 5 test questions, the cited document contained information that directly supported the answer. |

## Diagnoses

No acceptance criterion was missed in the before test.

The system met all five original targets across the three runs. This suggests
that the original criteria were achievable for the selected `campus_life`
corpus and the five test questions.

Because every criterion passed, the criterion I would tighten is Criterion 1.
Instead of requiring the retrieved chunks to contain the answer for at least
4 of 5 questions, I would require all 5 of 5 questions to retrieve a chunk
containing the answer. The before test already achieved 5 of 5 consistently,
so the original 4-of-5 target appears conservative.
     Milestone 3. -->

