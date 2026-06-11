# CFA Society Switzerland – CV Review Tool
## CLAUDE.md – Project Knowledge Base for Claude Code

> **Last updated:** 2026-06  
> **Current release:** v0.2.0  
> **Repo:** https://github.com/Amarows/cfa_skills  
> **Local path:** C:\Users\alexe_ne2qg0j\OneDrive\Files\Programming\Python\cfa_skills

---

## 1. Project Context

### Who Owns This

**Alex Malashonak** (Product Owner, AI architect) and **Karol Brodzinski** (domain expert, Careers Committee) are building an AI-powered CV review tool for the **CFA Society Switzerland Careers Committee**. The Committee provides a free CV review service to CFA members and candidates seeking employment in the Swiss financial industry. Previously this was entirely manual; the goal is to encode Karol's expertise into a consistent, scalable AI-assisted workflow.

### Current Phase

Phase 1 – Committee internal use. A Committee member uploads a CV, the tool scores it, and the member reviews the AI output before sending it to the client. No client-facing access yet.

---

## 2. Repository Structure

```
cfa_skills/
├── CLAUDE.md                          ← this file
├── README.md
├── .gitignore                         ← excludes cv_samples/ (PII) and .idea/
├── cfa_cv_review.html                 ← main application (single-page HTML, v0.3.0)
├── cv_samples/                        ← LOCAL ONLY (gitignored): paired real CVs + expert feedback
│   ├── CV_1.pdf  +  CV_1_feedback.docx
│   ├── CV_2.pdf  +  CV_2_feedback.docx
│   └── CV_3.pdf  +  CV_3_feedback.docx
└── skills/
    ├── cv_review_generic/
    │   └── SKILL.md                   ← 7-dimension generic CV scoring rubric (v0.3.0)
    ├── cv_target_role_score/
    │   └── SKILL.md                   ← role-fit extension (v0.1.0)
    ├── cv_calibration_eval/           ← diff AI review vs expert feedback, read-only (v0.1.0)
    │   └── SKILL.md
    └── cv_rubric_refine/              ← batch rubric refinement from samples, human-gated (v0.1.0)
        └── SKILL.md
```

### Key File: cfa_cv_review.html

Single-page HTML application. All CSS and JS are inline (no build step, no npm). Deployed by copying the file to a hosting location or opening locally. Features implemented as of v0.2.0:

