# 05 — Resources and inspiration

Everything we read, in the order it is most useful. All links point to the original; we do not
republish transcripts or slides.

## The 34 talks we transcribed and read (~176,000 words)

Machine-readable index of all 34 — series, title slug, URL, transcript size in bytes and words:
[`resources/talks.csv`](../resources/talks.csv). Totals: **175,575 words across 895 KB** of transcript
(sprint 60,658 · summit 77,120 · loops 37,797).

### Build-in-public: a software factory shipped in two weeks (10 streams)

The reference production for anyone starting: a note-taking app built in ten working days with
**zero human-written code**, streamed daily and everything left public. Day 1 starts with an empty
repository.

| # | Stream | Link |
|---|---|---|
| 01 | Day 1 — zero code, zero repo | <https://www.youtube.com/watch?v=4Al_EIVkc6U> |
| 02 | Day 2 — setting up the assembly line | <https://www.youtube.com/watch?v=7DtaqZ0dg-k> |
| 03 | Day 3 — from spec to issues, with the CTO | <https://www.youtube.com/watch?v=yL3XEEF3Llw> |
| 04 | Day 4 — how agents review each other's code | <https://www.youtube.com/watch?v=ELS-DvDT3Yg> |
| 05 | Day 5 — what the factory catches and what it misses | <https://www.youtube.com/watch?v=SgBITxT-LhM> |
| 06 | Day 6 — giving the factory taste | <https://www.youtube.com/watch?v=7t_52lKysFg> |
| 07 | Day 7 — augmenting the product manager | <https://www.youtube.com/watch?v=LU8Mo4z4fBI> |
| 08 | Day 8 — when shipping is free, what do you cut? | <https://www.youtube.com/watch?v=-1_NTT9p_0g> |
| 09 | Day 9 — platform or ground up? | <https://www.youtube.com/watch?v=3zlFEva26Eo> |
| 10 | Day 10 — how we went from 0 to factory (recap) | <https://www.youtube.com/watch?v=00Ndri8q8LU> |

Repo: <https://github.com/gitpod-io/memo> · Live app: <https://memo.software-factory.dev> ·
Landing page: <https://www.software-factory.dev/> · Channel: <https://www.youtube.com/@ona_hq>

### Background Agents Summit (17 sessions)

The state of the art from teams running factories in production, including the disagreements in
[04](04-what-we-learned.md). Summit index: <https://background-agents.com/summit/>

| # | Session | Link |
|---|---|---|
| 01 | What comes after AI coding assistants (opening) | <https://www.youtube.com/watch?v=1VZPX7QD2tk> |
| 02 | Stripe — Building Minions: agents on a 30-million-line codebase | <https://www.youtube.com/watch?v=W42t-CoXyuE> |
| 03 | Cloudflare — from assisted to delegated: the AI engineering stack | <https://www.youtube.com/watch?v=MbLdrAZFQRs> |
| 04 | Agent runtime security — enforce, attest, decide | <https://www.youtube.com/watch?v=pg0t9jf5DY4> |
| 05 | Building a company-internal background agent system | <https://www.youtube.com/watch?v=-T_Qc1Vmbtk> |
| 06 | Background agents for genomics: science and cloud ops | <https://www.youtube.com/watch?v=GXDtw0EmoX0> |
| 07 | Why prompt-level guardrails fail, and what works | <https://www.youtube.com/watch?v=XKFKwyFhk8A> |
| 08 | Context is the new code | <https://www.youtube.com/watch?v=Pz-vVV0Jmfc> |
| 09 | Summit day 1 recap | <https://www.youtube.com/watch?v=TrT1ysbWDos> |
| 10 | Summit day 2 — foundations to production | <https://www.youtube.com/watch?v=bguL5ZEWbPo> |
| 11 | Harvey — Spectre, a collaborative cloud agent platform | <https://www.youtube.com/watch?v=2pod6GeckXc> |
| 12 | incident.io — AI SRE and collaborating with agents | <https://www.youtube.com/watch?v=uG0Jz6kLWV0> |
| 13 | A software factory built in public | <https://www.youtube.com/watch?v=a4pEznCiWNo> |
| 14 | Uber — backgrounding the toil on an agent-ready platform | <https://www.youtube.com/watch?v=o7G5EHDqQ1g> |
| 15 | Dark factories — a Rust state machine in 72 hours | <https://www.youtube.com/watch?v=e7AvdrxsbaU> |
| 16 | Monzo — enabling AI tools without losing control (regulated bank) | <https://www.youtube.com/watch?v=ptVO89qaxY4> |
| 17 | Summit wrap — what to build next | <https://www.youtube.com/watch?v=yuVFSvJf8tk> |

