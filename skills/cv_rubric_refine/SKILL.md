---
name: cv_rubric_refine
version: 0.1.1
description: >
  Propose refinements to the CV review rubric from a batch of
  accumulated discrepancy reports. Aggregates recurring gaps across
  reports produced by cv_calibration_eval, drafts specific SKILL.md
  diffs with provenance and confidence, and runs a golden-snapshot
  regression check to catch unintended score drift. Outputs a
  proposed-changes document for human approval — NEVER auto-edits any
  rubric SKILL.md. Run deliberately in batch (cold-start trigger: first
  5-6 accumulated CVs), not per-CV.
changelog:
  - version: 0.1.1
    date: 2026-06-11
    changes: >
      Pinned model updated claude-sonnet-4-20250514 (retiring 2026-06-15)
      -> claude-opus-4-8 in the golden snapshot format. Deliberate full
      re-baseline event per the anti-drift controls (CLAUDE.md S7, issue #22).
  - version: 0.1.0
    date: 2026-06
    changes: >
      Initial draft. Defines batch aggregation of discrepancy reports,
      cold-start vs mature refinement modes, the golden-snapshot
      regression harness and snapshot format, the noise-floor tolerance
      band, and the human approval gate. Consumes the discrepancy report
      schema defined in cv_calibration_eval.
---

# CV Rubric Refine (Human-Gated, Batch)

## Purpose

Turn accumulated calibration evidence into deliberate, low-risk
improvements to the CV review rubric — without silent score drift.

The philosophy: rubric changes are rare, batched, and human-approved.
The calibration history of `cv_review_generic` is a record of score
inflation caused by rubric changes; the controls in this skill exist
to prevent a repeat. This skill **proposes** changes and **measures**
their impact. It does not apply them.

**This skill never edits a rubric SKILL.md.** Its output is a
proposed-changes document. A human reviews it; an approved change is
applied separately, with a version bump per CLAUDE.md §9.

---

## When to Use This Skill

- **Cold-start trigger:** the first run happens once **5-6 CVs** have
  accumulated discrepancy reports — not per CV. This lets precedents
  build before any generalization.
- Thereafter: in deliberate batches, not on every new CV.
- After a model upgrade, to re-baseline the golden snapshot (see below).

---

## Inputs

| Input | Required | Description |
|---|---|---|
| Discrepancy reports | Yes | `cv_samples/reports/*_eval.md` (from `cv_calibration_eval`) |
| Rubric under refinement | Yes | usually `cv_review_generic/SKILL.md` |
| Golden snapshot | Yes | `cv_samples/snapshot.md` (baseline scores; format below) |
| Batch label | Optional | e.g. "cold-start batch 1 (CV_1–CV_6)" |

All inputs and outputs live in the gitignored `cv_samples/` tree —
they are derived from PII and must never be committed.

---

## Refinement Mode (depends on corpus size N)

The discipline changes with how many CVs the corpus holds. State which
mode applies at the top of every run.

| | Cold start (N small — current) | Mature (N >~10) |
|---|---|---|
| Mental model | Case law — each CV is a precedent; every observation counts | Statistics — require corroboration before generalizing |
| New-rule scope | Narrow conditionals only; no sweeping rules | Generalize once a pattern recurs |
| Corroboration | Single-source rules allowed | Require ≥2 supporting examples |
| Provenance | Every rule tagged `source: CV_N, confidence: low/med/high` | Confidence promoted/demoted as evidence accumulates |
| Held-out set | None — all CVs used as calibration (accepted tradeoff) | Carve out a held-out eval set never seen by refinement |

**In-sample weighting:** a hypothesis sourced only from an `in-sample`
CV (one the rubric was built from) is weak evidence — the rubric
already "knows" that CV. Weight `out-of-sample` hypotheses higher.

---

## Step 1 – Aggregate Hypotheses

Read every discrepancy report in the batch. Collect all
**Improvement Hypotheses** rows (Step 4 of `cv_calibration_eval`).
Group them by target dimension and by observation.

For each group record: how many CVs raised it, their confidence tags,
and how many were out-of-sample.

---

## Step 2 – Cluster and Prioritize

Rank candidate changes by: **frequency** (how many CVs) ×
**confidence** × **out-of-sample weight**. A gap caught in three
out-of-sample CVs outranks one caught once in-sample.

In **cold-start mode**, a single-source hypothesis may still become a
change — but only as a **narrow conditional**, never a sweeping rule.
Flag it `confidence: low` so it can be revisited as N grows.

**Conflicts are not auto-resolved.** If a hypothesis contradicts an
existing rubric rule, or two hypotheses conflict, do not pick a winner.
Surface the conflict in the output for human decision (see Human Gate).

---

## Step 3 – Draft Proposed Diffs

For each prioritized change, draft a **specific** SKILL.md edit:

- Exact target (skill, dimension, the lines to change)
- The proposed new text (a concrete diff, not a vague intention)
- Provenance: which CVs, which confidence
- Scope tag: `narrow conditional` or `general rule`

Respect the existing rubric's design: additive scoring, seniority
ceilings, the consistency rule (no high dimension score paired with a
contradicting recommendation). Do not introduce logic that duplicates
the rubric elsewhere (single source of truth).

---

## Step 4 – Golden-Snapshot Regression Check

The core anti-drift control. The goal is **not** identical scores after
a change — a good change is *supposed* to move some scores. The goal is:
**only intended scores move, and every unintended move is detected.**

### Golden snapshot format (`cv_samples/snapshot.md`)

```
# Golden Snapshot
- Rubric version: cv_review_generic vX.Y.Z
- Model:          claude-opus-4-8
- Date baselined: YYYY-MM-DD

| CV   | D1 | D2 | D3 | D4 | D5 | D6 | D7 | Overall | Key flags |
|------|----|----|----|----|----|----|----|---------|-----------|
| CV_1 | .. | .. | .. | .. | .. | .. | .. |   ..%   | QR/cloud red flag, … |
| …    |    |    |    |    |    |    |    |         |           |
```

Scores are the 3-run means from the discrepancy reports. The snapshot
is the "expected output" baseline.

### Procedure

1. **Noise band.** Take the per-dimension spread recorded in the
   discrepancy reports. Any score move within the spread band is
   **noise**, not a real change.
2. **Re-run.** With the proposed change applied to a working copy of
   the rubric, re-run `cv_review_generic` (3 runs, averaged) on every
   CV in the snapshot.
3. **Diff vs snapshot.** For each dimension of each CV, compare new mean
   to baseline.
4. **Adjudicate every move outside the noise band:**
   - **Intended** (the change targeted this dimension, moved the
     expected direction) → accept; this score will be re-baselined.
   - **Unintended** (a dimension or CV the change should not have
     touched moved) → the change has a side effect. Revise it before
     proposing. Do not ship.
5. Re-baseline the snapshot **only after** a change is human-approved
   and applied.

### Model upgrades

A model change shifts all scores at once. Treat it as a deliberate,
full re-baseline event: re-run the whole snapshot under the new model,
record the magnitude, and review before adopting the new baseline.
Never let a model swap silently overwrite the baseline.

---

## Step 5 – Emit Proposed-Changes Document

Write to `cv_samples/reports/refine_<batch-label>.md`. Two renderings,
mirroring the discrepancy report contract:

```
# Rubric Refinement Proposal – <batch label>

## Header
- Mode:            cold-start | mature
- Batch:           CVs covered
- Rubric version:  current → proposed
- Date:            YYYY-MM-DD

== PART A — OPERATOR RENDERING (structured) ==

## Proposed Changes
For each: target (skill/dimension), the diff, provenance, confidence,
scope (narrow conditional / general rule).

## Regression Impact
Snapshot diff table: which CV/dimension scores moved, by how much,
classified intended / unintended / within-noise.

## Conflicts for Human Decision
Hypotheses that contradict existing rules or each other — presented,
not resolved.

## Deferred
Hypotheses not acted on this batch and why (too weak, in-sample only,
needs corroboration).

== PART B — REVIEWER RENDERING (plain prose, for Karol) ==

Narrative: what we propose to change in the rubric and why, which of
your observations drove it, and any conflicts where your judgment is
needed. Your approval is requested before anything is applied.
```

---

## Human Gate

- **Never auto-edit a rubric.** Output is a proposal only.
- **Conflicts are never auto-resolved.** They are surfaced for the
  operator (Alex) to decide, with the prose rendering sent to the
  domain expert (Karol) for input. This is the Human-in-the-loop
  principle (CLAUDE.md §4) applied to the rubric itself.
- On approval, the edit is applied as its own focused change with a
  version bump (§9), and the golden snapshot is re-baselined.

---

## Quality Controls

- Read-many, write-one: reads reports and the rubric; writes only a
  proposal document. Never edits a rubric SKILL.md.
- No change ships if its regression check shows an unadjudicated
  unintended move.
- Cold-start changes are narrow conditionals tagged `confidence: low`.
- Out-of-sample evidence outweighs in-sample.
- A proposal without a regression-impact section is incomplete — do not
  emit it.
- If the batch is smaller than the cold-start threshold (5-6 CVs) and
  this is the first run, say so and recommend waiting unless the
  operator explicitly overrides.
