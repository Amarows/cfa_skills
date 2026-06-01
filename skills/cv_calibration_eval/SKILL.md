---
name: cv_calibration_eval
version: 0.1.0
description: >
  Measure how closely the current CV review rubric agrees with an
  expert human review of the same CV. Runs cv_review_generic on a
  Client CV, normalizes the expert's free-form feedback onto the seven
  scoring dimensions, and emits a discrepancy report (operator and
  reviewer renderings). Read-only: never modifies any rubric SKILL.md.
  Use to calibrate the rubric against Karol Brodzinski's reviews and to
  produce the inputs consumed by cv_rubric_refine. The discrepancy
  report schema defined here is the shared contract between the two
  skills.
changelog:
  - version: 0.1.0
    date: 2026-06
    changes: >
      Initial draft. Defines the discrepancy report schema, the
      prose-to-seven-dimension normalization step, 3-run averaging with
      spread as a per-CV noise reading, and in-sample vs out-of-sample
      tagging. Grounded in the structure of Karol Brodzinski's expert
      feedback (section-by-section prose + General + Strengths/
      Weaknesses/Missing triage, no numeric scores).
---

# CV Calibration Eval (AI Rubric vs. Expert Review)

## Purpose

Measure agreement between the AI rubric (`cv_review_generic`) and an
expert human review of the **same** CV. This is how the project's
consistency metric is produced and how rubric drift is caught.

The philosophy: the expert review is the benchmark, not the rubric.
Where the AI and the expert disagree, the disagreement is *evidence*,
not a verdict. A miss may be a rubric gap (the AI should have caught
it) or a genuine improvement over the human (the AI caught something
the expert did not). This skill records and classifies the
disagreement; it does **not** decide what to change. That decision
belongs to a human and, downstream, to `cv_rubric_refine`.

**This skill is read-only with respect to every rubric SKILL.md.** It
never edits scoring logic. Its only output is a discrepancy report.

---

## When to Use This Skill

- Calibrating `cv_review_generic` against an expert-reviewed CV
- Building the discrepancy reports that `cv_rubric_refine` consumes
- Spot-checking for drift after a model change or rubric edit

---

## Inputs

| Input | Required | Description |
|---|---|---|
| CV file | Yes | PDF or DOCX (`cv_samples/CV_N.pdf`) |
| Expert feedback | Yes | The human review (`cv_samples/CV_N_feedback.docx`) |
| Sample type | Yes | `in-sample` (rubric was derived from this CV) or `out-of-sample` |
| Reviewer name | Optional | Expert author (e.g., Karol Brodzinski) |
| Target role | Optional | If the expert reviewed against a specific role, note it |

**On `sample type` — read this.** If the rubric was built from a CV's
feedback (CV_1 is the founding example for `cv_review_generic` v0.2.0),
an eval on that CV measures *encoding fidelity*, not generalization,
and will look near-perfect. Out-of-sample CVs are the ones that
actually test the rubric. The report header must carry this flag so
the consistency number is never read out of context.

---

## Relationship to Other Skills

- **Consumes** `cv_review_generic` (runs it; never edits it).
- **Produces** discrepancy reports per the schema below.
- **Feeds** `cv_rubric_refine`, which reads accumulated reports.

---

## Step 1 – Run the Current Rubric (Internal)

Run `cv_review_generic` on the CV **three times**, identical input each
time, model pinned to `claude-sonnet-4-20250514`.

For each of the seven dimensions, record all three scores, then
compute:
- **Mean** (used for all downstream comparison)
- **Spread** = max − min across the three runs

The spread is a per-CV reading of model noise. Flag any dimension with
spread > 10 points as **unstable** — a large disagreement on an
unstable dimension may be noise, not a real gap. Carry both mean and
spread into the report; `cv_rubric_refine` uses the spread to set its
regression tolerance band.

If the three runs disagree wildly across most dimensions (overall
spread > 15), stop and report instability before continuing — the
rubric is too non-deterministic to calibrate meaningfully on this CV.

---

## Step 2 – Normalize the Expert Feedback (Internal)

Expert reviews are free-form prose, organized **by CV section**, not by
the seven scoring dimensions, and carry **no numeric scores**. Karol's
reviews follow a recurring shape:

- **General** — opening overall assessment and headline themes
- **Section-by-section** — Photo, Layout, Personal Information,
  Objective, Profile, Skill Set, Experience, Education, Languages,
  Military, Hobbies, Appendix/References, and "sections to add"
- **Strengths / Weaknesses / Missing** — a terse closing triage

Normalize this into a structured form the rubric can be compared
against:

1. Extract every discrete point the expert raises (a critique, a
   strength, or a recommendation).
2. **Map each point to one or more of the seven dimensions.** This is a
   judgment call — make it explicit in the report so a questionable
   mapping is auditable, never hidden. Use this reference mapping:

   | Expert point (examples) | Dimension(s) |
   |---|---|
   | Missing evidence / impact / quantification; generic claims; corporate speak with no proof | D1 Achievement Clarity |
   | DOB/civil status present; language stars vs. framework; Swiss employer/military/Zunft signals; work permit | D2 Swiss Market Fit |
   | Hard skills absent; IFRS/Swiss GAAP/GRI not named; certifications placement | D3 Financial Industry Signals |
   | Missing headline; repetitive Objective/Profile/Skills; no core competencies; identity unclear | D4 Positioning |
   | CV length; reverse-chron; empty pages; references appendix; "Curriculum Vitae" title; signature/date; early-career handling | D5 Structure & Readability |
   | Long single sentences; adjectives without evidence; passive/first-person; filler phrases | D6 Language & Tone |
   | QR codes / private-cloud links; serif vs sans-serif; mixed fonts; filled section backgrounds; section separators | D7 Document Hygiene |

