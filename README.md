# Build a Council for Your Muse Assistant

*A beginner-friendly guide to giving your personal AI assistant a small panel of other AI models it can consult — so one assistant can get a second, third, and fourth opinion before it answers you.*

> **Guide version:** 1.0 · **Last updated:** 2026-11-11
> **Versions:** the pattern in this guide is provider-agnostic. Record the exact model version strings *you* pin for each seat in the table under [Maintenance](#maintenance) — that table is a template, and keeping it current is part of running a council.

---

## Disclaimer — read this first

- This is an **unofficial, community-written guide**. It is not published, endorsed, or supported by Muse, Meta, or any AI model provider mentioned in generic terms below.
- Everything here is provided **as-is, without warranty**. If you follow it, **you are responsible for your own accounts, your own API keys, and your own API costs.** Multi-model setups spend money on every provider you connect — see [Costs & cost caps](#costs--cost-caps) before you connect anything.
- **Support is best-effort.** If something doesn't work, you're welcome to open an issue, but there is no guaranteed help and no response time. Please give the troubleshooting section a try first.
- **Never post your own secrets in an issue, a comment, or a pull request** — no API keys, passwords, one-time codes, personal details, or screenshots showing your accounts, device names, or home network. Issues containing secrets will be closed and the content should be treated as compromised: revoke any key you posted. See the issue-template note at the end of this guide.
- Check the current terms of service for Muse and for every model provider you connect before you build. Terms change; this guide can't track them for you.

---

## What is a "council"?

A **council** is a setup pattern, not a product:

- You have **one main assistant** — the AI you actually talk to. In this guide the fictional main assistant is named **Nova**.
- Behind Nova sit several **other AI models**, called **seats**. Nova can send them a question, collect their independent answers, and then synthesize a single reply for you.
- **You only ever talk to Nova.** The seats don't chat with you, don't remember you between consults, and don't act on their own. Nova leads: it frames the question, checks the answers, and owns the final reply. **You decide** — a council recommends, it never authorizes anything by itself.

Think of it like a lead advisor with a small panel of consultants behind the curtain.

### Why would you want one?

- **Fewer blind spots.** Different models are trained differently. A weakness or bad habit in one is often visible to the others.
- **Better hard decisions.** On a question with real trade-offs — money, a big purchase, a plan with risks — three independent takes are usually better than one confident take.
- **A built-in fact-check instinct.** When seats disagree, that's a signal to slow down and verify, instead of charging ahead.
- **One seat can be free and private.** A model running on your own computer costs nothing per question and never sends your words to a cloud provider — useful as a first pass on private, everyday material.

### What a council is *not*

- **Agreement is not correctness.** If all the seats say the same wrong thing — and they can, especially if they were all given the same incomplete facts — the answer is still wrong. Consensus raises confidence a little; it never replaces checking.
- **More seats is not automatically better.** Every seat adds cost and reading time. A small council you actually read beats a big one you skim.
- **Seats are not live sources.** A model answering from training is not the same as checking a current price, a schedule, or an account. For anything that changes, the main assistant should gather real evidence first and hand it to the seats (see "evidence packets" below).

---

## Prior art & credit

The general idea of having several models answer and review each other is not new, and this guide doesn't claim it. Andrej Karpathy's open-source **llm-council** project is the best-known earlier take: a research-style web app where models answer a question and anonymously review each other's responses.

This guide covers a different job: a **personal assistant's** council — one assistant you live with, seats it consults privately behind the scenes (including a local, offline seat for private material), per-seat lanes, evidence packets, and hard cost caps — written for beginners setting it up at home. If the research-style version interests you, go look at llm-council too; the two approaches can learn from each other.

---

## The architecture, in generic terms

Everything in this guide uses **fictional names**. Copy the pattern, not the names.

```
                    ┌─────────────────────────────┐
        You ──────► │  Nova (main assistant)      │
                    │  - the only voice you hear  │
                    │  - frames the question      │
                    │  - verifies + synthesizes   │
                    │  - owns the final answer    │
                    └──────┬───────┬───────┬──────┘
                           │       │       │
              ┌────────────▼─┐ ┌───▼─────┐ ┌▼──────────────┐ ┌────────────────┐
              │ Seat 1       │ │ Seat 2  │ │ Seat 3        │ │ Local seat     │
              │ Cloud        │ │ Cloud   │ │ Cloud         │ │ Local model    │
              │ Model A      │ │ Model B │ │ Model C       │ │ on "homebase"  │
              │ (reasoning)  │ │ (fast)  │ │ (diversity)   │ │ (private/free) │
              └──────────────┘ └─────────┘ └───────────────┘ └────────────────┘
```

- **Nova (main assistant):** your Muse assistant. It holds your context and preferences, and it's the only participant that talks to you.
- **homebase:** a fictional always-on home computer that hosts the local model. Any modest computer you leave on can play this role.
- **Seats 1–3 (cloud models A, B, C):** three *different* model families from three different providers. Diversity is the point — three seats from the same family mostly echo each other.
- **Local seat:** an open-weight model running on homebase through a local model runner (tools like Ollama or llama.cpp are common choices). Free per question, works offline, and nothing leaves your house — which is why it gets first pass on private material. An optional second, slower local model can be added later as a "deep pass" for hard reasoning.

### How a consult flows

1. A question comes up that's worth more than one opinion.
2. Nova writes a **consult brief**: the question, the relevant facts, and — for anything factual or current — an **evidence packet** (the actual prices, dates, or documents it found from primary sources, pasted in).
3. Nova sends the **same brief to every seat, independently**. Seats do not see each other's answers. This matters: if they see each other first, they herd toward the first answer and you lose the diversity you paid for.
4. Each seat returns: a recommendation, its key assumptions, and anything it thinks should be verified.
5. Nova **synthesizes**: where the seats agree, where they differ, and what it checked itself. Disagreements get named, not averaged away.
6. **You decide.** Especially on spending, contacting people, publishing, or committing to anything — the council proposes, a human disposes.

---

## Prerequisites

You don't need much, but you do need all of this:

- [ ] A Muse assistant set up and working — this guide adds a pattern *on top of* a working assistant; it is not a Muse setup guide.
- [ ] An account with each cloud model provider you want as a seat (start with **one or two** — you can add seats later).
- [ ] A payment method or prepaid credit on each provider account, **with auto-reload turned OFF** (see Costs).
- [ ] A computer that stays on at home if you want the local seat (homebase). Optional — you can start cloud-only.
- [ ] A safe place for API keys: your assistant's secure credential storage if it has one, or a password manager. **Never** paste keys into chat logs, issues, or files you might share.
- [ ] About an hour for a first, minimal version: one cloud seat + the local seat is a perfectly good start.

**Skill level:** if you can copy a config file, paste an API key into a secure prompt, and run one command in a terminal, you can do this.

---

## Step-by-step setup

### Step 0 — Set your spending caps FIRST

Do this **before** creating any keys. On each provider's billing page:

1. Add the smallest amount of credit you're comfortable testing with (single-digit dollars is plenty to start).
2. **Turn auto-reload / auto-recharge OFF.**
3. Set the provider's own usage/budget alert if one exists.

A council multiplies your spend: one question to three cloud seats is three billable calls, plus your main assistant's own work. Caps are what make experimenting safe. Full details in [Costs & cost caps](#costs--cost-caps).

### Step 1 — Create one API key per seat

For each cloud provider:

1. In the provider's developer console, create a **new key dedicated to this project** (a name like `council-seat` helps future-you).
2. Store it immediately in secure storage — don't leave it in a downloads folder, a note, or a screenshot.
3. Treat a chat subscription and API access as **different products**: a paid chat plan with a provider usually does **not** include API credit. API use is billed separately, per use.

> **If a key ever leaks** — posted, committed to a repo, screenshotted — **revoke it and create a new one.** Deleting the file it was in does not un-leak it.

### Step 2 — Connect each seat to your assistant

The exact mechanics depend on how your assistant accepts custom model connections, but the pattern is the same every time:

1. Create a connection for the seat with:
   - a **seat name** you'll recognize (`seat-a`, `seat-b`, `seat-c`, `local`),
   - the provider's API endpoint,
   - the key, entered through the secure prompt (never typed into ordinary chat),
   - a **pinned model version string** — copy the provider's exact current model ID; don't use a floating "latest" alias for a seat you depend on (see Maintenance).
2. Send a **hello test**: one short question with a known answer ("Reply with the word: ready"). Confirm the right model answers and that the cost shows up where you expect in the provider's usage page.
3. Repeat per seat. **Test each seat alone before running them together** — when a full consult fails, you want to know which seat is the problem.

### Step 3 — (Optional) Set up the local seat on homebase

1. Install a local model runner on homebase (Ollama and llama.cpp are the common open options).
2. Pull **one modest open-weight model** that fits comfortably in homebase's memory — a small model that actually runs beats a huge one that swaps and crawls. As a rough guide, a quantized 7–9B-class model is a sensible first local seat on typical home hardware.
3. Make the runner reachable **only inside your own private network** (for example, over your own encrypted mesh/tailnet tool or your LAN). **Do not forward its port to the public internet** — a local model endpoint has no login of its own.
4. Connect it to your assistant the same way as a cloud seat (endpoint + no key, or a placeholder if the form insists), and run the same hello test.
5. Optional, later: add a second, slower **reasoning** model as a deep-pass seat for the hardest item in a meeting. Keep the small fast model as the default.

### Step 4 — Write down the lanes (who does what)

Seats are most useful when each has a job. A starter set of lanes:

| Seat | Lane |
|---|---|
| Local seat | First pass on private/local material: summarizing, cleaning notes, pulling fields out of documents, deduping, tagging, routine drafts, simple scripts, basic analysis. |
| Cloud Model A | Careful general reasoning; drafting and critique on consequential questions. |
| Cloud Model B | Fast, cheap checks; second opinions on everyday questions. |
| Cloud Model C | Diversity seat — a different model family on purpose; good at spotting what the others assumed. |
| Local deep-pass (optional) | One bounded hard-reasoning pass on the single hardest item; slow is fine. |

And an equally important **not-its-job** list for every seat:

- Live or current facts (prices, schedules, availability) — the main assistant gathers evidence; seats reason over it.
- Final calls on legal, medical, financial, or security questions.
- Acting on anything. Seats advise; the main assistant verifies; **you approve.**

### Step 5 — Define your consult brief format

Keep it short and identical every time. A format that works:

```
QUESTION: <one clear question>
CONTEXT: <only the facts a seat needs — nothing personal beyond that>
EVIDENCE: <primary-source facts the main assistant already checked, pasted in>
RETURN: your recommendation, your key assumptions, and anything you
        think should be verified before acting.
```

Two habits make consults dramatically better:

- **Evidence packets.** Before a consequential consult, the main assistant checks the real sources first (actual prices, actual dates, the actual document) and pastes the findings in. Seats reasoning over real evidence beat seats reasoning from memory — this was, in the original build this guide is based on, a bigger quality jump than adding another seat would have been.
- **Independence.** Send the brief to all seats at once, without each other's answers. Collect first, compare after.

### Step 6 — Run your first consult

Pick something low-stakes and real. Then:

1. Have your assistant send the brief to every seat.
2. Read **all** the answers yourself at least the first few times — you're calibrating how much to trust each seat.
3. Ask your assistant for the synthesis: agreements, disagreements, and what it verified on its own.
4. Check the usage pages. Note what the consult cost. Adjust caps or lanes if it surprised you.

### Step 7 — (Optional) Hold regular "meetings"

Once one-off consults feel routine, a standing rhythm can help:

- A short recurring meeting (for example, every other day) with a fixed agenda: review what changed since last time, each seat proposes **one** improvement, **one** gap, **one** idea.
- Minutes get saved as plain notes you can skim.
- The meeting **proposes; you decide.** No purchases, installs, or external changes come out of a meeting without your explicit approval.

Start this only if you'll actually read the minutes. A meeting nobody reads is just a recurring bill.

---

## A sample consult (fully synthetic)

*Invented from scratch for this guide. Nova, homebase, the people, and the numbers are all fictional.*

**Question:** "Sam is choosing between two secondhand laptops for home use and light photo editing. Laptop Pine: $420, newer chip, 16 GB RAM, small screen. Laptop Cedar: $350, older chip, 8 GB RAM, larger screen. Which is the better buy?"

**Evidence packet Nova gathered first:** the two listing pages (prices, specs, battery health percentages, return windows), and a price check showing Pine's model typically lists at $450–$500 used.

**How the seats answered:**

- **Seat 1 — Cloud Model A:** "Pine. The RAM difference decides photo-editing comfort, and the price is below typical for that model. Assumption: battery health in the listing is honest."
- **Seat 2 — Cloud Model B:** "Pine, but ask the seller for a photo of the battery-cycle screen before paying — that's the one unverifiable claim."
- **Seat 3 — Cloud Model C:** "Cedar, *if* the larger screen matters more than speed and the budget is tight — the $70 saved covers an external drive. Otherwise Pine."
- **Local seat:** Summarized the two listings into a spec table and flagged that Cedar's listing doesn't state battery health at all.

**Nova's synthesis to Sam:** "Three of four lean Pine, for the RAM and the below-typical price. The honest disagreement is Cedar's screen and price. Two things to verify before money moves: Pine's battery health (seller photo) and Cedar's missing battery info. My recommendation: Pine, conditional on the battery photo."

Notice what happened: the seats didn't just vote — they surfaced the *one unverified claim* the decision depended on. That's the pattern you're building for.

---

## Costs & cost caps

Read this twice. It's the section that keeps a fun project fun.

- **Every seat is a separate bill.** One consult to three cloud seats = three API calls, each billed by its own provider, **plus** your main assistant's own usage. A subscription to a provider's chat product does not pay for its API.
- **Costs scale with length.** Long briefs, long evidence packets, and long answers all cost more. Short briefs are a cost feature, not just a style choice.
- **The local seat is your pressure valve.** It costs electricity, not tokens. Route high-volume, low-stakes work (summaries, cleanup, first drafts) to it first.
- **Practical starting posture:**
  - Small prepaid balance per provider; **auto-reload OFF everywhere.**
  - Provider budget alerts on.
  - One consult on a question that matters > five consults on trivia.
  - If a provider offers per-feature paid extras (live web search is a common one), treat each as a separate spending decision — leave them off by default, and approve them deliberately, not as a bundle.
- **Keep a cost line.** After notable consults, note what they cost. Patterns appear fast, and they're how you learn which seats earn their keep.
- **Agreement ≠ correctness, even at a price.** Paying four models to agree doesn't make a fact true. Budget a little skepticism along with the tokens.

---

## Privacy & safety habits

- **Seats get briefs, not your life.** Send the facts a question needs — not your full history, household details, or documents "just in case." One provider may have a free tier whose terms treat your inputs differently from a paid tier; know which tier each seat is on, and keep personal material off free tiers.
- **Seats don't hold memory.** Treat every consult as stateless: if it matters, it's in the brief; if it's not in the brief, the seat doesn't know it.
- **Local-first for private work.** The local seat exists partly so the most personal material never leaves homebase.
- **Keep endpoints private.** Model endpoints — cloud keys and your local runner alike — are not things to expose publicly. Private network only for the local seat; keys only in secure storage.
- **Human approval for real-world actions.** Spending, contacting people, publishing under your name, and making commitments always need your explicit yes — no matter how unanimous the council is.

---

## Maintenance

A council guide (and a council) goes stale in predictable ways. Plan for it:

- **Pin your model versions.** In the table below, record the *exact* model version string each seat uses. Providers retire and rename models; a pinned table turns "seat 2 stopped working" into a two-minute fix instead of a mystery.

  | Seat | Provider | Exact model version | Date pinned |
  |---|---|---|---|
  | Seat 1 | Cloud A | `model-a-exact-version` | YYYY-MM-DD |
  | Seat 2 | Cloud B | `model-b-exact-version` | YYYY-MM-DD |
  | Seat 3 | Cloud C | `model-c-exact-version` | YYYY-MM-DD |
  | Local | homebase | `local-model-name-and-size` | YYYY-MM-DD |

- **Date-stamp everything.** When you publish or update a guide like this one, stamp it: "Tested as of YYYY-MM-DD." A date-stamped guide that's old is honest; an undated guide that's old is a trap.
- **Review on a cadence.** Every month or two: click every seat's hello test, confirm prices haven't changed materially, confirm the steps still match the providers' current screens.
- **Re-test from scratch occasionally.** The gold standard for a setup guide: follow your own instructions in a fresh environment and see if they still work end-to-end.
- **Archive, don't abandon.** If you stop maintaining a setup or a guide, say so at the top — "Archived, no longer maintained, steps may be out of date" — and leave it readable. An archived guide helps; a silently rotten one hurts.
- **Watch your reading time.** If consults or meeting minutes pile up unread, shrink the council (fewer seats, fewer meetings) rather than letting it run unread. The scarce resource in this whole design is your attention, not tokens.

---

## Troubleshooting quick hits

- **One seat errors, others fine** → that provider's key, balance, or model version. Check the usage page first, then the pinned version string.
- **A seat answers as if it were a different seat** → it happens occasionally with some models on persona-heavy prompts. Keep seat prompts plain and factual; judge the substance, and note repeat offenders in your lanes.
- **Costs higher than expected** → shorten briefs, route first passes to the local seat, and check whether any paid extra (like live search) got switched on.
- **All seats agree but it still feels wrong** → trust the feeling; agreement isn't verification. Ask the main assistant to check a primary source.
- **Local seat very slow** → the model is too big for homebase's memory. Drop to a smaller quantized model; a fast small seat gets used, a slow big one gets skipped.

---

## A note on issue templates

If you publish this guide as a repo, add an issue template (for example under `.github/ISSUE_TEMPLATE/`) that says, at the top, in plain language:

> **Do not post secrets or personal information here.** No API keys, passwords, one-time codes, account emails, device names, network names, or screenshots of your accounts. If you posted a key by accident, revoke it immediately — deleting the post is not enough. Issues are public.

A matching note in the README (like the disclaimer at the top of this guide) helps too. Beginners leak keys by accident, in good faith, while asking for help — a clear warning is a kindness to them.

---

## Suggestions & improvements are always welcome

This guide is a starting point, not a finished product — and it gets better the same way a council does: more perspectives.

- **Ideas and upgrades:** found a cheaper seat, a simpler setup step, a better local model, a clearer way to explain any of this? Please share it. Open an issue describing the improvement, or a pull request if you've already written it.
- **Corrections count too:** a step that no longer matches a provider's screens, a price that moved, a version that got retired — flagging that is a real contribution, and it's how this guide avoids going stale.
- **Beginner reports are especially valuable:** if you followed this as your first council and got stuck somewhere, that stuck point is exactly what the next reader needs clarified. Say where it happened.
- Two ground rules, same as everywhere in this guide: **never include secrets or personal information** in a suggestion (no keys, no account details, no screenshots of your accounts), and please be patient — reviewing suggestions is best-effort, in spare time, with no guaranteed response time.
- Good suggestions may be folded into the guide with a thank-you in the commit or release notes. By suggesting a change, you're agreeing it can be shared under the guide's license.

---

## A companion pack — planned, not promised

This guide is, and stays, **complete and free**. Everything you need to build a working council is in the page above — nothing essential is held back.

A small paid **companion pack** may be offered later, purely as a convenience: ready-to-copy worked examples, fill-in templates (consult briefs, evidence packets, meeting minutes), cost worksheets, and a troubleshooting playbook — the tedious setup artifacts, done for you, so you don't have to retype or rebuild them from the guide.

Whether it ever exists depends on real reader interest — people actually building councils from this guide and asking for the templates — not on a launch date. If it never happens, this free guide loses nothing.

---

## License

This guide is released under the **MIT License** (see the `LICENSE` file in this repo), so you can copy and adapt the pattern freely. If you adapt it, a link back is appreciated but not required.

---

*Built as a fictional demonstration: Nova, homebase, Sam, and every example above were invented for this guide. Copy the pattern; invent your own names.*
