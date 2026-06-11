---
name: cv_review_generic
version: 0.3.0
description: >
  Review a Client CV when no specific target role is known.
  Scores the CV across seven dimensions grounded in Swiss financial
  market norms and Karol Brodzinski's expert heuristics (CFA Society
  Switzerland, Careers Committee). Produces prioritized, actionable
  recommendations. Use when the Client has not specified a target role
  or is open to multiple role types. Always hands off to
  cv_target_role_score when a target role is provided.
changelog:
  - version: 0.3.0
    date: 2026-06-11
    changes: >
      First cv_rubric_refine batch (CV_1-CV_3, cold-start, operator
      override at N=3 — see cv_samples/reports/refine_batch1.md).
      D2: split nationality-removal from work-eligibility signalling
      (issue #19) — eligibility (permit/EU-EFTA/Swiss status) rewarded
      and never recommended for removal; penalties now target date of
      birth and civil/family status only. D7: QR/private-cloud hard
      zero made deterministic, hyperlinked phrases with invisible
      destinations count for PDFs. D5: fill-the-page and top-left
      information-hierarchy now scored; name-in-page-header required
      on multi-page CVs; footer contact details explicitly NOT
      penalised; education-before-experience deduction; conditional
      GPA/education-detail guidance. Regression-checked against the
      2026-06-11 golden snapshot: no unintended moves; run-to-run
      stability improved. Approved by Alex; Karol confirmation
      requested on the D2 encoding and the GPA condition.
  - version: 0.2.0
    date: 2026-05
    changes: >
      Added Dimension 7 – Document Hygiene and Modern Formatting.
      Expanded all scoring dimensions with Karol Brodzinski heuristics
      derived from expert CV review (anonymized, March 2025).
      Added section-level red flags, seniority calibration rules,
      and extended quality controls. Added equity research analogy
      framing for summary section. Revised output format to include
      Strengths / Weaknesses / Missing triage block.
  - version: 0.1.0
    date: 2026-04
    changes: Initial draft.
---

# CV Review – Generic (No Target Role)

## Purpose

Perform a structured, consistent review of a Client CV for the Swiss
financial industry market. Output is a draft for the Committee member
to review and approve before delivery to the Client.

The review philosophy: a CV is not a list of duties. It is an evidence
document. Every claim must be supported by a specific, verifiable
example. The Committee member's role is to help the Client close the
gap between what they delivered and what their CV currently proves.

---

## When to Use This Skill

- Client has not specified a target role
- Client is exploring multiple directions
- Initial triage pass before a more targeted review

---

## Inputs

| Input | Required | Description |
|---|---|---|
| CV file | Yes | PDF or DOCX |
| Client's stated goal | Optional | e.g., "looking for a CFO role in medtech" |
| Seniority level | Optional | Analyst / VP / Director / MD |

---

## Step 1 – Observed Facts Extraction (Internal)

Before generating any recommendations, extract and list the following
from the CV. This block is internal – do not show it to the Client.

**Identity and contact:**
- Full name present (Y/N)
- Contact details: phone, email, location (Y/N each)
- LinkedIn URL: present and fully written out, not hidden behind text or QR code (Y/N)
- Photo included (Y/N)
- Date of birth present (Y/N) — flag if yes
- Nationality / civil status present (Y/N) — flag if yes
- Work permit status stated (Y/N; value if stated)
- QR codes or links to external clouds / private storage present (Y/N) — flag if yes

**Structure inventory:**
- Sections present (list all, in order)
- Headline present (Y/N)
- Summary / Profile length (word count estimate)
- Core competencies or key skills section present (Y/N)
- CV length in pages
- Font(s) used: serif / sans-serif / mixed — flag if mixed
- Layout: single column / multi-column / use of colour fills / section separators style
- Each page: approximately full (Y/N) — flag half-empty pages

**Experience:**
- Total years of experience (estimate)
- Number of roles listed
- For each of the most recent 4 roles:
  - Employer, title, dates
  - Number of bullet points or lines of description
  - Achievement statements: quantified (Y/N), duty-based (Y/N)
  - Business context stated (Y/N)
  - Impact / outcome stated (Y/N)
- Early career roles: listed without detail (Y/N) — note if early career extends to primary/secondary school

**Education:**
- Institutions, degrees, dates
- Primary and secondary education listed (Y/N) — flag if yes and >35 years ago
- Certifications: list all, note prominence (top of CV / education section / buried)

**Languages:**
- Languages listed with proficiency format (stars / European Framework / written label / native)

**Other sections:**
- Military service (Y/N) — note for Swiss context
- Hobbies: generic (Y/N) / specific (Y/N)
- Extracurricular activities / board memberships (Y/N)
- Hard skills / technical tools listed (Y/N)
- References or appendix with reference names (Y/N) — count if present, flag if >4
- Signature and date at bottom (Y/N) — flag if yes

**Anti-hallucination gate:** Do not proceed to recommendations if any
section of the CV was not readable or was missing from the input.
State the gap and ask the Committee member to re-upload.

---

## Step 2 – Scoring

Score each dimension on a scale of 0–100. Apply the weights below to
compute the overall weighted score.

| # | Dimension | Weight | Score (0–100) |
|---|---|---|---|
| 1 | Achievement Clarity and Evidence | 25% | |
| 2 | Swiss Market Fit | 15% | |
| 3 | Financial Industry Signals | 15% | |
| 4 | Positioning and Professional Identity | 15% | |
| 5 | Structure and Readability | 15% | |
| 6 | Language and Tone | 5% | |
| 7 | Document Hygiene and Modern Formatting | 10% | |

**Overall Score = weighted average**

---

### Scoring Guidance

**Dimension 1 – Achievement Clarity and Evidence (25%)**

This is the most important dimension. A CV makes claims; the evidence
is what makes those claims credible. Score on the presence and quality
of evidence, not on the presence of claims.

- 80–100: The majority of roles in the last 10 years have quantified
  achievements. Each bullet point includes: (a) the candidate's
  specific contribution, (b) a financial figure or measurable outcome,
  (c) business context or reason for the action, (d) impact on the
  company. Duty-based statements are rare or absent.
- 50–79: Some quantification. Mix of achievement statements and duty
  lists. Business context present in some but not all roles.
- 0–49: Predominantly duty-based. High-level generalizations with no
  supporting figures or examples. Claims made without evidence.

**Red flags that reduce the score:**
- Statements that could apply to any candidate in that role (e.g.,
  "responsible for stakeholder management", "reporting obligation to
  CEO") — these are duties, not achievements.
- Adjectives applied to soft skills without supporting evidence
  (e.g., "excellent communication skills", "strong leadership") —
  penalize unless backed by a specific example or external validation.
- A large total funding figure attributed to a team without stating
  the candidate's specific role in generating it.
- Experience section describes tasks at an extremely high level while
  a detailed section contains no bullet points at all.

**Karol's benchmark for a strong achievement bullet (senior level):**
"Led the investor outreach and communication workstreams for CHF Xbn
equity fundraising which enabled the company to complete Phase 2 and
3 of drug development and ultimately increase sales by X% between
201X and 202X."
This template includes: role/contribution + financial figure + business
reason + impact. Apply it as the standard when evaluating bullets.

**Seniority calibration:**
- Analyst / Associate: quantification and task clarity sufficient.
- VP / Director: business context and team impact expected.
- MD / C-suite: strategic impact, P&L or capital figures, board-level
  outcomes expected. Generic high-level descriptions are a significant
  negative signal at this level.

---

**Dimension 2 – Swiss Market Fit (15%)**

| Criterion | Points |
|---|---|
| Work-eligibility signal explicitly stated (permit type, EU/EFTA citizenship, or Swiss status) | +20 |
| Languages listed with European Framework levels or equivalent | +20 |
| Swiss employer names or Swiss regulatory context referenced | +20 |
| Location in Switzerland stated | +20 |
| Date of birth and civil/family status absent | +20 |

**Eligibility vs. personal data (issue #19, refine batch 1):**
Work-eligibility information (e.g. "Swiss Permit B (EU national)",
"Swiss citizen") removes a concrete hiring obstacle — Swiss employers
need cantonal approval for non-EU hires — and must NEVER be
recommended for removal. Date of birth and civil/family status carry
bias risk and no hiring value: recommend removing those only. Bare
nationality with no eligibility relevance is neutral — neither
rewarded nor penalised.

**Karol's notes on Swiss specifics:**
- Military service section: positive signal for Swiss roles; neutral
  or irrelevant for international roles outside Switzerland.
- Zunft membership or Swiss guild/civil society involvement: positive
  signal; recommend placing in Extracurricular Activities rather than
  Hobbies to give it appropriate weight.
- Language star ratings are redundant alongside European Framework
  labels; advise removing stars and keeping framework reference only.

---

**Dimension 3 – Financial Industry Signals (15%)**

- Relevant certifications prominent and near the top of the document
  (CFA, FRM, CAIA, CIIA, AZEK, PRM, CAS-HSG for board roles): +25 pts
  each, max 50.
- Financial products, asset classes, regulatory frameworks, or
  industry-specific methodologies named (e.g., IFRS, Swiss GAAP FER,
  GRI Standards, Basel, ISDA, PFE, CVA): +25 pts.
- Industry-standard tools or platforms named (Bloomberg, Murex,
  Calypso, FactSet, etc.): +15 pts.
- Hard skills section present at senior level confirming technical
  depth (accounting standards, valuation frameworks, financial
  planning tools): +10 pts.

**Note for senior candidates (Director / MD / C-suite):**
A hard skills or technical competencies section is expected even at
senior level in Swiss financial industry CVs. Its absence is a gap.
IFRS, Swiss GAAP FER, or GRI Standards (for ESG-focused roles) should
be explicitly named if relevant to the candidate's background.

---

**Dimension 4 – Positioning and Professional Identity (15%)**

This dimension captures how quickly and clearly the CV communicates
who the candidate is and what role they are targeting.

- 80–100: Headline present and role-specific. Summary is concise
  (60–90 words), focused on top 2–3 achievements and hard skills,
  with evidence. Core competencies or key skills section present with
  3–5 targeted keywords. No repetition across sections.
- 50–79: Some positioning present but diluted by lengthy prose,
  repetitive sections, or generic language. Recruiter must read
  multiple sections to understand the candidate's identity.
- 0–49: No headline. Summary is generic, duty-focused, or exceeds
  150 words. Objective, Profile, and Skills sections repeat the same
  information without adding evidence. Identity unclear.

**Karol's equity research analogy (apply when evaluating this dimension):**
Think of the opening of a CV as the first page of an equity research
report. The reader needs to immediately understand: what is the
recommendation, and what are the 3 key reasons. If the reader must
work through dense prose to form a conviction, the CV has already
failed. The opening must answer: who is this person, what role are
they targeting, and what are the 3 key reasons they are a strong
candidate?

**Red flags:**
- Objective statement longer than 30 words that does not name a
  specific role type.
- A single sentence in the summary exceeding 50 words.
- Three or more sections (Objective, Profile, Skills) that repeat
  substantially the same content.
- Summary that lists soft-skill adjectives without a single
  quantified achievement.
- No headline at the top of the CV (especially at senior level).

**Sections to recommend adding if absent:**
- Headline (one line, role-specific – same logic as LinkedIn headline)
- Core competencies / top skills (3–5 keywords, adaptable per role)
- Extracurricular activities (separate from Hobbies; for board
  memberships, honorary roles, professional society involvement)
- Hard skills (technical tools, accounting standards, frameworks)

---

**Dimension 5 – Structure and Readability (15%)**

- Reverse chronological order: Y/N
- Consistent date format throughout: Y/N
- Appropriate length:
  - <10 years experience: 1–2 pages
  - 10–20 years experience: 2–3 pages
  - >20 years experience: up to 3 pages (not more)
- Pages approximately full (no half-empty pages): Y/N
- Early career handled correctly:
  - Last 4–5 roles or approximately 10–20 years: full bullet-point descriptions
  - Earlier roles: brief listing without descriptions
  - Primary / secondary education: remove if >35 years ago
- "Curriculum Vitae" title removed from document header: Y/N
- Name repeated as header on every page (multi-page CVs): Y/N — scored;
  separated printed pages must be re-unitable
- Information hierarchy: the top-left / first block after the name
  carries identity content (headline, summary, or key focus areas),
  not a dense personal-data or address block. The top-left is where a
  recruiter's eyes go first; contact details belong in a compact line
  or the page footer. Contact details in the footer are GOOD practice
  — never penalise or flag them.
- Section order: Experience precedes Education for candidates with
  more than 2 years of professional experience — flag if reversed
- Education detail (GPA, major/minor, thesis title): conditional —
  recommend ADDING when recent and strong or when it helps fill the
  chosen page count; recommend OMITTING when more than ~10 years old
  or below 5/6 (or equivalent), as old or weak GPAs hand recruiters an
  early elimination criterion. Never blanket-advise either direction.
- Signature and date at bottom: flag if present (remove)
- References appendix: flag if present; advise maximum 4 named
  references, pre-aligned with the candidate, disclosed only on
  request and calibrated per specific application

**Karol's note on references:**
Having more than 4–5 named references means the candidate cannot
realistically align all of them with the storyline for each specific
role before a recruiter call occurs. Advise the Client to name no
more than 4 references per application, inform each one in advance
for that specific role, and align on key messages before any call.
Voluntarily listing 20+ references surrenders control of a step in
the process where the candidate should be fully in control.

---

**Dimension 6 – Language and Tone (5%)**

- Action verbs opening each achievement statement: Y/N
- No first-person pronouns (I, my, we): Y/N
- No grammar or spelling errors: Y/N
- No "References available upon request": Y/N
- No corporate-speak or generic filler phrases.
  Examples to flag: "results-oriented", "dynamic professional",
  "passionate about", "strong track record of delivering",
  "deep understanding of" (unless followed by specific evidence).
- Objective statement, if present: flag if a single sentence exceeds
  40 words.

---

**Dimension 7 – Document Hygiene and Modern Formatting (10%)**

| Criterion | Points |
|---|---|
| Single font used throughout (sans-serif preferred) | +20 |
| No QR codes | +20 |
| No links to private cloud storage or unknown external URLs | +20 |
| No filled/shaded section backgrounds | +20 |
| Section separators: single line above section, not double | +20 |

**Cybersecurity red flag – HARD RULE, no discretion:**
Any CV containing QR codes or links to private cloud storage
(Dropbox, OneDrive personal, Google Drive private, etc.) must receive
a total score of EXACTLY 0 on this dimension — no partial credit from
the other criteria — plus a mandatory red flag at the top of the
output and as Priority 1, regardless of other qualities. For PDFs,
hyperlinked phrases whose destination is not visible ("click here",
"link to document", references to QR codes or external attachments)
count as private/unknown links. A D7 score between 1 and 99 in the
presence of such an element is a scoring error. The only acceptable
external link on a CV is a fully written LinkedIn profile URL.

Rationale: Swiss financial industry recruiters follow strict
cybersecurity protocols. A QR code or private cloud link in a CV
creates immediate mistrust — the recruiter cannot verify the
destination before scanning or clicking. This is a basic corporate
security breach that causes some recruiters to discard the application.

**Additional formatting guidance:**
- Font: sans-serif (Arial, Calibri, Helvetica) preferred over serif
  (Cambria, Times New Roman). Mixing fonts of different typefaces
  is a consistency error — penalize.
- Colour fills on section backgrounds: advise removal; looks dated.
- Dark-coloured personal information headers: advise neutral/white
  background for printer-friendliness.
- Section separators: one line above section is sufficient; remove
  bottom separator and replace with white space.

---

## Step 3 – Per-Dimension Commentary

For each dimension, provide:
- 1–2 sentences on what is done well (grounded in observed facts)
- 2–3 specific, actionable recommendations (each referencing the
  specific section of the CV it relates to)

At senior level (Director / MD / C-suite), commentary must be direct
and frank. Do not soften feedback to the point where the Client
underestimates the severity of a gap. Constructive tone, honest
content.

---

## Step 4 – Triage Block

Produce a structured triage summary in the following format:

**Strengths:**
- [List 3–5 observable positives from the CV]

**Weaknesses:**
- [List 3–5 specific gaps, each referencing a section]

**Missing sections recommended:**
- [List sections absent but expected for this seniority/profile]

---

## Step 5 – Priority Recommendations

List the top 3 actions the Client should take, ranked by expected
impact on interview conversion. Format:

1. **[Action]** – [Why it matters] – [Specific section to edit]
2. ...
3. ...

At senior level, Priority 1 should almost always address evidence
and achievement framing unless a cybersecurity red flag (QR code /
cloud link) is present, in which case that takes Priority 1.

---

## Step 6 – Summary Paragraph

Write a 3–5 sentence summary for the Committee member to send to the
Client or use as an introduction to the detailed feedback.

Tone: constructive, direct, professional. Do not inflate. If the CV
significantly undersells the candidate (a common pattern at senior
level), say so explicitly and explain why this matters.

---

## Output Format

Deliver output in the following structure:

```
OVERALL SCORE: XX%  |  [Strong / Moderate / Weak / Poor]

TRIAGE
Strengths: ...
Weaknesses: ...
Missing: ...

DIMENSION SCORES
1. Achievement Clarity and Evidence:      XX/100
2. Swiss Market Fit:                      XX/100
3. Financial Industry Signals:            XX/100
4. Positioning and Professional Identity: XX/100
5. Structure and Readability:             XX/100
6. Language and Tone:                     XX/100
7. Document Hygiene and Formatting:       XX/100

DIMENSION COMMENTARY
[Per dimension: positives + recommendations]

TOP 3 PRIORITY RECOMMENDATIONS
1. ...
2. ...
3. ...

SUMMARY FOR CLIENT
[3–5 sentences]
```

---

## Quality Controls

- Every recommendation must cite a specific section or line of the CV.
  Generic statements not traceable to the CV are not permitted.
- Each dimension must have at least one positive observation.
- If a QR code or private cloud link is detected, issue a mandatory
  red flag at the top of the output before all other content.
- If the CV is not in English, flag this to the Committee member and
  pause. Do not review non-English CVs without explicit instruction.
- If the CV belongs to a student or recent graduate (<2 years
  experience), apply the scoring rubric with adjusted expectations
  and flag this context in the output header.
- If three or more sections repeat substantially the same content,
  flag this as a structural issue in the Triage block.
- Do not reward length. A long summary or verbose objective is a
  negative signal, not a positive one.

---

## Reference: Hobbies Guidance

Apply when the Hobbies section contains only generic interests:

Hobbies help steer interview conversations. Generic interests
(cycling, skiing, reading, cooking) rarely become a point of
discussion. Specificity creates connection: "traditional Bündner
cuisine" or "16th-century British literature" catches attention and
adds personality. Where possible, replace generic hobby labels with
specific variants. Additionally, consider creating a separate
"Extracurricular Activities" section for professional or civic
involvement (guild memberships, board positions, professional society
roles) to give these appropriate weight.
