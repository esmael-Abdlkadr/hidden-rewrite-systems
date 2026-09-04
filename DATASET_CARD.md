# Hidden Rewrite Systems: Sequence Transduction from Demonstrations

## Overview

Each of the 5,000 cases in this dataset hides a small **string-rewriting system**
— an ordered list of rules that replace one short pattern of symbols with
another. The system is never shown. What is shown is a set of demonstration
pairs: sequences before rewriting, and the same sequences after rewriting has
run to completion. The task is to reproduce the same transduction on query
sequences the demonstrations do not cover.

Every case has its own private rule set, so nothing transfers between cases by
memorisation. A case is solved by working out, from that case's demonstrations
alone, what the rules must be.

## The rewriting semantics

Sequences are lists of integer symbols drawn from the alphabet `{0, 1, 2, 3, 4}`.

A rule rewrites a two-symbol pattern into a replacement of one, two, or three
symbols. A rule's replacement is never identical to its pattern. Each case has
between three and five rules, held in a fixed priority order.

Rewriting a sequence proceeds deterministically:

1. Scan positions from left to right. At each position, try the rules in
   priority order.
2. Apply the first rule whose pattern matches at that position, then restart the
   scan from the beginning of the new sequence.
3. Stop when no rule matches at any position. The sequence that remains is the
   **normal form**.

Because the position and the rule are both chosen by a fixed precedence, the
normal form is a function of the input: every query has exactly one correct
answer. All cases in this dataset terminate.

The generator kept only cases where:

- every demonstration and query derivation completes in at most 2, 3, or 4
  rewriting steps. The bound is drawn per case, so the set spans a real
  difficulty range. Because a deeper bound accepts far more candidate
  sequences, drawing the bound uniformly would have skewed the finished set
  toward the hardest tier; the draw is weighted to cancel that acceptance bias,
  giving 39.3% / 33.2% / 27.5% at bounds 2 / 3 / 4;
- every query needs at least two rewriting steps, so no query is answered by
  spotting a single substitution;
- every rule a query depends on fires in at least two demonstrations, so the
  evidence needed to recover it is genuinely present;
- no output equals its own input, and no query input appears among the
  demonstration inputs.

## Files

| File | Rows | Contents |
| --- | --- | --- |
| `train.csv` | 4,014 | `case_id`, `demo_inputs`, `demo_outputs`, `query_inputs` |
| `train_labels.csv` | 4,014 | `case_id`, `query_outputs` |
| `test.csv` | 986 | `case_id`, `demo_inputs`, `demo_outputs`, `query_inputs` |
| `sample_submission.csv` | 986 | `case_id`, `query_outputs` |

Test answers are held privately in `answer.csv`, which carries `case_id`,
`query_inputs` and `query_outputs` — the grader needs the inputs to measure
skill relative to echoing them.

Every sequence-valued column holds a JSON array. `demo_inputs` and
`demo_outputs` are arrays of 20–26 sequences, aligned by position:
`demo_outputs[k]` is the normal form of `demo_inputs[k]`. `query_inputs` holds
3–5 sequences per case, and `query_outputs` must hold their normal forms in the
same order. Input sequences are 10–14 symbols long; normal forms run from 6 to 18
symbols, so rewriting both shortens and lengthens sequences.

`case_id` is an opaque hash. It encodes nothing about the case — not the rules,
not the difficulty, not the position in the file — and sorting on it carries no
signal.

## Split

Cases are assigned to train or test by hashing the opaque `case_id`, which puts
19.7% of cases in the test split. The split is a property of the case, so it
does not move if the files are reordered. Rule sets are sampled independently
per case; no rule set is shared across the split.

## Provenance and licence

The dataset is entirely synthetic, produced by `generate.py` from the fixed seed
`20260906`. No third-party data is included and no personal data is involved.
Re-running the generator reproduces the dataset bit for bit.

Released under **CC0 1.0 Universal (Public Domain Dedication)**,
<https://creativecommons.org/publicdomain/zero/1.0/>.

The underlying formalism — rewriting systems over strings, also called
semi-Thue systems — is standard; the classical reference is Book and Otto,
*String-Rewriting Systems* (Springer, 1993).
