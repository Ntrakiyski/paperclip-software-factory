# 01 — What is an AI software factory?

## The definition

A **software factory** is an automation around the core loop of software development:

```
triage → spec → implement → review → verify → ship → monitor
```

At each step, a mix of agents and humans moves the work forward. The word *factory* is doing real
work here: it implies a fixed process, repeatable inputs, measured output and — most importantly —
that the process, not the individual craftsman, is what you improve.

An **AI** software factory is the same loop where agents own the steps that used to be human.
The distinction that matters is not "AI writes code". It is:

| | Interactive agent | Factory |
|---|---|---|
| Who starts work | a human, per task | the system, from a queue |
| Where it runs | your terminal, your session | in the background, isolated |
| How it stops | you stop it | a gate or a cap stops it |
| What it produces | a diff you review | a *mergeable* artifact with evidence |
| What you measure | nothing, usually | runs, cost, review time, rework |

You can have a factory with a mediocre model and a great harness, and get useful output. The
reverse is not true — that is the single most repeated claim across every source in
[docs/05-resources.md](05-resources.md), and it is the reason this repo is about harness design
rather than about prompts.

## What the loop looks like when it works

The published accounts converge on the same shape:

- **Spec before code.** A brief or PRD is turned into a plan, and the plan is turned into
  sequential, independently-verifiable units of work. Ona's build-in-public sprint used a *Feature
  Planner* automation to break a spec into GitHub issues before any builder ran; their own
  retrospective conclusion was blunt: *spec quality determines output quality*.
- **Build in a loop with a hard ceiling.** A worker implements one unit, runs the tests, and
  escalates rather than looping forever. Every factory we studied caps its loop (see §Caps).
- **Verify with evidence, not adjectives.** Tests, type checks, linters, a diff-scoped review,
  sometimes a second agent whose only job is to attack the first one's output. The verification
  must be runnable by the harness, so the worker cannot fake it.
- **Ship behind a human.** Across all 34 talks, the merge button is the last thing to be
  automated — and in the more cautious accounts it stays human on purpose.
- **Feed what you learn back.** Failures become gates; gates become automation; the count of
  automations grows. Ona's 10-day sprint went from "no repo" to **16 automations**; the ones
  maintaining the codebase outnumbered the ones writing it by the end.

## Evidence: who reports what

These are the public numbers, quoted from the talks and pages listed in `05-resources.md`.
They are self-reported. Use them for direction, never as a benchmark for your own shop.

| Organisation | Claim | Source |
|---|---|---|
| Ona / software-factory.dev | 2-week sprint, **375 PRs, 50k+ lines, 16 automations, 1,067 tests, 87% autonomous, median PR→merge 4.9 min**, zero human-written code | build-in-public streams + landing page |
| Stripe | Agents working a **~30-million-line Ruby codebase**, turning Slack/Jira requests into reviewed PRs ("minions") | Background Agents Summit — Alistair Gray |
| Uber | An "agent-ready developer platform" that turns platform toil into background agent work | Summit — Nikhil Ramakrishnan |
| Cloudflare | Moved from **assisted coding to delegated work** with identity, context and review as the enabling layers | Summit — Rajesh Bhatia |
| Spotify | **Agent fleets** coordinating large-scale legacy migrations | Spotify engineering blog |
| StrongDM | A "dark software factory", with a stated token budget of **~$1,000/day per human engineer** | factory.strongdm.ai + Summit — Shardul Vaidya |
| Warp | **20–30% of issues fully automatable today**; most of the rest partially | Warp factory guide |
| Monzo | A working playbook for adopting AI tools inside a **regulated bank** without losing control | Summit — Suhail Patel |

## The honest limits

Four things temper the enthusiasm, and they come from inside the corpus rather than from critics:

1. **The numbers are vendor-reported and the corpus is not neutral.** A vendor-hosted summit plus
   one company's build-in-public sprint. One talk (Dex Horthy) says openly:
   *"every company and their mother is talking about how they built a coding agent factory that
   ships 75% of their code"* — and argues the interesting question is what those factories are
   *not* measuring.
2. **Autonomy percentage hides the shape of the work.** 87% of PRs merged autonomously on a
   greenfield note-taking app is not the same claim as 87% on a ten-year-old codebase with paying
   customers. Greenfield, self-contained, test-covered work automates first.
3. **Verification does not scale with generation.** The most sophisticated organisations in the
   corpus employ humans on judgement, and use gates — not prompts — for the rest. Where they
   differ is how much they trust the automated reviewer, and that disagreement is real (see
   `03` and `04`).
4. **The infrastructure is not the hard part.** Three of the smallest factories in the corpus run
   on a laptop, a cron entry and a SQLite file. The difficulty is in the specification, the gates
   and the discipline — not in buying a platform.

## What this means for a one-person shop

The factory framing scales *down* better than it scales up, because the bottleneck it addresses —
human attention — is exactly what a single operator does not have:

- You are the spec author and the merger. Everything else can be delegated, including the review.
- Your queue is the thing to instrument: how many items are waiting on a human, and for how long.
- The floor is low: a queue, a runner, a gate script and a timer is a functional factory. The
  smallest examples in the corpus are [one skill plus a Python state machine](05-resources.md).
- The ceiling is your machine's uptime. If your agents run on a laptop, the factory stops when
  the laptop sleeps — plan for it rather than pretending otherwise.

Next: **[02 — How we build it with Jarvis](02-how-we-build-it-with-jarvis.md)**
