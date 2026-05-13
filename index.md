# StorageGRID New Hire Onboarding — Redesign Proposal

*Fewer firehoses, more flow*

**Author:** DJ Rajkhowa | **Reviewer:** Claude  
**Date:** 2026-05-13  
**Based on:** SG Onboarding + child pages  
**Audience:** Hiring managers, onboarding buddies, and the team maintaining this content

---

## TL;DR

The existing onboarding is thorough but front-loads too much at once — admin setup, environment config, product theory, and tooling all compete for a new hire's attention in the first two days. This proposal restructures the flow into five phases spread over ~three weeks, makes the QA/Dev track split happen at Day 1 (not Module 4), and identifies specific places where AI can reduce friction without replacing the human elements that make onboarding enjoyable.

---

## What's Wrong With the Current Flow

| Problem | Where It Appears | Impact |
|---|---|---|
| Day 1 is 12+ procedural steps with no urgency ranking | Main onboarding page §2 | Decision fatigue, new hire spends half the day on printers |
| Terminology dumped as a wall of text before context exists | Terminology child pages | Memorization without meaning — forgotten within a week |
| Product intro is just links, no narrative | Main page §3-4 | New hire has no mental model before diving into modules |
| QA vs Dev track split happens too late (Module 4+) | QA Modules / Dev Modules | QA new hires still do generic setup modules without understanding why |
| Module 3 references 11.2 docs and outdated Confluence URLs | Module 3 §2 | Wrong link → confusion → buddy interrupt |
| Videos listed without durations, dead links, no curation | Main page §4 | No one watches them |
| "Ask your buddy" appears ~20 times with no buddy prep guide | Throughout | Buddies are surprised by repetitive questions |
| No explicit daily/weekly pacing | Entire doc set | New hire has no sense of progress |
| GenAI tools buried in a table cell | Main page §2.1 | Missed entirely — huge missed opportunity |
| VTC-specific content not flagged for other sites | §2 printers, mailing lists | RTP / NB new hires get confused |

---

## Architecture at a Glance

A new hire who understands how the pieces connect before touching any tooling will absorb Module 3 and Module 4 in a fraction of the time. These diagrams come from the **StorageGRID Repository Architecture Overview** (SGWS Confluence, March 2026). The doc itself recommends starting with the Executive Summary and Sections 2–3 for onboarding purposes.

**System Context — two entry points:**
- **Management plane** (UI + API): Grid and Tenant Admins connect over HTTPS to the Management UI, which talks REST to the Management API
- **Data plane** (Gateway → Storage): S3 clients connect to nginx-gw (Gateway node), which routes to LDR on Storage nodes

**Node types and what runs on each:**

| Node | Role | Key components |
|---|---|---|
| **Admin** | Brain + API | mgmt-ui, mgmt-api, gpt2, mi (config bundles), MySQL |
| **Gateway** | S3 front door | nginx-gw, cache-svc |
| **Storage** | Data + metadata | LDR (ade), chunk storage, Cassandra |

**S3 data path (what happens when you run `aws s3api get-object`):**
1. Request hits `nginx-gw` on a Gateway node
2. If caching is enabled and it's a hit → `cache-svc` returns the object directly
3. Otherwise → forwarded to LDR on a Storage node
4. LDR queries Cassandra for metadata and location
5. LDR reads chunks from disk and streams the response back

**Key repo components — a code atlas:**

| Layer | Components | What they do |
|---|---|---|
| Management | `mgmt-ui`, `mgmt-api` | Angular UI + Ruby/Sinatra REST API |
| Gateway / Data path | `nginx-gw`, `cache-svc` | S3 entry point, load balancing, object cache |
| Storage engine | `ade`, `chunk`, `dmv`, `rsm` | LDR, chunk I/O, data movement, replication |
| Config & metadata | `mi`, `gator`, `gpt2`, Cassandra | Bundle distribution, provisioning, accounts |
| Observability | prometheus, grafana, alertmanager, `audit-analysis` | Metrics, dashboards, alerts, audit logs |
| Platform | `stem-cell`, `container-services`, `servermanager` | Node images, container workloads — all on Debian |

---

## Proposed Five-Phase Flow

### Phase 0 — Before Day 1 (Buddy/Manager sends this)
**Goal:** New hire arrives with mental model already forming, not starting cold.

