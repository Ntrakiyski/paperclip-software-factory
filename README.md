# Paperclip Software Factory

A working notebook on **AI software factories** — what they are, what the teams running them at
scale actually do, and how one operator is building the same thing on a single laptop with an
agent stack: **Hermes/Jarvis** for the agent runtime, **Paperclip** for task state.

This is the distilled output of a deliberate research phase, not a pitch:

- **34 talks and build-in-public streams** transcribed and read (~176,000 words) — Stripe, Uber,
  Cloudflare, Harvey, incident.io, Monzo, Genentech, StrongDM, Boundary, Ona, plus the
  loop-engineering crowd.
- **~40 primary sources** — vendor engineering blogs, licences, docs — each fetched and cited.
- **Two adversarial verification passes.** Quotes are machine-verified against the transcripts;
  claims that could not be traced to a source are labelled as opinion. Where the sources disagree,
  both sides are printed.

Everything here is either traced to a source or explicitly marked as our own judgement.

## What is an AI software factory?

A factory is an **automation around the core loop of software development**:

```
triage → spec → implement → review → verify → ship → monitor
```

At every step a mix of agents and humans moves the process forward. The difference from
"using an AI coding assistant" is who is driving: an assistant runs one turn inside a human's
session, a factory runs work to completion on its own, in the background, and reports back.

Three things are true at once, and the sources are unusually consistent about them:

1. **It is automation around the SDLC, not a better model.** The wins come from harness design —
   gates, environments, isolation, context. "Harness engineering" is OpenAI's term for encoding
   engineering practice so agents can drive development; StrongDM operationalises it as a
   ["dark software factory"](https://factory.strongdm.ai/); Spotify deployed
   [agent fleets](https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1)
   for large-scale migrations.
2. **It works today for a subset of work, and the subset grows.** Warp's estimate: roughly
   **20–30% of issues are fully automatable today**; most of the rest are partially automatable.
   You do not need 100% — the value is that some work completes without a human in the loop.
3. **The bottleneck is verification and attention, not generation.** Every serious source says
   the same thing in different words: code is now cheap, so review, trust and taste become the
   scarce resources. Factories are designed to protect *those*, not to produce more text.

The most quoted line from the build-in-public sprint that produced this repo:

> "You don't build Memo, you build a factory to build Memo."

Two weeks, zero human-written code, **375 PRs, 50k+ lines, 16 automations, 87% autonomous,
median PR-to-merge 4.9 minutes** — [software-factory.dev](https://www.software-factory.dev/).
Those numbers are the vendor's own; treat them as a direction, not a benchmark.

## How we do it

One operator, one laptop, one agent that runs the machine. Three state layers, **one writer each**:

| Layer | Holds | Owned by |
|---|---|---|
| **Identity + hands** | the agent itself: skills, memory, OS control, delegation | Hermes (`Jarvis`) |
| **Task state** | issues, assignments, runs, budgets — what is happening and what it cost | Paperclip |
| **Knowledge** | durable, cross-session knowledge with provenance | Shared Living Memory |

The human surface is deliberately tiny and boring: **one spreadsheet**. You write a row of intent;
a timer reads the sheet, validates it, routes it to a seat by task type, dispatches it with gates
and caps attached, and writes status, artifact, cost and reason back into the same row. The
sheet's left block is yours; the right block is generated and never typed by hand.

Full detail: **[docs/02-how-we-build-it-with-jarvis.md](docs/02-how-we-build-it-with-jarvis.md)**

## The three hard problems

| Problem | How we deal with it | What we refuse to do |
|---|---|---|
| **Verification doesn't scale** — generation got cheap, review and trust did not; an agent's "done" is a claim, not a fact | gates are **exit codes** the worker cannot skip; every brief carries binary `done_when`; verification evidence travels with the artifact; rework carries failure context, never a blind retry; **a human merges** | trust self-reports; auto-merge on a reviewer's opinion; use prompt guardrails as a safety net; use approval-per-run as a gate (fatigue becomes the bypass) |
| **Cost and runaway loops** — an unbounded loop is an unbounded bill, and a budget of `$0` means "unmetered", not "free" | hard caps per run (wall-clock, tokens, iterations) and per month (per seat *and* company, with hard stop); concurrency capped; auto-pause after 3 consecutive failures; every run leaves a ledger row | ship a loop with no ceiling; treat an unset budget as a guarantee; optimise a token-consumption leaderboard |
| **Blast radius** — the prompt is not a security boundary, and a worker inheriting the operator's identity can do everything the operator can | least privilege per seat (scoped toolsets), workers get no SSH keys, no `.env`, no browser state; one isolated workspace per run; enforcement *below* the agent (kernel sandboxing) rather than inside it | assume a prompt can contain a determined agent; hand-code container theatre for trusted code; run agents on a machine that sleeps and call it reliable |

Full reasoning, with sources: **[docs/03-three-hard-problems.md](docs/03-three-hard-problems.md)**

## Repo map

| Path | Contents |
|---|---|
| [docs/01-what-is-a-software-factory.md](docs/01-what-is-a-software-factory.md) | The concept, the loop, the evidence, and its limits |
| [docs/02-how-we-build-it-with-jarvis.md](docs/02-how-we-build-it-with-jarvis.md) | Our architecture: layers, seats, the sheet contract, routing, gates, caps |
| [docs/03-three-hard-problems.md](docs/03-three-hard-problems.md) | The three problems, and the anti-patterns we refuse |
| [docs/04-what-we-learned.md](docs/04-what-we-learned.md) | Distilled lessons with short quotes, including the strongest counter-argument |
| [docs/05-resources.md](docs/05-resources.md) | All 34 talks with links, plus articles, tools and further reading |

## Reading order

1. `01` for the concept and the evidence.
2. `03` if you only read one thing — it is where the opinions are.
3. `02` for the concrete design (a single-operator factory, `systemd` timers, one spreadsheet).
4. `04` and `05` to go deeper into the sources.

## Honest status

This repo documents a factory that is **mapped and partially built**, not finished. We say so in
the docs because the sources that pretend otherwise are the least useful ones.

- **Built and verified:** the sheet read path, the timer pattern, task state on Paperclip,
  per-seat and company budget policies, the brief standard, agent skill/toolset scoping.
- **Designed, not built:** the dispatcher that turns a row into a gated issue, the type cards,
  the per-run ledger, the nightly reflection loop.
- **Known gaps we have not fixed:** per-run isolation needs one privileged step; nothing yet
  reports per-run spend into the budget meter, so the hard stop is real but unfed; the operator's
  agent still shares the operator's account.

## Licence and provenance

No licence is granted for this repository's text yet — it is a working notebook. Quoted material
belongs to its speakers; links point to the originals, and transcripts are **not** republished
here (only indexes and short quotes). See [docs/05-resources.md](docs/05-resources.md).
