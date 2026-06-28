# The Dark Factory — a wave-based multi-agent build system, for humans

The dark factory is a **software-building crew made of AI workers**. You decide *what* to
build and approve the parts that matter; the crew does the labour — designing, building,
testing, and double-checking the work — and brings it back to you for one yes/no before
anything lands on `main`. This is the map, in plain terms.

It's deliberately **product-agnostic**: every domain-specific value lives in a single
config file, so the same machinery can be pointed at any codebase. Everything below is
about the machinery itself, not any one thing it happens to build.

> **One idea to hold onto: the _wave_.** A wave is one batch of related work — "add
> rate-limiting to the public API", say. You describe the goal; the factory designs it,
> builds it on its own branch, has two expert panels double-check it, and parks it for your
> approval. **Nothing touches `main` until you say yes.**

## The mental model

- **You** — the architect. You describe what you want, answer the few questions only you
  can answer, and give the final yes/no. You never have to open a file.
- **A wave** (`waves/<name>.yaml`) — one batch of work, split into **tasks**. Each task
  knows what "done" looks like (its plain-language `acceptance`) and which other tasks it
  waits on.
- **The crew** — AI workers. Each task gets its own worker and its own private copy of the
  code (a *worktree*), so several build at once without stepping on each other.
- **The studies** — when a wave finishes building, two panels of experts read the whole
  thing over (a "talmudic study"), and the factory auto-fixes anything they catch.
- **The front desk** — the one place you interact. It shows you a finished wave in plain
  language and asks: merge, or hold?

Everything lives on disk (the wave files + an append-only ledger), so the whole thing
survives reboots and ended sessions and picks up where it left off.

## The life of a wave — step by step

**1. You describe what you want.** Run **`/design-waves <your goal>`** in a Claude
session. You don't write a spec — you say the goal in your own words.

**2. The factory designs it.** `/design-waves` reads your current code and direction, asks
you the handful of questions that actually shape the work, writes up the new direction,
runs a deep-research **study** on *how* to build it, then drafts the wave — a list of
tasks, each with a plain-language "what done looks like". It red-teams its own plan, fixes
what it finds, and shows you the result in plain language. **You approve → the build
starts automatically.**

**3. It builds.** Each task runs a fixed **five-step cycle** — brainstorm → implement →
test → review → wrap-up — inside its own private copy of the code. A task that depends on
another waits until that one is done. Each finished task becomes one tidy commit on the
wave's own branch. Your `main` is never touched.

**4. It double-checks itself.** When every task is built, the wave **closes**: two expert
panels study the finished work end-to-end (the second can consult the first), a writer
distils their verdict for you, and the factory spins up **corrective tasks** to fix
anything real they found — then re-checks.

**5. You approve the merge.** The wave parks at the **front desk** with a plain-language
summary. Open a session and run **`/factory-frontdesk`** (or just ask): it shows you what
the wave did, whether it holds up, and a recommendation. You pick **Approve** (it merges
to `main` for you) or **Hold** (it stays parked, to fix in a follow-on wave). That is the
only command you ever *have* to give.

> A big goal can be several waves that depend on each other. The runner builds them in the
> right order — a wave only starts once the waves it builds on have merged.

## The map — where things live

```
factory/
  waves/         the work you've defined — one <name>.yaml per wave (schema in waves/README.md)
  config.yaml    every domain/operator-specific value (the one file to edit to point the factory at a different codebase)
  frontdesk.py   the front-desk helper (drives the factory-frontdesk skill)
  README.md      this map  ·  dark_factory.md  the deeper manual  ·  design/  the design docs

  engine/        the build system itself — runs as `python3 factory/engine/<module>.py`
    wave_runner.py     runs a wave: dispatches tasks, reaps them, keeps the order
    wave_close.py      the two studies + corrective tasks + the approval gate
    wave_git.py        the branch / worktree / squash / merge plumbing
    wave_model.py      the wave schema + the append-only ledger (pure, unit-tested)
    factory_core.py    shared low-level plumbing (tmux, session-ids, notifications, OAuth)
    factory_config.py  loads config.yaml

  hooks/         auto-run by Claude Code (wired by absolute path in .claude/settings.json)
    stop_hook.py          detects when a worker finishes
    log_post_tool_use.py  records what workers do
    ruff_hook.py          auto-formats the Python the workers write

  scripts/       things you run by hand
    sync_skills.sh   copies the shared skills to ~/.claude/skills
    log_rotate.py    weekly housekeeping on the activity log

  redteam/       safety-audit reports (from an ad-hoc /redteam pass)
  tests/         the engine's own test suite — cd factory && python3 -m pytest tests/ -q
```

(Transient, gitignored: `.notify/`, `.wave-briefs/`, `.agent-data/`, `wave_ledger.jsonl`,
`log.jsonl`, `log-archive/`; `bubble_ups.md` is the plain-language decision record.)

## The commands you actually use

Almost always, just two slash commands in a Claude session:

```
/design-waves <goal>     # describe what to build → it designs, gets your OK, then builds it
/factory-frontdesk       # the front desk: shows finished waves, you approve or hold
```

Under the hood (the skills run these for you — you rarely type them):

```bash
python3 factory/engine/wave_model.py  validate factory/waves/<name>.yaml   # check a wave file
python3 factory/engine/wave_runner.py run      factory/waves/<name>.yaml   # build one wave
python3 factory/engine/wave_runner.py run-waves                            # build every wave, in dependency order
python3 factory/frontdesk.py poll                                          # what's awaiting you
python3 factory/frontdesk.py approve-wave <name>                           # merge an approved wave
```

For an occasional safety sweep outside the wave flow, run the **`/redteam`** skill in a
session. There is no nightly job and no always-on daemon — **the two per-wave studies are
the routine safety net.**

## Load-bearing — handle with care

- **The hooks are wired by absolute path** in `.claude/settings.json` (`factory/hooks/*.py`).
  Moving or renaming one breaks the wiring — update both sides together.
- **The engine modules import each other by bare name** and locate the repo root by
  walking up to `factory/`, so they keep working wherever they sit.
- **`config.yaml` is the one file to edit** to point the factory at a different codebase —
  don't hardcode domain-specific values back into the code.
- **Never start a red-team pass while a wave is building** — its startup sweep kills every
  `factory-*` tmux session, including the wave's own workers.

## Where the depth is

- **`design/`** — why the wave model is shaped this way (`wave-design.md`,
  `wave-execution.md`, `build-plan.md`).
- **`waves/README.md`** — the exact wave-file schema.
- **`dark_factory.md`** — the operating manual (deeper mechanics and history).