- [ ] Email the new hire a single 1-page "What to expect" brief
- [ ] Attach the StorageGRID Grid Primer link (30-min read, works outside VPN)
- [ ] Share one intro video (David Leung's product overview)
- [ ] Pre-create access requests for Bitbucket, Mantis, Jira (these take days to process)
- [ ] Reserve a block of IPs in ICT/RTP so the new hire is unblocked on Module 3

**AI Opportunity:** Use an LLM to auto-generate a personalised "what you already know that maps to StorageGRID" brief based on the new hire's resume/background. e.g., "You know Kubernetes distributed state — here's how StorageGRID's metadata Cassandra cluster is conceptually similar."

---

### Phase 1 — Day 1: Human Setup (≈ half-day)
**Goal:** Get connected to people and systems. No technical setup yet.

**Urgent blockers only:**
- [ ] ID badge (WPR)
- [ ] Initial password change
- [ ] MFA enrollment (blocks VED access)
- [ ] Laptop enrollment (macOS: Self Service App)

**Social onboarding:**
- [ ] Buddy intro + office tour
- [ ] Team introductions
- [ ] Join: Teams channel for StorageGRID, your scrum team's channel
- [ ] Join: your scrum team's mailing list (ng-sg-`<team>`)
- [ ] Bookmark: MyNetApp, HR portal, Canada Benefits, IT Front Door

**Defer to Day 2:**
- Printers, Zoom SoftPhone, Outlook rules, shell default (csh→bash)

**AI Opportunity:** An AI onboarding chatbot (Claude-backed) pre-loaded with SG-specific FAQs, org chart, and the Confluence space can field "what's the Jira URL again?" questions so the buddy isn't interrupted every 15 minutes.

---

### Phase 2 — Day 2–3: Development Environment
**Goal:** Get into the lab. Know your tools. Make your first SSH connection.

**Path split starts here:**

| Step | QA | Dev |
|---|---|---|
| VED vs BigTop | BigTop (more common for QA) | Either — ask buddy |
| Runner setup | Set up shared QA runner | Set up cycl server |
| Shell | Change default to bash (enghelp@netapp.com) | Same |
| Key tools | Jira, Bitbucket, MrClean, OpenGrok, Jenkins, Mantis | Jira, Bitbucket, SG repo, OpenGrok, Jenkins |

**Curated tool bookmarks (replace the current 20-item table with this short list):**

| Tool | Why You Need It |
|---|---|
| Jira | Your sprint work lives here |
| Bitbucket | All code and PRs |
| OpenGrok | Search codebase without cloning everything |
| Jenkins | Automated test runs on your PRs |
| Mantis | Bug tracking across releases |
| Confluence (SGWS) | Team documentation home |

**Non-AI Improvement:** Add a validation script that checks whether all tools are accessible (can curl each URL from runner, checks git config, checks `aws` is installed). New hire runs one script, gets a green/red per tool. Eliminates the "I wasn't sure if my Bitbucket access worked" ambiguity.

---

### Phase 3 — Day 3–4: Product Understanding First, Modules Second
**Goal:** Understand what you're building before touching more code.

**Curated learning sequence (replaces the unstructured videos section):**

| Resource | Format | Time | What You'll Learn |
|---|---|---|---|
| **Architecture Overview** — Executive Summary + Sections 2 & 3 | Reading + diagrams | 20 min | How the whole system fits together; node types and what runs where |
| Grid Primer (latest docs) | Reading | 45 min | Nodes, tenants, ILM — operational detail behind the diagrams |
| David Leung's SG fundamentals video | Video | ~45 min | Architecture, key design decisions |
| Explore a reference grid (ICT) | Hands-on | 30 min | Grid Manager UI, tenants, ILM rules |
| NB team onboarding Zoom recordings | Video | Pick 2-3 relevant ones | Feature-specific deep dives |

**Why reference grid first, own grid later?** 
New hires can explore a pre-built grid through the Grid Manager UI on Day 3 — clicking around ILM, Tenants, Dashboard — and build intuition before Module 3 asks them to install their own. This makes Module 3 feel like a confirmed understanding rather than a blank-slate setup.

**Terminology approach (replace the wall-of-text page):**
Instead of a static table, keep the Initialisms/Key Terms pages as a *reference* you link from each module contextually. Don't assign it as required reading. Add a sidebar "glossary" link to each module page.

**AI Opportunity:** AI-generated self-check quizzes after the primer read. 5 questions like "What is the minimum number of storage nodes for data redundancy?" with explanations. No grade — just reflection. Keeps learning active rather than passive.

---

### Phase 4 — Week 1 (Days 4–5): Git Exercise + First Contribution
**Goal:** Make your first real contribution. Understand the review culture.

This maps to the existing Module 2, which is actually well-structured. Improvements:

- [ ] Contextualize *why* MrClean exists before the exercise (one paragraph: "This is the automation repo that verifies StorageGRID behaves correctly for customers — every test you write prevents a future bug.")
- [ ] Add a "what makes a good PR description" section — the current guidance is buried at §6.1 and easy to miss
- [ ] Flag the `jenkins run <test>` trigger prominently — new hires often don't know about it until their first real PR
- [ ] Add: how to trigger a presub re-run (currently hidden in §3.5)

**Non-AI Improvement:** Create a `new-hire-checklist` branch in MrClean where the print_name.spec.rb file already has a well-commented example, so the new hire understands the test structure before editing.

---

### Phase 5 — Week 2: Your Grid + S3 Basics
**Goal:** Own your development environment. Ingest your first object.

Modules 3 and 4 run sequentially — this order is correct. Improvements:

**Module 3 — Grid Setup:**
- Remove the "OUTDATED, USE ICT values here" banner and just link directly to the ICT VMware Deployment Guide (eliminate the confusion)
- Add explicit "VTC" / "RTP" tabs or callouts so site-specific steps are obvious
- Emphasise `deploy_tools` as the primary path — the manual .ini/.json edit is a learning exercise, not the production workflow
- Dead link audit: replace all 11.2 doc links with the current version equivalent

**Module 4 — S3 Basics:**
- Before starting Module 4, revisit the S3 Data Path diagram in the Architecture section — it turns the CLI exercise from "follow these steps" into "I know which service is handling my request"
- Add the AWS config symlink pattern earlier (it's a better practice than `aws configure` — bury the latter)
- Add a "common S3 test patterns" section showing what MrClean tests look like in relation to these commands — closes the loop from the Git exercise

**AI Opportunity:** An AI "grid doctor" script — after the grid is installed, a new hire pastes their admin IP and the AI checks: grid health, tenant count, ILM policy, storage node status, and flags anything that looks wrong. Replaces the current "navigate and check out the features" instruction with structured validation.

---

### Phase 6 — Week 2–3: Role Track Deep-Dive

#### For QA (maps to QA Module 1):
- [ ] MrClean walkthrough with buddy (1-hour live session, not a doc)
- [ ] Run your first presub test against your own grid
- [ ] Shadow the nightly regression triage for one day before doing it solo
- [ ] Mantis workflow: shadow a `resolved → qa → tested` cycle on a real ticket
- [ ] Write your first new test (small scope — a new parameter on an existing test)
- [ ] First solo PR to MrClean

**Move to "Additional Resources" (not onboarding):** Cassandra internals, s3tester, advanced IO workload config. These are Week 4+ topics, not Week 2.

#### For Dev (maps to Dev Modules):
- [ ] Clone and build the SG repo
- [ ] Dev Module 3 (Grid Setup for Dev — cycl server)
- [ ] Read ADE introduction
- [ ] First story on the scrum board (shadow buddy first sprint)

---

## AI Opportunities Summary

| Opportunity | Effort | Impact | Tool |
|---|---|---|---|
| Personalised "skills mapping" brief (resume → SG concepts) | Medium | High | Claude API + HR data |
| AI onboarding chatbot (SG FAQ, org, Confluence search) | High | High | Claude + RAG on Confluence |
| AI self-check quizzes after each module | Low | Medium | Claude API |
| Video summarisation for internal Zoom recordings | Medium | Medium | Claude + Whisper transcription |
| Grid doctor validation script | Medium | High | Claude Code / scripted |
| AI-assisted .ini/.json editor (detect IP conflicts, flag mistakes) | Medium | High | Claude Code |
| AI PR reviewer (already partially in place via Claude Code) | Low (done) | High | Claude Code ✓ |
| Glossary tooltip generator from Confluence pages | Low | Medium | Claude API |

---

## Non-AI Structural Improvements

1. **Urgency tiers on Day 1**: Separate "Must do before lunch" from "Before end of week" from "Whenever"
2. **Buddy prep guide**: A parallel checklist for the buddy (what to prep before Day 1, what to cover in Week 1 1:1s)
3. **Site-specific callouts**: Clearly flag VTC vs RTP vs NB steps — don't bury them in general prose
4. **Update dead content**: Module 3 still references SG 11.2, old wikid URLs, old confluence.eng.netapp.com paths
5. **Video curation**: Remove dead links, add durations, add a 1-sentence "what this covers" for each
6. **Progress tracker**: Interactive HTML checklist with localStorage so new hires can pick up where they left off (even across sessions)
7. **Feedback loop**: 30-day and 60-day survey for new hires — what was confusing, what was missing, what was great
8. **Rookie Cookies formalise**: The "Rookie Cookies" tradition is buried in a sidebar note in QA Module 2 — it should be in the main Day 1 page so everyone knows about it, not just QA

---

## Proposed Module Sequence (Revised)

```
Before Day 1       → Pre-read brief (buddy sends)
Day 1 AM           → Human setup (badge, password, MFA, introductions)
Day 1 PM           → Communications (Teams, mailing lists, bookmarks)
Day 2              → Dev environment (VED/BigTop + runner, role split begins)
Day 3              → Product understanding (primer, videos, reference grid)
Day 4-5            → Git exercise + first PR (Module 2)
Week 2, Days 1-2   → Your own grid (Module 3)
Week 2, Days 3-4   → S3 basics on your grid (Module 4)
Week 2-3           → QA: MrClean deep-dive, nightly triage shadow
                   → Dev: Repo build, first story
Week 3+            → Active sprint work, first solo ticket
```

---

## What to Keep (It's Good)

- The Git exercise (Module 2) is well-structured and has a real deliverable — keep it
- The tool link table in Module 1 is useful — trim to 6-8 rows and keep it
- The Mantis workflow steps in QA Module 2 are clear and actionable — keep them
- The "Common Passwords" table in Module 3 is a lifesaver — keep it prominently
- The Rookie Cookies tradition — keep it and promote it
- The buddy system model is the right foundation — everything should enhance it, not replace it