- PDF and DOCX CV upload via `mammoth.convertToHtml` (DOCX) and PDF.js (PDF)
- Hidden private cloud link detection via HTML extraction (security fix) – catches OneDrive/Dropbox/GDrive URLs that are hyperlinked but not visible as plain text
- Anthropic API call to `claude-opus-4-8` with the `cv_review_generic` SKILL.md prompt injected as system prompt
- Score display: 7-dimension table, overall weighted score, triage block, priority recommendations, summary paragraph
- Diagnostics panel: exposes raw JSON response, weight verification, model score drift detection
- Dark mode support via `prefers-color-scheme`
- No server-side component; API key stored AES-GCM-encrypted in the page, decrypted in memory by the access password (PBKDF2, WebCrypto — issue #21). Rotation helper: `cv_samples/encrypt_key.py` (local)
- Prompt caching (issue #17): system prompt sent as cached multi-block (`cache_control: ephemeral`); `max_tokens: 0` pre-warm fires once a valid password is entered; cache hit/miss shown in diagnostics panel

### GitHub Issue Backlog

| # | Title | Status |
|---|---|---|
| #17 | Implement prompt caching | Closed – implemented 2026-06-11 |
| #21 | Encrypt API key in HTML | Closed – implemented 2026-06-11 |
| #22 | Model upgrade to claude-opus-4-8 (re-baseline event) | Open – in progress |

---

## 3. Skills

### cv_review_generic (v0.3.0)

The core scoring rubric. Encodes Karol Brodzinski's expert heuristics. Seven scoring dimensions:

| # | Dimension | Weight |
|---|---|---|
| 1 | Achievement Clarity and Evidence | 25% |
| 2 | Swiss Market Fit | 15% |
| 3 | Financial Industry Signals | 15% |
| 4 | Positioning and Professional Identity | 15% |
| 5 | Structure and Readability | 15% |
| 6 | Language and Tone | 5% |
| 7 | Document Hygiene and Modern Formatting | 10% |

**Scoring rules:**
- Scores are additive from zero (not deductive from 100)
- Hard deductions apply in Dimension 7 for QR codes or private cloud links (score forced to 0 on that dimension + mandatory red flag at top of output)
- Director-level seniority ceiling applies to Dimension 1: senior candidates without quantified achievements cannot score above a defined ceiling regardless of other signals
- Consistency rule: a high dimension score paired with a recommendation that contradicts it is not permitted (e.g., scoring D1 at 80 then recommending adding quantified achievements)

**Calibration history:** Early versions inflated scores significantly. Root causes identified and fixed: wrong rubric encoding, missing dimensions, and incorrect weight application. Diagnostic tooling added to catch future drift.

### cv_target_role_score (v0.1.0)

Extends `cv_review_generic` with an additional role-fit dimension (D7 in this skill, replacing the generic D7). Used when a specific job posting or role type is available. Adds role decomposition (must-have / nice-to-have / Swiss-specific criteria), gap analysis, and CV tailoring recommendations.

---

## 4. Design Principles

| Principle | Detail |
|---|---|
| Anti-hallucination discipline | Every review claim must be traceable to an observable fact in the CV. The skill begins with an "Observed Facts" extraction block before generating recommendations. |
| Minimal, targeted changes | Each commit addresses a single root cause. Do not batch unrelated changes. |
| Single source of truth | Scoring rubric lives in SKILL.md; the HTML application injects it at runtime. Do not duplicate rubric logic in JavaScript. |
| Diagnostic visibility | The diagnostics panel must remain visible in the UI to catch model score drift early. Never remove it without replacement. |
| Human-in-the-loop (Phase 1) | AI output is a draft. Committee member reviews before client delivery. The UI must make this gate explicit. |
| Privacy | Client CVs contain PII. No server-side storage. API call is direct browser-to-Anthropic. nDSG and GDPR compliance required for Phase 2. |

---

## 5. GitHub Operations – Patterns and Gotchas

**Authentication:** Personal access token with `repo` and `project` scopes. Store in environment variable or Claude Code project instructions; never hardcode in committed files.

**File update pattern (critical):**
```python
# ALWAYS fetch a fresh SHA before any PUT operation
# Stale SHAs cause 409 Conflict errors
response = requests.get(
    f"https://api.github.com/repos/Amarows/cfa_skills/contents/{path}",
    headers={"Authorization": f"token {PAT}"}
)
current_sha = response.json()["sha"]

# Then PUT with the fresh SHA
requests.put(
    f"https://api.github.com/repos/Amarows/cfa_skills/contents/{path}",
    headers={"Authorization": f"token {PAT}"},
    json={"message": commit_message, "content": base64_content, "sha": current_sha}
)
```

**Shell scripting reliability:** For long payloads with special characters, write to `/tmp/script.py` and execute with `python3 /tmp/script.py`. Do not use inline `-c` strings for complex operations.

---

## 6. Anthropic API Usage

**Model in use:** `claude-opus-4-8`

**Prompt caching (Issue #17 – implemented):**
```javascript
// As implemented in cfa_cv_review.html
{
  model: "claude-opus-4-8",
  system: [
    {
      type: "text",
      text: "<full SKILL.md content here>",
      cache_control: { type: "ephemeral" }
    }
  ],
  messages: [
    { role: "user", content: "<CV text>" }
  ]
}
```
Pre-warm call: a `max_tokens: 0` request with the cached system prompt fires once a valid access password is entered (prefill-only — writes the cache, bills no output tokens). This seeds the cache so the first real CV review also benefits from caching. Note: minimum cacheable prefix on Opus 4.8 is 4096 tokens; the rubric prompt is ~5.6k tokens, above the bar.

**DOCX extraction:** `mammoth.convertToHtml` – security-safe, does not execute macros, extracts text and basic structure. Used in preference to direct XML parsing for the main flow.

**Security detection:** After `mammoth.convertToHtml`, the HTML output is scanned for `<a href>` tags pointing to private cloud storage domains (onedrive.live.com, 1drv.ms, dropbox.com, drive.google.com, etc.). This catches links that are visually hidden in the CV (hyperlinked text with no visible URL) – a vector that plain-text extraction misses.

---

## 7. CV Learning Loop (Continuous Calibration)

The mechanism for turning accumulating CV samples into rubric improvements **without score drift**. This is core architecture, not a one-off task.

### Data

`cv_samples/` (local only, gitignored — PII). Each sample is a **paired ground-truth example**: `CV_N.pdf` (input) + `CV_N_feedback.docx` (expert human review = Karol's judgment). New samples are dropped into this folder over time. They serve a dual role — *training signal* (mine for heuristics) and *evaluation signal* (benchmark the tool against the human). Use requires client consent.

### Two-Skill Architecture

Deliberately split so that cheap/frequent evaluation never touches the rubric, and rubric changes stay rare and gated.

| Skill | Mode | Role |
|---|---|---|
| `cv_calibration_eval` | Read-only, run anytime | Run the current rubric on a CV, diff AI output against the expert feedback, emit a **discrepancy report** (score gaps, missed flags, tone mismatches). Never modifies SKILL.md. Delivers the consistency metric directly. |
| `cv_rubric_refine` | Deliberate, batch, human-gated | Read *accumulated* discrepancy reports, find *recurring* patterns, propose **specific SKILL.md diffs** with provenance + confidence tags. Outputs a proposed diff for human approval — **never auto-commits**. |

**Discrepancy report, two renderings:** a structured/diff version for the operator (Alex) and a plain-prose version for Karol to react to without reading JSON. Karol's reactions feed the refinement decision.

**Cold-start trigger:** `cv_rubric_refine` runs for the first time once **5–6 CVs** have accumulated — not per-CV. This lets precedents build before any generalization.

### Anti-Drift Controls

The calibration history (§3) is a record of score inflation from rubric changes. These controls prevent silent drift:

1. **Golden-snapshot regression harness.** Baseline = current scores + key flags for the calibration set, stored as a snapshot (**local/gitignored — derived from PII**). On *any* rubric change, re-run the set and diff against the snapshot. Adjudicate every moved score: *intended* → accept and re-baseline; *unintended* → fix the rubric before shipping. The goal is not identical scores but **"only intended changes, every unintended change detected."**
2. **Noise floor.** LLMs are non-deterministic even at temperature 0. Before trusting any diff, score one CV 3–5× unchanged to measure variance. Diffs inside that band are noise; only larger moves count.
3. **Pinned model.** `claude-opus-4-8`. A model upgrade shifts all scores — treat it as a deliberate, full re-baseline event, never an accident.

### Cold-Start vs. Mature Mode

The discipline for incorporating knowledge changes with corpus size N:

| | Cold start (N small — current) | Mature (N >~10) |
|---|---|---|
| Mental model | Case law — each CV is a precedent; every observation counts | Statistics — require corroboration before generalizing |
| New-rule scope | Narrow conditionals only; no sweeping rules | Generalize once a pattern recurs |
| Corroboration | Single-source rules allowed | Require ≥2 supporting examples |
| Provenance | Every rule tagged `source: CV_N, confidence: low/med/high` | Confidence promoted/demoted as evidence accumulates |
| Held-out set | None — all 5–6 CVs used as calibration (accepted tradeoff) | Carve out a held-out eval set never seen by refinement |

### Human Gate

Conflicts between new knowledge and the existing rubric are **never auto-resolved**. Alex reviews and decides; Karol sees the prose discrepancy report for domain input. This is the Human-in-the-loop principle (§4) applied to the rubric itself.

---

## 8. Active Workstreams

| Workstream | Status | Next Action |
|---|---|---|
| Prompt caching (Issue #17) | Open | Implement multi-block system prompt; add pre-warm call on form load |
| Karol knowledge transfer | In progress | Structured HR interviews using 45-minute question guide; training CVs with Karol's assessments for skill calibration |
| Role profile library | Planned | Expand `cv_target_role_score` role-type reference library beyond current 5 profiles |
| CFO skill mapping appendix | Under discussion | Email drafted to Karol; awaiting response |
| German-language CV support | Backlog | Phase 2 scope; requires Committee capacity to validate output |
| LinkedIn promotion post | Pending | Alex prefers short, factual content; self-promotion noted as personally uncomfortable |
| CV learning loop (§7) | In progress | First full cycle complete 2026-06-11: evals on CV_1–3 under Opus 4.8, golden snapshot baselined, refine batch 1 applied as cv_review_generic v0.3.0 (operator override at N=3). Next: Karol confirmation of D2/GPA encodings; next batch as CVs 4–6 arrive |

---

## 9. Open Questions

1. Should the tool cover all Swiss financial roles, or prioritize buy-side / sell-side / risk / quant profiles where Committee expertise is strongest?
2. When should German-language support be added? What is the Committee's capacity to validate output?
3. How will the Committee collect outcome data (did the client get interviews?) to improve the rubric over time?
4. Should a disclaimer be published stating the tool provides guidance only and does not guarantee employment outcomes?
5. CFO-specific skill mapping: should this be a dedicated appendix to `cv_review_generic` or a separate skill?

---

## 10. Versioning Convention

Files follow semantic versioning: `MAJOR.MINOR.PATCH`

- MAJOR: breaking change to scoring logic or output format
- MINOR: new dimension, new heuristic section, or new quality control
- PATCH: wording clarification, typo fix, minor rubric adjustment

Version is declared in the YAML front matter of each SKILL.md and in the HTML `<title>` tag.

---

## 11. Path to Phase 2 (Client-Facing)

Phase 2 gate criteria (from Phase 1):
- At least 10 reviews completed with Committee approval
- Client satisfaction score >= 4.0 (1–5 scale)
- Inter-rater consistency >= 0.80 (Cohen's kappa)
- Legal / data protection review complete
- Privacy policy published

Phase 2 architecture options under consideration: Claude.ai Artifacts wrapper, lightweight web app with serverless backend, or direct Claude.ai Project sharing with clients.

---

*End of CLAUDE.md*
