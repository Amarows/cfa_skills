# CFA Society Switzerland – CV Review AI Agent

## Project Overview

This project builds an AI-assisted CV review service for the CFA Society Switzerland Careers Committee ("the Committee"). The Committee helps members and candidates ("Clients") improve their CVs to find employment in the Swiss financial market.

**Background:** The review process was previously conducted manually by one Committee member. The goal is to systematize, scale, and eventually automate that accumulated knowledge into a repeatable, high-quality AI-assisted workflow.

---

## Step 1 – Targets, Values, and Constraints

### Ultimate Target

> Help Clients increase the probability of securing a job interview in the Swiss financial industry by improving the quality, relevance, and positioning of their CV.

### Success Metrics

| Metric | Description | Target |
|---|---|---|
| Review throughput | CVs reviewed per month by Committee members using the tool | Baseline TBD after pilot |
| Time-to-review | Committee member time spent per CV with AI assistance | < 20 min (vs. estimated 60–90 min manual) |
| Client satisfaction | Post-review survey score (1–5 scale) | >= 4.0 |
| Interview conversion rate | % of Clients who report getting interviews within 90 days | Longitudinal; establish baseline in Year 1 |
| Consistency score | Inter-rater agreement across two independent AI reviews of the same CV | >= 0.80 (Cohen's kappa or equivalent) |

### Core Values

1. **Specificity** – Advice is grounded in Swiss financial market norms and financial industry hiring practices, not generic career coaching.
2. **Honesty** – Feedback is constructive but frank; the tool must not inflate assessments to please Clients.
3. **Anti-hallucination discipline** – All review claims must be traceable to observable facts in the CV or to documented review criteria. The tool must not invent achievements or infer facts not present in the submitted document.
4. **Human-in-the-loop (Phase 1)** – In the initial phase, a Committee member reviews and approves all AI output before it reaches the Client.
5. **Consistency** – The same CV reviewed twice must produce substantially the same output.
6. **Privacy** – Client CVs contain personal data (PII). Data handling must comply with Swiss data protection law (nDSG) and, where applicable, GDPR.

### Constraints

- **Scope:** Swiss financial industry only (banking, asset management, insurance, fintech). Do not provide generic career advice applicable to other industries or markets.
- **Language:** Initial version in English. German-language CVs are in scope for a later phase.
- **Output ownership:** The AI produces a draft review. The Committee member owns and is accountable for the final output sent to the Client.
- **No fabrication:** The tool must not suggest adding experience or achievements the Client has not documented.
- **No legal advice:** The tool must not comment on work permit eligibility, visa requirements, or employment law.

---

## Step 2 – Review Guidelines and Quality Controls

### Review Dimensions

The CV review skill will evaluate the following dimensions. Each dimension receives a score and actionable comments.

| # | Dimension | Weight | Description |
|---|---|---|---|
| 1 | Relevance to Target Role | 25% | How well does the CV match the role type the Client is targeting? |
| 2 | Achievement Clarity | 20% | Are accomplishments quantified? Are outcomes stated, not just duties? |
| 3 | Swiss Market Fit | 20% | Does the CV address Swiss-specific signals: language skills, work permit, location, format norms? |
| 4 | Financial Industry Signals | 15% | Are relevant certifications (CFA, FRM, CAIA, etc.), products, and regulatory frameworks surfaced prominently? |
| 5 | Structure and Readability | 10% | Logical flow, length (typically 1–2 pages for most roles), formatting consistency. |
| 6 | Language and Tone | 10% | Professional, precise, no grammar errors. Action-verb openings. No passive voice in achievement statements. |

**Overall Score:** Weighted average of the six dimensions, expressed as a percentage.

### Swiss Financial Market – Key Review Criteria

The following are known differentiators for Swiss financial industry CVs:

- **Work permit status** must be stated explicitly (EU/EFTA citizen, C permit, B permit, etc.). Omitting this creates friction for HR.
- **Language skills** must be listed with proficiency levels (e.g., B2, C1, native). German is a strong positive signal for Swiss roles; French for Geneva/Lausanne.
- **Certifications** (CFA, FRM, CAIA, PRM, CIIA, AZEK) should appear near the top of the document, not buried in an education section.
- **Quantified achievements** are expected at the senior level. Vague statements ("improved processes") are penalized.
- **Photo:** Optional and declining in prevalence; not penalized if absent.
- **Date of birth / nationality:** No longer required in Switzerland; advise removal if present to reduce bias risk.
- **References:** "Available upon request" is redundant; advise removal.
- **CV length:** For professionals with 5–15 years of experience, two pages is the norm. Senior/Director-level may extend to three.
- **ATS compatibility:** Plain formatting is preferred. Avoid text boxes, tables for layout, graphics, and headers/footers for critical information.

### Quality Controls

- **Source citation:** Every recommendation must reference the specific section or line of the CV it relates to. Generic comments are rejected.
- **Positive framing:** All feedback must include both what is done well and what needs improvement. Pure negative output is not permitted.
- **Grounding check:** Before generating recommendations, the skill must confirm it has read the full CV text. No recommendations on content not visible in the input.
- **Consistency check (optional in Phase 1):** For calibration, randomly re-run 10% of reviews and compare scores. Flag discrepancies > 10 percentage points.
- **Committee member review gate:** The tool produces a draft. A Committee member must approve before delivery to Client.

---

## Step 3 – Architecture

### Component Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Committee Member UI                   │
│          (Claude.ai Project or custom interface)         │
└───────────────────────┬─────────────────────────────────┘
                        │
              ┌─────────▼──────────┐
              │   cv-review skill  │  ← Primary skill (to be built)
              │   (SKILL.md)       │
              └─────────┬──────────┘
                        │
          ┌─────────────┼──────────────┐
          │             │              │
  ┌───────▼──────┐  ┌───▼───────┐  ┌──▼───────────────┐
  │ Swiss Market │  │ Scoring   │  │ Output Formatter  │
  │ Criteria DB  │  │ Rubric    │  │ (docx or markdown)│
  └──────────────┘  └───────────┘  └──────────────────┘
```

### Skills Required

| Skill | Status | Purpose |
|---|---|---|
| `cv-review` | To build | Core review logic: ingest CV, score dimensions, generate recommendations |
| `docx` | Available (public) | Read uploaded .docx CVs; produce formatted review output |
| `pdf-reading` | Available (public) | Read uploaded PDF CVs |
| `job-search` | Available (user) | Optional: cross-reference CV against a live role to assess fit |

### Skill: `cv-review` – Specification (Draft)

**Inputs:**
- Client CV (PDF or DOCX)
- Target role type (optional: e.g., "Risk Manager", "Portfolio Manager", "Quant Analyst")
- Target seniority level (optional: VP / Director / MD / Analyst)

**Outputs:**
- Dimension scores (six dimensions + weighted overall)
- Per-dimension commentary (max 3–5 bullet points each)
- Priority recommendations (top 3 actions, ranked by impact)
- Summary paragraph (for Committee member to use verbatim or edit)

**Anti-hallucination protocol:**
- Every claim in the review must be traceable to a line in the CV.
- The skill must begin by extracting and listing the factual content it will review (an "observed facts" block), before generating recommendations. This block is internal and not shown to the Client.

### Access and Sharing

**Phase 1 (Committee internal):**
- The skill lives in a Claude.ai Project accessible to the Committee member(s).
- Access is managed via the Claude.ai Project sharing feature.
- No external access. Client CVs are uploaded by the Committee member.

**Phase 2 (Client-facing – conditional on Phase 1 success):**
- See Step 4 below.

### Data Handling

- Client CVs must not be stored outside the Claude.ai session unless the Client has given explicit consent.
- Committee members must not upload CVs to any other AI service or public tool.
- A data processing note (one paragraph) should be prepared for Clients explaining how their data is used. Legal review recommended before Phase 2 launch.

---

## Step 4 – Path to Client-Facing Release

### Feasibility Assessment

| Factor | Assessment |
|---|---|
| Technical feasibility | High – Claude.ai Artifacts or a lightweight web wrapper could expose the skill |
| Legal / data protection | Medium risk – nDSG compliance required; PII handling policy needed |
| Differentiation vs. LinkedIn/LHH | High – Swiss-specific, finance-specific criteria; no generic advice |
| Committee capacity to support | Low risk in Phase 2 if fully automated; Medium if hybrid |
| Brand risk | Moderate – output quality must be high before public release |

### Recommended Phasing

| Phase | Description | Gate Criterion |
|---|---|---|
| Phase 0 | Build and test `cv-review` skill internally using historical CVs (with Client consent) | Skill produces consistent, accurate reviews on test set |
| Phase 1 | Committee members use the tool as a drafting assistant; they review and send output | >= 10 reviews completed; satisfaction >= 4.0; consistency >= 0.80 |
| Phase 2 | Client-facing self-service tool with optional Committee member review gate | Phase 1 criteria met; legal review complete; privacy policy published |

### Differentiation vs. Existing Services

| Dimension | LinkedIn CV Review (Contractor) | LHH / Career Agency | This Tool |
|---|---|---|---|
| Industry specificity | Low | Medium | High (finance only) |
| Swiss market norms | Low | Medium | High |
| Consistency | Low (human variance) | Medium | High (AI-grounded rubric) |
| Cost to Client | Medium–High | High | Low / free (Committee service) |
| Speed | Days | Days–weeks | Minutes |
| CFA Society credibility | None | None | High (trusted brand) |

---

## Open Questions

1. **Scope definition:** Should the tool cover all financial roles in Switzerland, or prioritize buy-side / sell-side / risk / quant profiles where Committee expertise is strongest?
2. **German-language CVs:** When should German-language support be added? What is the Committee's capacity to validate German-language output?
3. **Feedback loop:** How will the Committee collect outcome data (e.g., did the Client get interviews?) to improve the rubric over time?
4. **Karol knowledge transfer:** Before building the skill, a structured interview or written knowledge dump from Karol is recommended. His heuristics should be encoded into the rubric, not lost.
5. **Liability:** Should the Committee publish a disclaimer that the tool provides guidance only and does not guarantee employment outcomes?
6. **Version control:** SKILL.md and criteria files will evolve. A versioning convention (e.g., `cv-review-v1.0`) should be established from the start.

---

## Next Actions

| Priority | Action | Owner |
|---|---|---|
| 1 | Conduct knowledge transfer session with Karol; document his review heuristics | Alex + Karol |
| 2 | Define target role taxonomy (which roles / seniority levels are in scope) | Committee |
| 3 | Collect 5–10 anonymized historical CVs for testing | Committee |
| 4 | Build `cv-review` SKILL.md v0.1 | Alex |
| 5 | Run pilot on test CVs; calibrate scoring rubric | Alex + 1 Committee member |
| 6 | Legal / data protection review for Phase 2 scope | Committee + legal advisor |
