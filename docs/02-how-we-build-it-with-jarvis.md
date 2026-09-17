# 02 — How we build it with Jarvis

This is the concrete design: one agent, one laptop, one spreadsheet. It is deliberately small. The
sources that describe large factories (Stripe, Uber, Cloudflare) solved problems we do not have;
what transfers is the *shape*, and the shape fits on one machine.

## 1. Three state layers, one writer each

The single most useful architectural decision we made, because it removes an entire class of bug:
stale copies.

| Layer | Holds | Writer | Never |
|---|---|---|---|
| **Identity + hands** — Hermes (`Jarvis`) | skills, memory, OS control, delegation, the terminal and browser | the agent itself | task status |
| **Task state** — Paperclip | issues, assignments, runs, budgets, cost | Paperclip (and the harness that runs seats) | a mirror in markdown |
| **Knowledge** — Shared Living Memory | durable knowledge with provenance and links | the agent, deliberately, entry by entry | ephemeral task progress |

Corollary: **the agent does not keep its own task list in a markdown file.** The moment you have
two sources of truth for "what is in progress", the markdown one goes stale and late decisions get
made from it. The corpus says the same thing: external state engine + label taxonomy beats
markdown state files.

## 2. The human surface is one spreadsheet

The requirement we set ourselves: *the only thing a human maintains is the input rows.*

A row is intent, not instructions. The sheet has two blocks — yours on the left, generated on the
right — and the generator never asks you to fill in the right side.

**Left block — yours**

| Col | Field | Req | Meaning |
|---|---|---|---|
| A | `type` | ✅ | must match a declared task type |
| B | `title` | ✅ | one line; becomes the issue title |
| C | `intent` | ✅ | **your words**, copied verbatim into the brief |
| D | `context` | – | absolute paths / URLs, one per line |
| E | `done_when` | ✅ | binary checks, one per line — this *is* the definition of done |
| F | `priority` | ✅ | `P1` / `P2` / `P3` |
| G | `owner` | ✅ | who asked; drives attribution |
| H | `notes` | – | free text for humans; the machine ignores it |

**Right block — generated, do not edit**

| Col | Field | Values |
|---|---|---|
| I | `status` | `queued` · `running` · `blocked` · `review` · `done` · `failed` · `needs-info` |
| J | `issue` | `FRA-nn` |
| K | `artifact` | absolute path(s) of the deliverable |
| L | `cost` | tokens · wall-clock |
| M | `factory_note` | validation or stop reason |
| N | `updated` | ISO timestamp |

Why a spreadsheet and not a tracker: it is the lowest-friction surface a non-engineer already uses,
it is diffable, it survives the tooling, and it forces the one thing that actually determines
output quality — **writing the intent and the definition of done before the work starts.**

Read with the standard library only (`zipfile` + `xml.etree` over the `.xlsx` container): no
dependency, no daemon, and it keeps working on a machine where you refuse to install a Python
package ecosystem for a spreadsheet.

## 3. The flow

```
  sheet row
     │
     ▼
[ timer ]  systemd --user, Persistent=true (a missed run fires after wake)
     │
     ▼
[ validate ]  row complete? type known? done_when binary? → else write `needs-info` back
     │
     ▼
[ route ]  type → seat + brief template + deliverable path + gates + caps   (a table, not a model)
     │
     ▼
[ dispatch ]  create the issue, assign the seat, attach the brief
     │
     ▼
[ run ]  isolated workspace per run · caps enforced · heartbeat
     │
     ▼
[ gates ]  exit codes: tests, schema, path-scope, artifact presence
     │
     ▼
[ write back ]  status · artifact · cost · note · timestamp
```

Two rules make this deterministic rather than clever:

- **The routing table decides; the model only executes.** A row is never "interpreted". Same table
  in, same seat out — this is the same discipline we use for agent design (agents pick procedures
  from a fixed catalogue and do not invent behaviour).
- **A script checks before an agent wakes.** The timer runs a cheap deterministic pass; an agent
  starts only when there is real work. This is also the top cost lever in the loop-engineering
  material: *choosing the right trigger drives the cost down more than any prompt tuning.*

## 4. Task types are contracts, not labels

