# 03 — The three hard problems

Everything else in a factory is plumbing. These three are the ones that decide whether the thing
works, and they are the ones the sources argue about. Each section: what the problem is, why it is
hard, what we do, and what we refuse to do.

---

## Problem 1 — Verification doesn't scale with generation

### What it is

Writing code became cheap. Deciding whether the code is any good did not. Every factory therefore
converts into a **review-and-trust** problem: what do you accept without reading it, and what
evidence do you demand first?

The strongest version of this argument comes from Dex Horthy, whose talk is a deliberate
counter-argument to the whole genre. His claim is not "agents write bad code" — it is that the
system cannot *learn* not to:

> "There's no way in this system that we can penalize it for poor program design."

His reasoning: the reward signal is binary (tests pass / fail), the model's own test edits are
reverted, and the cost of bad architecture is measured in months — far too long a horizon for the
loop to feel it. So a factory can converge on code that passes every gate and is still expensive to
maintain. He concedes he cannot prove it. We think he is right about the mechanism and that it is
the reason gates are necessary but not sufficient.

### Why it is hard for us specifically

- An agent's "done" is a **claim**, not a fact. In our own work this session, a worker reported
  "0 remaining" on a task where our independent audit found 49 remaining items — the worker's
  scoping rule was reasonable and its summary was still wrong in a way that mattered.
- Review volume grows with generation volume, and review is the one task we cannot delegate to
  something with no skin in the game.
- Prompt-level guardrails do not hold. The security talks in the corpus are unambiguous: a
  determined agent works around instructions, so policy must be enforced by machinery.

### How we deal with it

| Mechanism | Concretely |
|---|---|
| **Gates as exit codes** | The runner executes the test/schema/scope command and records its exit code. A worker cannot skip it or narrate past it. |
| **Binary definitions of done** | Every brief carries `done_when` as binary checks, written *before* the work starts. If it cannot be checked, it is not done. |
| **Verification travels with the artifact** | Raw output, not a summary of the output. |
| **Rework carries context** | A failed gate re-dispatches with the verdict and artifacts attached. Never a blind retry — and rework is capped. |
| **Adversarial plan review** | A cheap pass attacks the plan and its task graph *before* any worker runs. Plan defects are the expensive ones. |
| **Independent verification of claims** | Anything a worker asserts is re-derived by the coordinator with a different method (script, diff, re-parse) before it is reported upward. |
| **Human merges** | The agent proposes; the human merges. The last irreversible step stays human. |

### What we refuse to do

- **Trust a self-report.** "Tested and verified" is a claim. Re-run it or re-derive it.
- **Auto-merge on a reviewer's opinion.** One school in the corpus merges low-risk PRs on verifier
  evidence alone; the other school is exactly the experiment that rejects it. We took the
  conservative branch.
- **Use prompts as guardrails.** Instructions are not enforcement.
- **Gate on approval-per-run.** If a human must approve every run, they will approve every run —
  *"it may ask you for 500 times confirmation and then you just click yes."* Gates must be exit
  codes, not clicks.
- **Measure output instead of review time.** Volume is the easy number and the misleading one.

---

## Problem 2 — Cost, and loops that don't stop

### What it is

An unbounded agent loop is an unbounded bill and an unbounded blast radius. The failure is quiet:
nothing errors, the run simply keeps going, and by the time anyone looks the meter is the only
evidence.

Two facts make this sharper than it looks:

- **A budget of `$0` means "unmetered", not "free".** Our own task-state system reported
  `spendCents: 0` for every seat because nothing reports spend — a policy that looks like a
  constraint and constrains nothing.
- **Small loops are where the money goes.** Not the big run, but the loop that retries, re-reads
  and re-reasons 40 times because no one told it when to stop.

### How we deal with it

- **Per-run caps**: wall-clock (45 min; 30 for mechanical work), tokens (250k / 120k), and a hard
  iteration ceiling of **3** — then a human. Loops escalate, they do not spin.
- **Per-month budget policies** per seat *and* for the whole company, with a warning threshold and
  a **hard stop** enabled.
- **Concurrency cap** — bounded parallelism, so one bad batch cannot saturate the machine.
- **Auto-pause after 3 consecutive failures** — a broken integration stops itself instead of
  burning budget on every tick.
