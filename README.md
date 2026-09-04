# Hidden Rewrite Systems: Sequence Transduction from Demonstrations

Documentation and provenance record for a synthetic dataset built to study a
specific question: **can you recover a rule system when you only ever see what
it did, never how it did it?**

**This repository is the canonical source record for the dataset.** The dataset
is original — not derived from, sampled from, or transformed from any existing
dataset, archive, repository or corpus.

## What the dataset contains

| | |
|---|---|
| Cases | 5,000 (4,014 train / 986 test) |
| Rules per case | 3 to 5, ordered by priority, **not disclosed** |
| Rule form | two-symbol pattern to a 1–3 symbol replacement |
| Alphabet | `{0, 1, 2, 3, 4}` |
| Demonstration pairs | 20–26 per case, 115,412 in total |
| Query sequences | 3–5 per case, 18,299 in total |
| Input length | 10–14 symbols |
| Normal form length | 6–18 symbols |
| Derivation depth | 2–4 rewriting steps, bound drawn per case, **not disclosed** |
| Size | 6.8 MB |
| Licence | CC0 1.0 Universal |

## Why it is interesting

Constrained sequence generation is well studied, but the constraints are handed
to the decoder. Work such as NeuroLogic decoding and its successors reports
near-total constraint satisfaction precisely because the rules are given. Here
they are not. The system that produced every output is withheld, and the only
evidence is 20–26 before-and-after pairs.

That gap matters because a demonstration shows two endpoints of a derivation
that took two to four steps. The intermediate states — and with them the record
of which rule fired where — are gone. Rules interact: one can manufacture the
pattern another consumes, so the visible change to a sequence is rarely the work
of any single rule, and candidate patterns cannot be read off the observed
inputs because rules routinely fire on substrings that never appear in them.

Scored as `0.60 x gain-over-copying + 0.40 x exact match`:

| Method | Score |
|---|---|
| Perfect | 1.0000 |
| Rule induction, 8s per case | **0.5075** |
| Rule induction, 3s per case | **0.3643** |
| Rule induction, 1.5s per case | 0.2623 |
| Deduplicate the input | 0.0133 |
| Replay the nearest demonstration's output | 0.0060 |
| Sort the input | 0.0003 |
| Shifted answers, random, constant, empty, echo the input | 0.0000–0.0002 |

Every shortcut that does not model the rewriting sits at zero. Gain is measured
relative to echoing the input, so there is no chance credit to bank, and partial
credit accrues only for genuinely moving a prediction toward the normal form.

## Identifiability

Grouping induction runs by how well each hypothesis reproduced the
demonstrations it was fitted on:

| Demonstrations reproduced | Cases | Query score |
|---|---:|---|
| all of them | 8 | **1.0000** |
| 95–99.9% | 24 | 0.6446 |
| 85–95% | 55 | 0.3906 |
| under 85% | 3 | 0.2394 |

Fitting every demonstration implies solving every query, so the task is well
posed and the score is a clean measure of how much of the system was recovered.
Difficulty rests on search hardness, not on ambiguity.

## Construction

Rewriting is deterministic. Scan positions left to right; at each position try
the rules in priority order; apply the first that matches; restart the scan.
Stop when nothing matches. The sequence that remains is the normal form, and
because position and rule are both chosen by fixed precedence it is a function
of the input — every query has exactly one correct answer.

Cases were kept only when every derivation terminated within the case's depth
bound, every query needed at least two rewriting steps, every rule a query
depends on fired in at least two demonstrations, no output equalled its own
input, and no query input appeared among the demonstration inputs.

A deeper depth bound accepts far more candidate sequences, so drawing the bound
uniformly would have skewed the finished set toward the hardest tier. The draw
is weighted to cancel that acceptance bias, giving 39.3% / 33.2% / 27.5% at
bounds 2 / 3 / 4.

## Files

| File | Contents |
|---|---|
| `DATASET_CARD.md` | full dataset card: semantics, columns, split, provenance |
| `VALIDATION_REPORT.json` | machine-readable integrity and baseline record |
| `LICENSE` | CC0 1.0 Universal |

## Provenance and licence

Entirely synthetic, generated from the fixed seed `20260906`. No third-party
data, no personal data. Re-running the generator reproduces the dataset byte for
byte; `train.csv` and `test.csv` hash to the values recorded in
`VALIDATION_REPORT.json`.

Released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

The underlying formalism — rewriting systems over strings, also called
semi-Thue systems — is standard; the classical reference is Book and Otto,
*String-Rewriting Systems* (Springer, 1993).