Each type is a card declaring **who runs it, what it produces, and what must be true before it
counts as done**:

| Type | Means | Seat | Deliverable | Gates (binary) | Default caps |
|---|---|---|---|---|---|
| `research` | answer a question from sources | Researcher | `research/<date>_<genre>.md` + `sources/` | every claim cited; quotes verbatim; sources on disk | 45 min · 250k tokens |
| `build` | change code in a repo | Engineer | commit on a branch + PR + raw test output | test command exits 0; changed paths ⊆ stated scope | 45 min · 250k tokens |
| `review` | judge a diff or artifact | Engineer | review note on the issue | every finding cites file + line; no unqualified verdicts | 30 min · 120k tokens |
| `ops` | run or maintain something on the machine | Engineer | unit/file changed + before/after evidence | service `active`; health check passes | 30 min · 120k tokens |
| `data` | extract or convert a dataset | Engineer | file at the stated path | row count matches source; schema matches | 30 min · 120k tokens |

Caps are per row and are enforced by the runner, not requested in the prompt.

## 5. Seats: scoped by toolsets, never by skills

One agent runtime, several seats, each with a different privilege level:

- **Coordinator** (Jarvis) — full machine: explores, plans, delegates, surfaces results.
- **Researcher** — read-heavy: web scraping, document analysis, knowledge writes.
- **Engineer** — code and local services, with write access to repositories.
- **Specialist seats** — narrow, e.g. scientific literature or client-facing writing.

The scoping rule that matters: **capability is scoped with toolsets; skills are global.** Skills
are knowledge (how to do something); toolsets are permission (what you may touch). Scoping an agent
by removing skills gives you an ignorant agent; scoping it by removing toolsets gives you a
contained one. Concretely: workers get no SSH keys, no secrets file, no browser profile — the
credential surface is the privilege surface.

## 6. Caps and budgets (as configured, 2026-09)

Every number below is live policy, not aspiration:

| Control | Value |
|---|---|
| Per-run wall clock | 45 min (mechanical types: 30) |
| Per-run tokens | 250k (mechanical types: 120k) |
| Loop iterations before escalation | 3, then a human |
| Concurrent workers | 3 |
| Auto-pause | after 3 consecutive dispatch failures |
| Per-seat monthly ceiling | coordinator $10 · research $5 · engineering $5 · specialists $2 + $2 |
| Company monthly ceiling | $25, warn at 80%, **hard stop enabled** |
| Merge | agent proposes, **human merges** |

The honest caveat, stated because a budget policy that cannot fire is a comfort blanket: per-run
**spend reporting into the meter is not wired yet.** The ceilings exist; the measurement of
`billed_cents` is still zero. Until that is fixed, the caps that actually bind are wall-clock,
iterations and concurrency.

## 7. Non-negotiables

- **Deterministic dispatch.** The routing table decides; nothing is interpreted.
- **Gates are exit codes, not sentences.** A row that cannot be verified is not done.
- **One writer per layer.** Humans → sheet left block. Factory → sheet right block. Paperclip → state.
- **Fail visibly.** Every error lands in `factory_note` and on the issue. Nothing is swallowed.
- **Caps exist.** A run that goes sideways dies early and says why.
- **Workers never inherit the operator's identity.**
- **Instrument the factory, not just the product.** Runs, cost, review time, rework rate.

## 8. What exists vs what is a draft

| Component | Status |
|---|---|
| Sheet read path (stdlib `.xlsx`), round-tripped and verified | **built** |
| Timer pattern (`systemd --user`, `Persistent=true`) | **built** |
| Task state, seats, assignments, budget policies | **built** |
| Brief standard (intent verbatim, sourced facts, absolute deliverable paths, scope locks, stop conditions, done-when, evidence contract) | **built** |
| Agent skill/toolset scoping | **built** |
| Dispatcher (validate → route → create → write back) | draft |
| Type cards as machine-readable config | draft |
| Per-run ledger row + spend reporting | draft |
| Nightly reflection loop (mining runs into skill proposals) | draft |
| Isolated workspace per run | designed, needs one privileged setup step |

The rule we hold ourselves to: **"done" means the artifact exists and was exercised**, not that a
plan exists. Where something is a draft, this table says so.
