---
name: cv_target_role_score
version: 0.3.0
description: >
  Score and review a Client CV against one or more specific target
  roles in the Swiss financial industry. Extends cv_review_generic
  by adding a role-fit dimension and role-specific criteria.
  Use when the Client has identified a specific role, job posting,
  or role type they are targeting.
changelog:
  - version: 0.3.0
    date: 2026-06-11
    changes: >
      Web-integration release. Role-fit dimension renamed D7 -> D8 (no
      collision with the generic D7 Document Hygiene). Blended-score
      table fixed: the phantom "Relevance Breadth" dimension removed,
      all seven cv_review_generic dimensions retained at compressed
      weights with D8 Role Fit at 25%. Added the operational contract
      (two-call mode: generic review runs unchanged and calibrated;
      role-fit is a separate, conditional second call that receives the
      generic scores as input and never re-scores them), the JSON
      output schema for the web app, an explicit do-not-run trigger
      rule when no role is selected (cost control), and a calibration
      disclaimer (no expert ground truth for role-fit yet; excluded
      from the golden snapshot).
  - version: 0.2.0
    date: 2026-06-11
    changes: >
      Role-Type Reference Library rebuilt (issue #5). Taxonomy
      re-derived from CFA-member career statistics, the Swiss market
      structure, and the CV Doctor corpus seen to date — replacing the
      earlier risk-heavy stub list. Six profiles, each with must-have /
      nice-to-have / Swiss-specific criteria validated against current
      Swiss job postings and career-board requirements (June 2026;
      sources logged in issue #5): Portfolio Manager, Investment
      Research Analyst, Wealth Management / Private Banking RM,
      Asset Management Sales / BD, Risk Manager, Corporate Finance /
      CFO-track / IR.
  - version: 0.1.0
    date: 2026-04
    changes: Initial draft with 5 stub profiles.
---

# CV Review – Target Role Fit Score

## Purpose

Assess how well a Client CV is positioned for a specific target role
or role type. Produces a role-fit score in addition to the generic
six-dimension review, and generates role-specific gap analysis and
recommendations.

## When to Use This Skill

- Client has specified a target role title (e.g., "Credit Risk Manager")
- Client has provided a job posting or job description
- Committee member wants to assess fit before advising whether to apply

## Relationship to cv_review_generic — Operational Contract

This skill **extends** cv_review_generic without ever modifying it.

**Two-call mode (web application):**
1. **Call 1 — generic review.** cv_review_generic runs exactly as
   calibrated (cached prompt, golden-snapshot protected). The model
   never sees role instructions during generic scoring — this is what
   guarantees role context cannot perturb the calibrated dimensions.
2. **Call 2 — role fit (this skill), CONDITIONAL.** Fires only when
   the user has selected a role type or provided a job description.
   **If no target role is given, this skill must not run at all** —
   the generic review is complete on its own and the second call would
   be pure cost.

**Call 2 inputs:** the CV, ONE role profile from the reference library
below (inject only the selected profile, not the whole library) or the
provided JD, and the seven dimension scores from Call 1.

**Score passthrough rule:** the seven generic dimension scores are
inputs, passed through verbatim into the blended computation. This
skill NEVER re-scores a generic dimension. It scores only D8 Role Fit
and computes the blend.

**Calibration disclaimer:** role-fit scores have no expert ground
truth yet (all expert feedback collected so far is role-agnostic).
D8 and the blended score are excluded from the golden snapshot and the
cv_calibration_eval loop until role-targeted expert reviews exist.
Present them as decision support, not calibrated measurements.

## Inputs

| Input | Required | Description |
|---|---|---|
| CV file | Yes | PDF or DOCX |
| Target role | Yes | Title, JD text, or URL of job posting |
| Target employer | Optional | Specific firm name |
| Seniority level | Optional | Analyst / VP / Director / MD |

## Step 1 – Role Decomposition (Internal)

Before scoring, decompose the target role into required and preferred
criteria. This block is internal – do not show to the Client.

Extract from the job description (or infer from role title if no JD):

**Must-have criteria** (deal-breakers if absent):
- Years of experience required
- Specific certifications required (CFA, FRM, etc.)
- Language requirements
- Technical skills required
- Domain knowledge required (e.g., credit risk, market risk, ALM)

**Nice-to-have criteria** (differentiators):
- Preferred certifications
- Preferred tools or platforms
- Preferred sector experience
- Leadership or management experience

**Swiss-specific requirements:**
- Work permit compatibility
- Language requirements for the location (Zurich = German; Geneva = French)
- Regulatory context (FINMA, SNB, BIS, etc.)

## Step 2 – Role-Fit Scoring

Score the CV against the target role. Generic dimensions D1–D7 are
NOT re-scored — their values come from Call 1 (cv_review_generic).

### Dimension 8 – Role Fit Score (added to the generic seven)

| Sub-dimension | Weight within D8 | Score (0–100) |
|---|---|---|
| Must-have criteria coverage | 50% | |
| Nice-to-have criteria coverage | 25% | |
| Swiss-specific requirements coverage | 25% | |

**D8 Role Fit = 0.50 × must_have + 0.25 × nice_to_have + 0.25 × swiss_specific**

Scoring discipline (same philosophy as cv_review_generic): start at 0,
award only for evidence explicitly present in the CV. A must-have
criterion is *covered* only if the CV demonstrates it — a claim
without evidence counts at half weight at most.

### Blended Overall Score (with target role)

All seven generic dimensions are retained at compressed weights;
D8 carries 25%. Generic scores are passed through verbatim.

| Dimension | Generic weight | Blended weight |
|---|---|---|
| D1 Achievement Clarity and Evidence | 25% | 20% |
| D2 Swiss Market Fit | 15% | 10% |
| D3 Financial Industry Signals | 15% | 10% |
| D4 Positioning and Professional Identity | 15% | 10% |
| D5 Structure and Readability | 15% | 10% |
| D6 Language and Tone | 5% | 5% |
| D7 Document Hygiene and Modern Formatting | 10% | 10% |
| D8 Role Fit (this skill) | — | 25% |

**Blended Overall = D1×0.20 + D2×0.10 + D3×0.10 + D4×0.10 + D5×0.10 + D6×0.05 + D7×0.10 + D8×0.25**

Weight rationale: there is no calibration benchmark for the blend yet
(noted 2026-06-11); the 25% D8 share follows the original v0.1.0
design intent, with the generic dimensions compressed roughly
proportionally. Revisit once role-targeted expert feedback exists.

### Fit Classification

| Overall Score | Classification | Recommended Action |
|---|---|---|
| 80–100 | Strong Fit | Apply; minor CV tuning recommended |
| 60–79 | Moderate Fit | Apply with targeted CV edits before submission |
| 40–59 | Weak Fit | Consider whether to apply; significant gaps identified |
| 0–39 | Poor Fit | Do not apply without addressing critical gaps |

## Step 3 – Gap Analysis

For each must-have criterion NOT covered in the CV:
- State the gap explicitly
- Indicate whether it is bridgeable (e.g., certification in progress,
  transferable experience) or a hard gap
- Suggest how the Client could address or frame it

For each nice-to-have criterion NOT covered:
- Note it briefly; do not overweight

## Step 4 – CV Tailoring Recommendations

Provide specific edits the Client should make to their CV to improve
fit for this role. Each recommendation must:
- Reference the specific section of the CV to edit
- State what to add, remove, or reframe
- Explain why it improves fit for this specific role

Limit to the 5 highest-impact edits.

## Step 5 – Summary for Committee Member

Provide:
1. One-sentence fit verdict (e.g., "This CV is a moderate fit for the
   role; two targeted edits would move it to strong fit.")
2. Recommended next step (apply now / edit first / do not apply)
3. Key strengths to highlight in a cover letter for this role

## Output Format

**Interactive use:** deliver structured markdown prefixed with the
role-fit classification banner:

```
ROLE FIT: [Strong / Moderate / Weak / Poor] – Overall Score: XX%
```

**Web application (Call 2):** respond ONLY with a valid JSON object,
no markdown fences, exact schema:

```json
{
  "role_fit": {
    "role_type": "",
    "jd_provided": false,
    "must_have":      { "score": 0, "weight": 0.50, "covered": [], "gaps": [] },
    "nice_to_have":   { "score": 0, "weight": 0.25, "covered": [], "gaps": [] },
    "swiss_specific": { "score": 0, "weight": 0.25, "covered": [], "gaps": [] },
    "d8_score": 0
  },
  "gap_analysis": [
    { "criterion": "", "type": "must_have", "bridgeable": true, "how_to_address": "" }
  ],
  "tailoring_recommendations": [
    { "edit": "", "section": "", "why": "" }
  ],
  "blended_overall": 0.0,
  "fit_classification": "",
  "fit_verdict": "",
  "next_step": "apply_now | edit_first | do_not_apply",
  "cover_letter_strengths": []
}
```

Rules: `gap_analysis` covers every uncovered must-have criterion;
`tailoring_recommendations` max 5, highest impact first;
`blended_overall` computed exactly from the passed-in generic scores
and `d8_score` using the blended weights; `fit_classification` from
the classification table.

---

## Role-Type Reference Library

Six core role types covering the typical CFA Society Switzerland
member profiles (taxonomy rationale and posting sources: issue #5).
Use these as defaults when no job description is provided. When a JD
is provided, the JD always takes precedence; use the profile to fill
gaps the JD leaves implicit.

### 1. Portfolio Manager / Investment Manager (equities, fixed income, multi-asset)
**Must-have:** 5+ years investment management or investment analysis;
CFA charter or Level III candidate; depth in at least one asset class;
demonstrable track record with performance figures (vs. benchmark, AUM
managed); portfolio construction and risk-budgeting experience;
industry platform fluency (Bloomberg, FactSet, Aladdin or equivalent).
**Nice-to-have:** CAIA (alternatives exposure); Python/R for analysis;
ESG/SFDR and sustainable-investing credentials; client-facing or
investment-committee experience; second Swiss language.
**Swiss-specific:** FINMA portfolio-manager licensing context (FinIA);
pension-fund (BVG/LPP) mandate knowledge for institutional books;
Zurich = German, Geneva = French; work-eligibility signal expected;
Swiss residency required by some employers.

### 2. Investment Research Analyst (equity or credit, sell-side / buy-side)
**Must-have:** 2+ years research or analysis; financial modelling and
valuation (DCF, multiples; for credit: PD/ratings frameworks, covenant
analysis); a defined sector or asset-class coverage; CFA progression
(charter expected at senior level); evidence of written research
output; Bloomberg/Refinitiv.
**Nice-to-have:** published coverage or analyst rankings; IFRS / Swiss
GAAP FER literacy; data skills (Python/SQL); coverage of Swiss sectors
(pharma, financials, industrials, luxury).
**Swiss-specific:** SIX-listed issuer coverage is a strong signal;
German (Zurich desks) or French (Geneva); awareness of research
independence/MiFID rules as applied by Swiss houses; work-eligibility
signal expected.

### 3. Wealth Management / Private Banking — Relationship Manager / Investment Advisor
**Must-have:** demonstrable client-book development (net new assets,
retention, AUM figures); client-advisory certification — SAQ CWMA is
required by many Swiss private banks (CFA accepted/valued alongside);
suitability and cross-border rules competence; languages matched to
client segment.
**Nice-to-have:** CIWM; wealth/tax-planning depth (structures,
treaties); Lombard lending and alternative products; UHNW or
entrepreneur segment experience; additional client-market languages
(e.g. Italian, Spanish, Portuguese, Arabic, Mandarin).
**Swiss-specific:** FinSA/FIDLEG suitability and client segmentation;
cross-border frameworks (FATCA, CRS) — named in most Swiss PB
postings; FINMA conduct context; Zurich/Geneva booking-centre
conventions; Swiss residency frequently required.

### 4. Asset Management Sales / Client Coverage & Business Development
**Must-have:** several years institutional or wholesale investment
sales; vehicle knowledge across UCITS funds, ETFs, segregated mandates
and Swiss institutional funds; fluent German AND English for the
Deutschschweiz market (French for Romandie coverage); existing network
among pension funds, insurers, foundations, family offices or
consultants; sales-target / net-new-asset achievement record.
**Nice-to-have:** CFA or CAIA (door-opener with institutional
clients); product specialism (fixed income, alternatives, ESG); CRM
discipline; consultant-relations experience.
**Swiss-specific:** BVG/LPP pension landscape and investment
guidelines; FinSA client segmentation for distribution; Swiss
investment-consultant ecosystem; work-eligibility signal expected.

### 5. Risk Manager (market / credit, banks and insurers)
**Must-have:** 4–7 years risk management in banking, insurance or
risk-focused consulting/audit; Basel III/IV framework knowledge;
quantitative methods (VaR/ES, stress testing, PD/LGD/EAD); FRM or CFA;
statistical tooling (Python, R, SAS, SQL); derivatives product
knowledge.
**Nice-to-have:** model validation experience; FRTB; climate/ESG risk;
SST (Swiss Solvency Test) for insurance roles; German fluency.
**Swiss-specific:** FINMA circulars and supervisory expectations;
Swiss banking structure (systemically important banks, cantonal
banks, private banks have different risk cultures); English-first
with German a strong plus; work-eligibility signal expected.

### 6. Corporate Finance / CFO-track / Investor Relations
**Must-have:** 10+ years finance with 5+ in leadership for CFO roles
(scale down for IR/controlling steps on the track); IFRS AND Swiss
GAAP FER applied knowledge; financial planning, controlling and
board-level reporting; capital-markets evidence for IR (fundraising,
roadshows, analyst relations) with the candidate's own contribution
quantified.
**Nice-to-have:** Swiss certified expert in accounting/controlling or
CPA; CFA; IPO/M&A transaction experience; ESG reporting (GRI);
treasury.
**Swiss-specific:** Swiss Code of Obligations reporting duties; SIX
listing rules and ad-hoc publicity for listed companies; German or
French per region (English often sufficient in listed/biotech/medtech
sector); employer-side social-system basics (AHV/BVG).

---

## Quality Controls

- Role-fit scores must be grounded in criteria extracted from the JD
  or the role-type reference library. Do not invent criteria.
- If no JD is provided and the role type is not in the reference
  library, flag this and ask the Committee member for more context
  before scoring.
- Gap analysis must distinguish hard gaps from bridgeable gaps.
  Overstating gaps is as harmful as understating them.
- Do not advise the Client not to apply based on protected
  characteristics (age, gender, nationality, etc.).