3. For each dimension, infer the expert's **implied sentiment band**
   from the mapped points: `strong` / `mixed` / `weak`. (Karol gives no
   numbers, so this band — not a score — is what the AI's numeric score
   is checked against.)
4. Capture the expert's **closing triage** (Strengths / Weaknesses /
   Missing) verbatim — it compares directly to `cv_review_generic`'s
   Step 4 triage block.
5. Capture the expert's **priority ordering** — what they led with in
   the General section is their implicit top priority.

**Anti-hallucination gate:** map only points the expert actually made.
Do not infer feedback the expert did not give. If the feedback document
is unreadable or truncated, stop and report the gap.

---

## Step 3 – Alignment Analysis

For each dimension, compare the AI output (mean score + commentary +
recommendations) against the normalized expert points, and assign one
status per point:

| Status | Meaning |
|---|---|
| `aligned` | AI and expert agree (both flag it, or both treat as a strength) |
| `ai_missed` | Expert raised it; AI did not surface it → candidate rubric gap |
| `ai_only` | AI raised it; expert did not → false positive **or** improvement over the human (human adjudicates) |
| `severity_mismatch` | Both noted it but disagree on how serious / high-priority it is |
| `score_contradiction` | AI's numeric score contradicts the expert's implied band (e.g., AI scores D1 = 80 while expert band is `weak`) |

Also compare:
- **Triage overlap** — AI's Strengths/Weaknesses/Missing vs. the
  expert's closing triage.
- **Priority agreement** — does the AI's top-3 lead with the same issue
  the expert led with?

---

## Step 4 – Improvement Hypotheses (Logged, Not Applied)

Every `ai_missed`, `severity_mismatch`, and `score_contradiction`
becomes a candidate rubric change — a hypothesis, **not an edit**. For
each, record:

- **Target** — which skill and dimension it concerns
- **Observation** — what the expert caught that the rubric did not
- **Provenance** — `source: CV_N`
- **Confidence** — `low` (single observation, the cold-start default) /
  `med` / `high`
- **Sample type** — `in-sample` / `out-of-sample` (out-of-sample
  hypotheses are stronger evidence)

These are the rows `cv_rubric_refine` aggregates across a batch. This
skill never promotes a hypothesis into a rubric edit.

---

## Step 5 – Emit the Report

Write the report to `cv_samples/reports/CV_N_eval.md` (inside the
gitignored folder — these reports are derived from PII and must never
be committed). Produce **both** renderings defined in the schema below.

---

## Discrepancy Report Schema (Shared Contract)

This schema is the contract between `cv_calibration_eval` (producer)
and `cv_rubric_refine` (consumer). Do not change it in one skill
without the other.

```
# Discrepancy Report – CV_N

## Header
- CV id:            CV_N
- Date:             YYYY-MM-DD
- Model:            claude-sonnet-4-20250514
- Rubric version:   cv_review_generic vX.Y.Z
- Reviewer:         <expert name>
- Sample type:      in-sample | out-of-sample
- Target role:      <if any>

== PART A — OPERATOR RENDERING (structured) ==

## AI Scores (3-run mean | spread)
| # | Dimension | Mean | Spread | Stable? | Expert band |
|---|-----------|------|--------|---------|-------------|
| 1 | Achievement Clarity        | XX | X | Y/N | strong/mixed/weak |
| … |                            |    |   |     |    |
Overall (mean): XX%

## Per-Dimension Alignment
For each dimension: list each expert point with its status
(aligned / ai_missed / ai_only / severity_mismatch / score_contradiction)
and a one-line note. Show the prose→dimension mapping used.

## Missed by AI   (expert caught, rubric did not — highest value)
- [point] — dimension — note

## AI-Only         (rubric caught, expert did not — adjudicate)
- [point] — dimension — false positive? or improvement over human?

## Triage Comparison
- AI Strengths/Weaknesses/Missing  vs.  Expert Strengths/Weaknesses/Missing
- Overlap + divergence

## Priority Comparison
- AI top-3  vs.  expert's implicit lead priority
- Agreement on what matters most: Y/N

## Stability Note
- Unstable dimensions (spread > 10) and caveats

## Improvement Hypotheses (logged, not applied)
| Target | Observation | Provenance | Confidence | Sample type |
|--------|-------------|------------|------------|-------------|

== PART B — REVIEWER RENDERING (plain prose, for Karol) ==

A short narrative (no JSON, no tables) covering:
- Where the AI and your review agreed
- What the AI missed that you caught
- What the AI flagged that you did not (and whether that is fair)
- Where the AI was softer or harsher than you
- One paragraph: your reaction is requested before any rubric change
```

---

## Quality Controls

- **Read-only.** Never edit `cv_review_generic`, `cv_target_role_score`,
  or any rubric file. Output is a report only.
- **Mapping transparency.** Every prose→dimension mapping is shown; a
  reader can challenge any mapping.
- **No invented feedback.** Only map points the expert actually made
  (anti-hallucination gate, Step 2).
- **In-sample honesty.** Never report a consistency figure without the
  `sample type` flag attached.
- **Noise before conclusions.** A disagreement on an unstable dimension
  (spread > 10) is reported as possible noise, not a confirmed gap.
- **Human reaction precedes change.** The reviewer rendering explicitly
  requests the expert's reaction; hypotheses stay logged until a human
  acts on them via `cv_rubric_refine`.
- If the CV or feedback document cannot be read in full, stop and report
  the gap; do not produce a partial report.