- **A ledger row per run** — duration, tokens, tool calls, model, pass/fail — so cost is a
  per-run property, not a monthly mystery.
- **Deterministic pre-checks before waking an agent.** The cheapest agent call is the one you
  never make; a script that checks for real work first is worth more than any prompt tuning.

### What we refuse to do

- **Ship a loop without a ceiling.** If it can run forever, it will.
- **Treat an unset budget as a guarantee.** An unfed meter is not a control. Say "unmetered",
  not "free".
- **Optimise token consumption.** Consumption leaderboards celebrate the metric that
  anti-correlates with value.
- **Confuse a cap with a policy.** A token cap protects the bill; it does not make the work
  correct.

---

## Problem 3 — Blast radius: identity, isolation and the machine underneath

### What it is

An agent that runs with the operator's identity *is* the operator. Everything it can reach — SSH
keys, cloud tokens, the browser profile, the home directory — is in scope for whatever the agent
decides to do, including being talked into it by content it reads.

The corpus splits on how to solve this, and the split is instructive:

- **Boundary**: a trust boundary, not containers — *"we do not use containers, I mean these are all
  trusted systems"* — with the rule *"we never execute raw code off the prompt"*.
- **Security talks (Falco, nono)**: prompt-level guardrails fail; enforce and attest **below** the
  agent, at the runtime/kernel layer.
- **Monzo (regulated bank)**: *"there's nothing in regulation that stops you from"* doing the work —
  the constraint is control and auditability, not prohibition.

They agree on the one thing that matters: **stop trying to solve this inside the prompt.**

### How we deal with it

- **One identity per layer.** *"Identity provisioning all goes into the dev environment, not the
  harness"* — the agent's configuration should not carry the credentials that grant its power.
- **Least privilege per seat, by toolset**: a research seat has no repository write access; a
  worker has no secrets file, no SSH key, no browser state.
- **One isolated workspace per run**: a per-run worktree of the target repo, so concurrent runs
  cannot tread on each other's files, plus a transient systemd unit so a run cannot outlive its
  own budget.
- **Enforcement below the agent** where the platform allows it (kernel sandboxing, filesystem
  scoping) — because instructions are advice and mechanisms are constraints.
- **Know your ceiling.** If the factory runs on a laptop, it stops when the laptop sleeps:
  *"you can't create reliable automations around machines that might be asleep or turned off."*
  We accept it, use `Persistent=true` scheduling so a missed run fires after wake, and do not
  pretend it is a datacentre.

### What we refuse to do

- **Run workers on the daily driver with the operator's credentials.** This is the single
  highest-leverage rule and the one most people skip.
- **Assume a prompt is a boundary.** It is documentation for the willing.
- **Blanket-ban capabilities.** Prohibition is not control; scoping and auditability are.
- **Container theatre.** Isolating trusted code in a container costs throughput and buys a feeling.
  Isolate the runs that touch untrusted input or irreversible actions, and enforce it where the
  kernel enforces things.
- **Keep secrets in the agent's context** and hope they are not used.

---

## The consolidated "don't" list

Collected from the corpus, each one something a named speaker paid for:

| Don't | Why |
|---|---|
| Don't over-skill | *"If you just slap skills on everything it can lead to unpredictable"* behaviour — skills are contracts, not stickers |
| Don't use approvals as the safety net | fatigue becomes the bypass |
| Don't swallow errors | *"any error that you encounter you want to make sure is highlighted"*, not smoothed over |
| Don't blanket-ban capabilities | regulation rarely requires it; control and audit do |
| Don't optimise a metric instead of taste | *"it improves the contrast ratio, but now"* the design is worse — the metric moved and the product did not |
| Don't scale the automation count | ~14 broad automations maintaining the codebase beat dozens of narrow ones |
| Don't keep two sources of truth | the markdown copy goes stale and late decisions get made from it |
| Don't run agents with your own identity | see Problem 3 |
| Don't automate the irreversible | merges, deploys, deletions, payments — human, or gated on an artifact a human approved |

Next: **[04 — What we learned](04-what-we-learned.md)**
