---
name: cv_target_role_score
version: 0.1.0
description: >
  Score and review a Client CV against one or more specific target
  roles in the Swiss financial industry. Extends cv_review_generic
  by adding a role-fit dimension and role-specific criteria.
  Use when the Client has identified a specific role, job posting,
  or role type they are targeting.
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

## Relationship to cv_review_generic

This skill **extends** cv_review_generic. Run the generic review first
(or run it in parallel), then add the role-fit layer below.

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

Score the CV against the target role on the following dimensions.

### Dimension 7 – Role Fit Score (added to generic six)

| Sub-dimension | Weight within D7 | Score (0–100) |
|---|---|---|
| Must-have criteria coverage | 50% | |
| Nice-to-have criteria coverage | 25% | |
| Swiss-specific requirements coverage | 25% | |

**Role Fit Score = weighted average of sub-dimensions**

### Overall Score (with target role)

| Dimension | Weight | Score |
|---|---|---|
| Achievement Clarity | 20% | |
| Swiss Market Fit | 15% | |
| Financial Industry Signals | 15% | |
| Relevance Breadth | 10% | |
| Structure and Readability | 8% | |
| Language and Tone | 7% | |
| Role Fit (D7) | 25% | |

**Overall Score = weighted average**

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

Deliver output as structured markdown. Prefix the output with the
role-fit classification banner:

```
ROLE FIT: [Strong / Moderate / Weak / Poor] – Overall Score: XX%
```

---

## Role-Type Reference Library (to be expanded)

The following role types have pre-defined must-have criteria profiles.
Use these as defaults when no job description is provided.

### Credit Risk Manager (VP / Director level, Swiss bank)
Must-have: 5+ years credit risk, quantitative background, CFA or FRM
preferred, FINMA regulatory knowledge, German or English fluent.

### Market Risk Analyst (Analyst / Associate level)
Must-have: Quantitative degree, VaR/ES methodology, Python or R,
Basel market risk framework knowledge.

### Portfolio Manager (Asset Management)
Must-have: CFA charterholder or Level 3 candidate, investment
process knowledge, client-facing experience, German or French
depending on location.

### Quantitative Analyst / Quant Risk
Must-have: Master's or PhD in quantitative field, Python/R/Matlab,
stochastic modelling, derivatives pricing knowledge.

### Risk Reporting / Data Specialist
Must-have: SQL, BI tools (Power BI / MicroStrategy / Tableau),
regulatory reporting experience, strong attention to detail.

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
