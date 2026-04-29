---
name: cv_review_generic
version: 0.1.0
description: >
  Review a Client CV when no specific target role is known.
  Scores the CV across six dimensions grounded in Swiss financial
  market norms and produces prioritized, actionable recommendations.
  Use when the Client has not specified a target role or is open to
  multiple role types. Always hands off to cv_target_role_score when
  a target role is provided.
---

# CV Review – Generic (No Target Role)

## Purpose

Perform a structured, consistent review of a Client CV for the Swiss
financial industry market. Output is a draft for the Committee member
to review before delivery to the Client.

## When to Use This Skill

- Client has not specified a target role
- Client is exploring multiple directions
- Initial triage pass before a more targeted review

## Inputs

| Input | Required | Description |
|---|---|---|
| CV file | Yes | PDF or DOCX |
| Client's stated goal | Optional | e.g., "looking for a risk or quant role" |
| Seniority level | Optional | Analyst / VP / Director / MD |

## Step 1 – Observed Facts Extraction (Internal)

Before generating any recommendations, extract and list the following
from the CV. This block is internal – do not show it to the Client.

- Full name and contact details present (Y/N)
- Work permit status stated (Y/N; value if stated)
- Languages listed with proficiency levels (list them)
- Certifications listed (list them)
- Total years of experience (estimate)
- Number of roles listed
- For each role: employer, title, dates, key achievements (quantified Y/N)
- Education: institutions, degrees, dates
- Technical skills listed (list them)
- CV length in pages (estimate)
- Format: plain text / tables / graphics / text boxes

**Anti-hallucination gate:** Do not proceed to recommendations if any
section of the CV was not readable or was missing from the input.
State the gap and ask the Committee member to re-upload.

## Step 2 – Scoring

Score each dimension on a scale of 0–100. Apply the weights below to
compute the overall weighted score.

| # | Dimension | Weight | Score (0–100) |
|---|---|---|---|
| 1 | Achievement Clarity | 25% | |
| 2 | Swiss Market Fit | 20% | |
| 3 | Financial Industry Signals | 20% | |
| 4 | Relevance Breadth | 15% | |
| 5 | Structure and Readability | 10% | |
| 6 | Language and Tone | 10% | |

**Overall Score = weighted average**

### Scoring Guidance

**Achievement Clarity (25%)**
- 80–100: Majority of roles have quantified achievements (numbers, %, CHF amounts, team sizes)
- 50–79: Some quantification; mix of duties and outcomes
- 0–49: Predominantly duty-based; no quantification

**Swiss Market Fit (20%)**
- Work permit status explicitly stated: +20 pts
- Languages with proficiency levels: +20 pts
- Swiss employer names or Swiss regulatory context referenced: +20 pts
- Location in Switzerland stated: +20 pts
- Date of birth / nationality removed (or not present): +20 pts

**Financial Industry Signals (20%)**
- Relevant certifications prominent (CFA, FRM, CAIA, CIIA, AZEK, PRM): +25 pts each, max 50
- Financial products, asset classes, or regulatory frameworks named: +25 pts
- Industry-standard tools named (Bloomberg, Murex, Calypso, etc.): +25 pts

**Relevance Breadth (15%)**
- Does the CV communicate a coherent professional identity?
- Is the headline / summary targeted to financial services?

**Structure and Readability (10%)**
- Reverse chronological order
- Consistent date format
- Appropriate length (1–2 pages for < 10 years; 2–3 for > 10 years)
- No text boxes, no tables used for layout, no graphics

**Language and Tone (10%)**
- Action verbs opening each achievement statement
- No first-person pronouns (I, my, we)
- No grammar or spelling errors
- No "References available upon request"

## Step 3 – Per-Dimension Commentary

For each dimension, provide:
- 1–2 sentences on what is done well (grounded in observed facts)
- 2–3 specific, actionable recommendations (each referencing the
  section or line of the CV it relates to)

## Step 4 – Priority Recommendations

List the top 3 actions the Client should take, ranked by expected
impact on interview conversion. Format:

1. **[Action]** – [Why it matters] – [Specific section to edit]
2. ...
3. ...

## Step 5 – Summary Paragraph

Write a 3–5 sentence summary for the Committee member to send to the
Client or use as an introduction to the detailed feedback. Tone:
constructive, direct, professional.

## Output Format

Deliver output as structured markdown. The Committee member will
review and may convert to DOCX using the docx skill before sending.

---

## Quality Controls

- Every recommendation must cite a specific section or line of the CV.
- Generic statements not traceable to the CV are not permitted.
- Each dimension must have at least one positive observation.
- If the CV is not in English, flag this to the Committee member and
  pause. Do not review non-English CVs without explicit instruction.
- If the CV belongs to a student or recent graduate (< 2 years
  experience), apply the scoring rubric with adjusted expectations
  and flag this context in the output header.
