# 04 — What we learned, and what we changed

Method note first, because it sets the confidence level: **34 talks and streams (~176,000 words of
transcript) plus ~40 fetched primary sources**, read in full, with every quotation machine-verified
against its transcript before being quoted here. Two independent verification passes were run over
the extracted material. Of the twenty transferable takeaways we initially wrote down, **two cleared
a three-independent-source bar; eighteen remained single-sourced.** Everything below is labelled
accordingly — the ones marked *repeated* appear independently in multiple talks, the ones marked
*single* rest on one account.

We kept the quality label in the document instead of deleting it, because the honest version of a
research artefact is more useful than a confident one.

## The lessons that survived scrutiny

**1. Harness over model — every miss becomes a permanent artefact.** *(repeated)* The differentiator
between a demo and a factory is what happens after a failure: a gate, a script, or a skill — never
"try again with a better prompt". The same failure twice is a process defect, not a model defect.

**2. Human attention is the binding constraint, not generation.** *(repeated)* The scarce resource
is the reviewer's hour. Instrument the queue of work waiting on a human — depth, and time-in-queue —
and optimise that number. Nothing else you measure correlates as well with real throughput.

**3. Gates are back-pressure; prompts are not guardrails.** *(repeated)* The most consistent finding
across Stripe, Uber, Cloudflare, Boundary and the security talks: verification must be executable
by the harness. *"We never execute raw code off the prompt"*, and *"you actually have to give us
metrics for every single version of this pipeline"* — metrics as a gate, not a dashboard.

**4. Spec quality is upstream of everything.** *(repeated)* Ona's own retrospective after shipping
375 PRs put it as: spec quality determines output quality. The factories that work spend a
disproportionate amount of effort before the first line of code — Dex Horthy's number is that
**30 minutes in pre-planning saves hours in review**, and his framing is the useful one: *"you don't
have too many PRs, you have too many bad PRs."*

**5. A non-reproducible environment makes a non-reproducible agent.** *(repeated)* Environment drift
was named the "single biggest friction point" in the corpus, and it matched our own experience —
interpreter and dependency chaos, not model quality, is what breaks runs.

**6. The harness is files in the repo; the instruction file stays short and indexes them.**
*(repeated)* Long monolithic agent instructions rot. Project-level instruction files should link to
versioned documents, not contain them.

**7. Cap the loop and escalate.** *(repeated)* *"Max iterations, like three"* — then a human. The
count is less important than the existence of the ceiling and the fact that hitting it produces an
escalation rather than a retry.

**8. Adversarial review of the plan, before the work.** *(single)* Cheap, and it catches the
expensive class of error. A plan with a bad task graph produces confident, well-tested work on the
wrong thing.

**9. Rework carries failure context forward.** *(repeated)* Re-dispatch with the gate's verdict and
the artefacts attached, and cap the rework count. A blind retry is a coin flip charged at full price.

**10. Agents are stateless; crashing is a feature.** *(repeated)* Run state lives outside the worker.
If the process dies, a new one resumes from state — this is what makes long work survivable.

**11. One isolated workspace per run.** *(repeated)* A per-run checkout (worktree or clone), plus a
sandbox for anything touching untrusted input. Cheap; removes a whole class of interference.

**12. Keep coding agents off the daily driver.** *(repeated)* And note the contradiction we lived
with: the operator's own agent *does* hold OS control, by design. The rule applies to workers, not
to the assistant you talk to.

**13. Instrument the factory, not just the product.** *(repeated)* Runs, cost, review time, rework
rate, autonomous share. Without this you cannot tell whether a change made the factory better — and
per source 3, metrics are a *gate* requirement for each pipeline version.

**14. Enforce below the agent.** *(repeated)* Isolation, observability and control at the runtime
layer: filesystem scoping, sandboxing, attestation. The prompt is not the boundary.

**15. The floor is much lower than the genre suggests.** *(repeated)* Three of the smallest factories
in the corpus are: a memory file plus skills plus a cron; one skill plus a Python state machine plus
YAML gates (largest workflow: 180 lines); a queue, a worker, a coordinator prompt and a SQLite run
store on one small VM. None of them needed infrastructure to start.

**16. Deterministic work belongs in code, not in the agent.** *(repeated)* Validation, routing,
scoping, counting, diffing: scripts. The agent's job is the part that needs judgement. Notably, the
more mature a loop got in the corpus, the *more* of its logic had moved out of the model.

## The counter-position, steelmanned

The sharpest talk in the corpus argues the opposite of its neighbours: **harness engineering is not
enough**. Its claims, fairly stated:

- The reward signal cannot penalise poor program design, so long-run maintainability is invisible to
  the loop.
- Cost of architectural mistakes is measured in months or years — longer than any feedback loop.
- A judge model's approval is an opinion, and opinions drift.
- Teams that do not own their model's weights are structurally disadvantaged on this axis.

What survives scrutiny: **factories produce mergeable code faster than humans produce reviewed
code**, so the review step must stay real, and the failure mode of a factory is not broken code but
*plausible code that is cheap to merge and expensive to own*. He also concedes he cannot prove it,
and the corpus's own numbers are unaudited vendor claims. So the honest position is:

> Build the factory. Keep the human at the merge button. Read the diff often enough to keep your
> taste calibrated, because taste cannot be delegated even when review can.

## Where the corpus disagrees with itself

We did not smooth these over; they are the most interesting part of the material.

| Disagreement | Positions | Our call |
|---|---|---|
| Can an automated reviewer merge? | One school merges low-risk PRs on verifier evidence alone. Dex Horthy's account is that experiment, and he rejects it. | Human merges. Reversible, cheap to adopt later; unbounded trust is not. |
| Containers or trust boundaries? | Boundary: no containers, trusted systems, trust boundary instead. Security talks: isolate and enforce at the kernel. | Both, by risk: trust boundaries for trusted code, kernel sandboxing for untrusted input and irreversible actions. |
| Laptop or cloud? | Cloud vendors: "you can't create reliable automations around machines that might be asleep or turned off." Small-factory talks: laptop, cron, SQLite. | Laptop, honestly labelled: `Persistent=true` timers, and no claims of five-nines. |
| How many automations? | Ona's sprint ended at 16 automations and concluded the *operations* side needed more than the build side. One small-factory talk warns against scaling automation count. | Start with two or three broad ones; add only when a failure proves the need. |

## What we changed in our own design because of this

Concrete edits, so this is not just reading:

1. **Type cards gained a loop contract**: goal, boundaries, SOP, state and an append-only log per
   task type — the shape used by the most disciplined loop in the corpus.
2. **Every loop got a hard ceiling**: wall-clock, tokens, and 3 iterations before escalation.
3. **Nothing wakes an agent until a script says there is work** — validated as the top cost lever.
4. **Real budget policies** with a hard stop, instead of a `$0` budget that looked like a control.
5. **Merge stays human** — the conservative branch of the material's central disagreement.
6. **A per-run ledger and metrics-per-version** became a requirement rather than a nicety.
7. **Multi-repo work gets a coordination root** with sibling repositories, never submodules.
8. **The worker integration surface** is a headless CLI with a structured transcript, because that
   is what every serious harness in the corpus actually parses.

Next: **[05 — Resources](05-resources.md)**
