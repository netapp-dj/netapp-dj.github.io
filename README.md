# StorageGRID Onboarding Redesign

A proposal for restructuring how new hires are onboarded to the StorageGRID engineering team — built around the principle of fewer firehoses, more flow.

## What this is

The existing onboarding Confluence pages (SGWS space) contain solid, hard-won content accumulated over years. This project doesn't replace that content — it restructures *when and how* it reaches a new hire, so that each piece lands with context behind it rather than all at once on Day 1.

The output is a single-page interactive proposal (`index.html`) and a companion markdown document (`index.md`) covering:

- What's working and what's not in the current flow
- A five-phase ramp (Before Day 1 → Week 3+) with explicit daily/weekly pacing
- A QA/Dev role split that starts on Day 2, not Module 4
- Eight AI-augmentation opportunities ranked by effort and impact
- Non-AI structural fixes (urgency tiers, buddy prep guide, site-specific callouts, dead link audit)
- A "keep as-is" list so good work isn't accidentally discarded

## Source material

Based on a full read of the Confluence onboarding tree rooted at:
[Onboarding: StorageGRID New Hire](https://netapp.atlassian.net/wiki/spaces/SGWS/pages/292749763/Onboarding+StorageGRID+New+Hire)

Including all child pages:
- Module 1 — Getting Started with VED or BigTop VPN
- Module 2 — Git Exercise
- Module 3 — StorageGRID Setup
- Module 4 — AWS S3 Basics
- QA Module 1 — Test Automation and MrClean
- QA Module 2 — Additional Resources
- Dev Modules 1–5
- Terminology / Key Information and Terms

## Proposed phase summary

| Phase | When | Focus |
|---|---|---|
| P0 | Before Day 1 | Pre-read brief, access requests, IP reservation |
| P1 | Day 1 AM | Badge, password, MFA — the three real blockers |
| P2 | Day 2–3 | Dev environment, role split begins (QA runner vs cycl) |
| P3 | Day 3–4 | Product understanding first: primer, reference grid, curated videos |
| P4 | Week 1 Days 4–5 | Git exercise, first PR to MrClean |
| P5 | Week 2 Days 1–4 | Your own grid (Module 3) + S3 basics on your grid (Module 4) |
| P6 | Week 2–3 | Role-specific deep-dive → active sprint work |

## AI opportunities highlighted

| Opportunity | Effort | Impact |
|---|---|---|
| Personalised skills-mapping brief (resume → SG concepts) | Medium | High |
| AI onboarding chatbot (Claude + RAG on SGWS Confluence) | High | High |
| Grid doctor validation script | Medium | High |
| .ini/.json editor guard (IP conflict detection) | Medium | High |
| AI PR reviewer on MrClean (partially in place) | Low | High |
| Self-check quizzes after each module | Low | Medium |
| Video summariser (Whisper → Claude) | Medium | Medium |
| Contextual glossary tooltips in Confluence | Low | Medium |

## Files

| File | Purpose |
|---|---|
| `index.html` | Interactive single-page proposal with collapsible phases, persistent progress checkboxes, dark/light theme |
| `index.md` | Markdown version of the same proposal — useful for Confluence import or plain reading |
| `README.md` | This file |

## Viewing

The HTML page is self-contained — open `index.html` directly in a browser or serve it via GitHub Pages. Progress checkboxes persist via `localStorage` so you can track which sections you've reviewed across sessions.

## Contributing

If you're a buddy, manager, or team lead and spot something wrong or missing, the markdown (`index.md`) is the easiest place to make edits. Structural or visual changes go in `index.html`.

---

*Author: DJ Rajkhowa — Reviewer: Claude — 2026-05-13*