### Loop engineering and the counter-argument (7 talks)

Smaller, more opinionated, and — for a single operator — more directly copyable. Two of these
disagree with each other, which is why both are here.

| # | Talk | Link |
|---|---|---|
| 01 | I was building loops wrong (AI Jason) | <https://www.youtube.com/watch?v=JQ_We_ztxrI> |
| 02 | I unleashed my agent (AI Jason) | <https://www.youtube.com/watch?v=2zhchG0r6iI> |
| 03 | Proactive agents and the self-improving company | <https://www.youtube.com/watch?v=ikH1--DSzMs> |
| 04 | I built a self-improving AI software factory | <https://www.youtube.com/watch?v=ZDOTYfJBuLw> |
| 05 | **How to build a software factory for AI coding agents** (Boundary, 71 min) — the densest single source in the corpus | <https://www.youtube.com/watch?v=tGbjIvvYuHE> |
| 06 | **Harness engineering is not enough** (Dex Horthy) — the counter-argument | <https://www.youtube.com/watch?v=Ib5GBkD555M> |
| 07 | My super simple software factory | <https://www.youtube.com/watch?v=haUfb1ievTE> |

We fetched these transcripts with a browser-automated notebook pipeline rather than caption
scraping; the fetching method and its two sharp edges (output truncation, auth state) are worth
knowing before you build your own corpus.

## Articles, guides and tools

| Source | Why it is worth reading |
|---|---|
| [OpenAI — harness engineering](https://openai.com/index/harness-engineering/) | Names the discipline: encoding engineering practice so agents can drive development |
| [StrongDM — dark software factory](https://factory.strongdm.ai/) | The most aggressive public position, including the token-budget-per-engineer framing |
| [Warp — a guide to cloud software factories](https://www.warp.dev/blog/a-guide-to-cloud-software-factories-for-engineering-leaders) | The best single overview of the SDLC loop and where humans stay; also the 20–30% automation estimate |
| [Warp — the factory stack](https://www.warp.dev/blog/the-factory-stack) | Reference stack, and "factories as code" — declaring the factory itself in a config file |
| [Spotify engineering — background coding agents](https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1) | Agent fleets for large-scale legacy migration |
| [Cole Murray — background agents](https://github.com/ColeMurray/background-agents) | A real, readable implementation: setup/start scripts, boot modes, model-provider notes |
| [Ona — background agents guide](https://ona.com/guides/background-agents) | The summit's written companion, with the patterns pulled out of the talks |
| [The complete background agents guide](https://background-agents.com/) | Session-by-session index of the summit |
| [Open Inbox / openinspect.dev](https://openinspect.dev/) | An inspectable background-agent product; useful as a UI reference |
| [gitpod-io/memo](https://github.com/gitpod-io/memo) | The artefact of the two-week sprint — read it to see what an agent-written codebase looks like |
| [Falco (CNCF)](https://falco.org/) | Runtime security; the enforcement-below-the-agent position in practice |
| [Paperclip](https://github.com/paperclipai/paperclip) | The task-state layer used in this repo's design: issues, seats, runs, budgets |
| [Hermes Agent](https://hermes-agent.nousresearch.com/docs) | The agent runtime used here: skills, toolsets, delegation, scheduling |

## How we used this material

1. **Transcribe, then read.** Transcripts indexed by series with titles, URLs and word counts.
2. **Extract with quotes, then verify.** Every extracted claim carries a verbatim quote; every quote
   is machine-checked against the transcript (`grep -F` on the first 50 characters, or an exact
   substring assertion in Python). Quotes that failed verification were fixed or dropped — including
   cases where the transcription had silently normalised a spoken "uh".
3. **Two independent passes.** A second pass re-verified all 150 extracted quotes, added a
   source-strength column per item, and corrected four quotations. Result: **2 of 20 takeaways
   cleared a three-independent-source bar.**
4. **Label the tail honestly.** Single-sourced items stay in the document, marked, rather than
   being deleted or promoted.

## Notes on reuse

Quotations in these docs are short excerpts used for commentary and criticism, attributed to the
speaker and linked to the original. Full transcripts, slides and video are **not** redistributed
here. Where a licence matters (Sentry's FSL-1.1-Apache-2.0, the Model Context Protocol's Apache-2.0),
we checked the licence text itself rather than a secondary summary, and cite what we read.
