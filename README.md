# CFA Society Switzerland – CV Review AI Agent

https://amarows.github.io/cfa_skills/cfa_cv_review.html

An AI-assisted CV review service for the CFA Society Switzerland Careers Committee.

## What This Is

The Committee helps members and candidates find employment in the Swiss financial market. This project systematizes that process into a repeatable, AI-assisted workflow – reducing manual effort for Committee members while improving consistency and quality of feedback for Clients.

## Repository Structure

```
cfa_skills/
├── CLAUDE.md                          # Project charter – targets, architecture, roadmap
├── README.md                          # This file
└── skills/
    ├── cv_review_generic/
    │   └── SKILL.md                   # CV review when no target role is known
    └── cv_target_role_score/
        └── SKILL.md                   # CV scoring against a specific target role
```

## How It Works

1. A Committee member uploads a Client CV to the AI agent.
2. The agent applies the relevant skill – generic review or role-targeted scoring.
3. The agent produces a structured draft: dimension scores, gap analysis, priority recommendations.
4. The Committee member reviews, edits if needed, and delivers to the Client.

## Differentiation

This tool is specifically designed for the **Swiss financial industry**. It encodes Swiss market norms (work permit signalling, language requirements, FINMA regulatory context, certification prominence) and financial industry hiring criteria that generic CV tools do not address.

## Project Status

**Phase 0 – Build and calibrate** (current)

See [open issues](https://github.com/Amarows/cfa_skills/issues) for the current backlog.

## Contact

Maintained by the CFA Society Switzerland Careers Committee.
