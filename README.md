# Building software with a crew of AI agents

> *Companion repo for an **AI Tinkerers** talk — building real software with a crew of Claude
> Code agents, not one agent in a chat box.*

## The talk

**A dark factory for code: an autonomous, multi-agent build crew in Claude Code.**

I describe what I want in plain language, and a crew of Claude agents does the labour —
designing, building, testing, and double-checking — with a single human yes/no before anything
lands on `main`. Work is batched into *waves*: each wave splits into tasks, and every task runs
a fixed five-stage cycle — **brainstorm → implement → test → review → wrap-up** — as its own
Claude Code subagent in its own git worktree, so many build in parallel without colliding. The
factory runs each agent as a full interactive Claude session in a **tmux** pane (rather than
headless `claude -p`), signalled complete by a Stop-hook; I drive and watch the windows through
**cmux**. The whole run is journaled to an on-disk ledger, so it survives reboots and resumes
where it left off.

The quality net is a Claude Code skill I lean on constantly — a **talmudic study**, where *N*
orthogonal expert agents each write an independent position on a design, then argue it out
(filing challenges, conceding points, revising) until a redactor synthesizes one settled design
with the dissent preserved. The factory runs two of these panels over every finished wave and
auto-spawns corrective tasks for anything they catch. Then it parks the wave at a "front desk"
and asks the only question I ever have to answer: merge, or hold? I'll show how it's wired and
share the two pieces that do the most work — the talmudic-study skill, and the plain-language
map of the factory itself.

Two things are shared here. Both are deliberately **product-agnostic** — the patterns work on
any codebase.

---

## 1. The dark factory — the system

📄 **[`dark-factory.md`](dark-factory.md)** — the full plain-language map.

A **wave-based, multi-agent build system.** You describe a goal; it splits the work into tasks
and runs each through a fixed five-step cycle — **brainstorm → implement → test → review →
wrap-up** — as its own Claude Code subagent, in its own git *worktree*, so many tasks build at
once without stepping on each other. When the batch (a *wave*) finishes, two expert panels
study the whole thing end-to-end and the factory auto-fixes anything they catch. Then it parks
at a **front desk** and asks you one question: merge, or hold?

**Nothing touches `main` until you say yes.** You never have to open a file — you describe what
you want, answer the few questions only you can answer, and give the final approval.

The whole thing lives on disk (the wave files + an append-only ledger), so it survives reboots
and ended sessions and picks up where it left off. The full layout, the lifecycle of a wave,
and the handful of commands you actually use are in **[`dark-factory.md`](dark-factory.md)**.

## 2. The talmudic study — a skill you can use today

📄 **[`talmud/SKILL.md`](talmud/SKILL.md)** — a drop-in Claude Code skill.

A skill for **planning anything that needs careful speccing before you build it.** It spins up
*N* orthogonal expert agents who each write an independent position, then argue — filing
challenges, conceding points, revising their own view — and a *redactor* synthesizes one
**settled design with the dissent preserved**, so you can see exactly what's still contested
and what's only settled-with-an-asterisk. A study where everyone agrees is a failed run;
disagreement is the product.

It's the quality net the factory runs on every finished wave — and it's just as useful on its
own, the moment before you start a non-trivial piece of work.

### Install

Copy the `talmud/` directory into your Claude Code skills folder:

```bash
cp -r talmud ~/.claude/skills/
```

### Use

In a Claude Code session:

```
/talmud <topic> <path-to-seed-doc>
```

Optional flags: set the number of experts (default 7), add `--dossier` for a code-verified
grounding pass before the debate, or `--parallel-b` to run two fully independent studies and
compare them (convergence across both = a strong signal). The full method, the output-document
structure, and the quality bars are in **[`talmud/SKILL.md`](talmud/SKILL.md)**.
