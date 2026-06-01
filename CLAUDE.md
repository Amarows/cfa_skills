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
├── cfa_cv_review.html                 ← main application (single-page HTML, v0.2.0)
└── skills/
    ├── cv_review_generic/
    │   └── SKILL.md                   ← 7-dimension generic CV scoring rubric (v0.2.0)
    └── cv_target_role_score/
        └── SKILL.md                   ← role-fit extension (v0.1.0)
```

### Key File: cfa_cv_review.html

Single-page HTML application. All CSS and JS are inline (no build step, no npm). Deployed by copying the file to a hosting location or opening locally. Features implemented as of v0.2.0:

- PDF and DOCX CV upload via `mammoth.convertToHtml` (DOCX) and PDF.js (PDF)
- Hidden private cloud link detection via HTML extraction (security fix) – catches OneDrive/Dropbox/GDrive URLs that are hyperlinked but not visible as plain text
- Anthropic API call to `claude-sonnet-4-20250514` with the `cv_review_generic` SKILL.md prompt injected as system prompt
- Score display: 7-dimension table, overall weighted score, triage block, priority recommendations, summary paragraph
- Diagnostics panel: exposes raw JSON response, weight verification, model score drift detection
- Dark mode support via `prefers-color-scheme`
- No server-side component; API key entered by user in the UI (not stored)

### GitHub Issue Backlog

| # | Title | Status |
|---|---|---|
| #17 | Implement prompt caching | Open – next priority |

**Issue #17 – Prompt Caching:**  
The `cv_review_generic` SKILL.md system prompt is static and large. Caching it with `cache_control: { type: "ephemeral" }` yields approximately 90% token savings from the second CV onward in a session. The plan includes a pre-warm call on form load (before the user submits a CV) to seed the cache. Implementation requires switching to multi-block system prompt format per Anthropic prompt caching spec.

---

## 3. Skills

### cv_review_generic (v0.2.0)

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

**Model in use:** `claude-sonnet-4-20250514`

**Prompt caching (Issue #17 – pending):**
```javascript
// Target implementation for Issue #17
{
  model: "claude-sonnet-4-20250514",
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
Pre-warm call: fire a minimal dummy request with the cached system prompt on form load, before the user submits. This seeds the cache so the first real CV review also benefits from caching.

**DOCX extraction:** `mammoth.convertToHtml` – security-safe, does not execute macros, extracts text and basic structure. Used in preference to direct XML parsing for the main flow.

**Security detection:** After `mammoth.convertToHtml`, the HTML output is scanned for `<a href>` tags pointing to private cloud storage domains (onedrive.live.com, 1drv.ms, dropbox.com, drive.google.com, etc.). This catches links that are visually hidden in the CV (hyperlinked text with no visible URL) – a vector that plain-text extraction misses.

---

## 7. Active Workstreams

| Workstream | Status | Next Action |
|---|---|---|
| Prompt caching (Issue #17) | Open | Implement multi-block system prompt; add pre-warm call on form load |
| Karol knowledge transfer | In progress | Structured HR interviews using 45-minute question guide; training CVs with Karol's assessments for skill calibration |
| Role profile library | Planned | Expand `cv_target_role_score` role-type reference library beyond current 5 profiles |
| CFO skill mapping appendix | Under discussion | Email drafted to Karol; awaiting response |
| German-language CV support | Backlog | Phase 2 scope; requires Committee capacity to validate output |
| LinkedIn promotion post | Pending | Alex prefers short, factual content; self-promotion noted as personally uncomfortable |

---

## 8. Open Questions

1. Should the tool cover all Swiss financial roles, or prioritize buy-side / sell-side / risk / quant profiles where Committee expertise is strongest?
2. When should German-language support be added? What is the Committee's capacity to validate output?
3. How will the Committee collect outcome data (did the client get interviews?) to improve the rubric over time?
4. Should a disclaimer be published stating the tool provides guidance only and does not guarantee employment outcomes?
5. CFO-specific skill mapping: should this be a dedicated appendix to `cv_review_generic` or a separate skill?

---

## 9. Versioning Convention

Files follow semantic versioning: `MAJOR.MINOR.PATCH`

- MAJOR: breaking change to scoring logic or output format
- MINOR: new dimension, new heuristic section, or new quality control
- PATCH: wording clarification, typo fix, minor rubric adjustment

Version is declared in the YAML front matter of each SKILL.md and in the HTML `<title>` tag.

---

## 10. Path to Phase 2 (Client-Facing)

Phase 2 gate criteria (from Phase 1):
- At least 10 reviews completed with Committee approval
- Client satisfaction score >= 4.0 (1–5 scale)
- Inter-rater consistency >= 0.80 (Cohen's kappa)
- Legal / data protection review complete
- Privacy policy published

Phase 2 architecture options under consideration: Claude.ai Artifacts wrapper, lightweight web app with serverless backend, or direct Claude.ai Project sharing with clients.

---

*End of CLAUDE.md*
