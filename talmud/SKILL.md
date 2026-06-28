---
name: talmud
description: Run an intense talmudic study — N orthogonal expert agents independently thesis, disputate, revise, and a redactor synthesizes a settled design with preserved dissent. For planning anything that needs careful speccing before implementation. Args; topic + seed doc path; optional N (default 7), --dossier, --parallel-b.
---

# Talmudic Study

You are running a **talmudic study**: N orthogonal expert agents enter intense structured disputation on a design topic, producing a study document the user rules on. It is a planning pattern for anything that needs careful speccing before implementation. Follow the structure and rigor below exactly.

## Arguments

- **topic** (required) — what the study is about, e.g. `web-enrichment`.
- **seed** (required) — path to the seed/brainstorm doc. If the user gives only a topic, look for `brainstorm-<topic>.md` in the cwd or session directory, else ask.
- **N** — number of experts (default 7). The user's phrasing "X orthogonal expert agents…" sets N.
- **--dossier** — run a code-verified dossier pass first (default ON when the topic touches existing code; OFF for green-field).
- **--parallel-b** — also run a second fully independent study with a different expert composition, then write a comparison doc (convergence across two independent studies = strong signal).

## Inputs to assemble before anything else

1. Read the seed doc fully.
2. Read every decisions doc the seed or its session index links as settled context (prior sessions' rulings are **binding constraints**, not open questions).
3. Collect the user's inputs stated in the current conversation — these are baked into every expert brief as requirements.

## Expert design (the heart of the method)

Design N **named personas, each holding one orthogonal professional lens** chosen to maximize productive friction *on this topic*. Orthogonal means: their failure modes differ; where one is blind, another stares. Persona names are evocative role-titles (precedent: Boundary Warden, Systems Scheduler, Ledger Bookkeeper, Pattern Statistician, Model Atomizer, Data Quality Healer, Product Choreographer / Type Theorist, Governance Counsel, Temporal Epistemologist, Storage Pragmatist, Workplace Ontologist, Minimalist Maintainer, Federation Futurist). Pick disciplines for THIS topic — never reuse a roster unexamined. State each persona's lens in one line.

## Procedure (orchestrate with the Workflow tool when available; otherwise parallel Agent calls)

The canonical run is **2N+2 agents** (dossier 1 + round-1 N + round-2 N + redactor 1):

1. **Dossier (1 agent, if enabled)** — reads the *actual code/docs* the design touches and returns a numbered list of code-verified **surprises**: facts that invalidate naive assumptions (file:line cited). The dossier grounds disputation in reality, not memory.
2. **Round 1 — independent theses (N parallel agents)** — each expert receives: seed + binding decisions + user inputs + dossier + ONLY their own persona. No exposure to other experts. Each writes a complete position: what the design should be, what in the seed is wrong, what is missing, with concrete mechanisms (schemas, predicates, key formats, thresholds — not vibes).
3. **Round 2 — disputation + revised theses (N parallel agents)** — each expert receives ALL round-1 theses and must: file **kushyot** (challenges) against specific claims of named others; answer challenges aimed at their own thesis (a kushya not answered is recorded as such); then write a revised thesis explicitly noting *what strengthened, what narrowed, what fell, what remains unresolved*. Concessions are named ("The Warden conceded the dirty-set design").
4. **Redactor (1 agent)** — receives everything and writes the study body:
   - extracts the named bilateral disputes (numbered: "Dispute 1 — <thing> (<Expert> vs <Expert>)", each with *what turns on it* and *resolved within the study* vs *left open for the user*);
   - synthesizes **the settled design**, tagging every element `[settled]` or `[settled-with-dissent: <who> — <concern>]`; dissent is preserved even on settled items;
   - compiles the tunables table and the questions only the user can answer **with a recommendation per question**.

## Output document

Write ONE file: `<study-dir>/YYYY-MM-DD-<topic>-study-<n-word>-experts.md` (e.g. `…-study-seven-experts.md`), where `<study-dir>` follows your local session convention (alongside the seed; e.g. `session-N-<topic>/`). Structure — match exactly:

```
# <Topic> — <n-word>-expert study (session N)

**Date:** YYYY-MM-DD
**Method:** <total> agents — [code-verified dossier,] <N> orthogonal experts (<names>) writing
independent positions, full disputation round, redactor with disagreements preserved. Run <id>.
**Inputs:** [seed](…) + [binding decisions](…) + user inputs (<list>) — all baked into expert
briefs as requirements.
**Status:** study output. The user's rulings land in <decisions-doc-name> — where they differ, the
decisions doc supersedes. This body is a historical record — do not edit.

## The <topic> verdict — one paragraph
## The <N> positions            (one dense paragraph per expert, post-disputation)
## The disputation              (numbered bilateral disputes)
## The settled design           ([settled] / [settled-with-dissent: …] tags)
## Tunables registry additions  (param | default | signal-to-adjust)
## Questions only the user can answer   (numbered, each with the study's recommendation)
## Handoffs                     (named handoffs to downstream sessions/specs)
# Appendix A: the <N> theses (round 1)     (verbatim)
# Appendix B: revised theses (after disputation)   (verbatim)
# Appendix C: dossier surprises (code-verified)    (if dossier ran)
```

If `--parallel-b` ran: also write the B study (different roster, e.g. three competing designers) and `YYYY-MM-DD-<topic>-comparison.md` with a scoreboard table (dimension | Study A | Study B | CONVERGED/DIVERGED), genuine divergences (D1…, each with which-is-sounder + "Your call:" framing), adopted-from-exactly-one-study, and merged open items.

## Closing the session

End by presenting the **Questions only the user can answer** verbatim in the conversation and stop. Do NOT write the decisions doc — that is the user's ruling step. Offer to scaffold `YYYY-MM-DD-<topic>-decisions.md` with its deep-dive-entry-point manifest header (listing seed → study → decisions in reading order) once the user has ruled. If this study belongs to an indexed session series (e.g. an `OVERVIEW.md`), remind the user to flip the index row to ✅ after the decisions doc lands.

## Quality bars

- Experts argue with mechanisms, not adjectives. Every claim about existing code must be file:line verifiable (the dossier exists to enforce this).
- Disagreement is the product: a study where all N agree on everything is a failed run — the personas were not orthogonal. Note it honestly rather than manufacturing consensus.
- The user's input requirements are non-negotiable inside the study; experts may flag costs but not silently drop them.
- Budget guidance: round agents are substantial (thesis ≈ 1–3k words each); use opus-class models for the redactor at minimum.
