# Pre-Publish Scrub Checklist

*Run every item, in order, before the first push of any guide repo. This list comes from a multi-model review of how "sanitized" projects actually leak: rarely through what you write — usually through what you copy.*

## The fresh-repo rule

- [ ] This repo is **brand new**. It was never a working setup, and it does not share history with one.
- [ ] **Never publish a "cleaned copy" of a real repo.** Deleting a secret from a file does not delete it from git history. If the material started life in a real project, retype/rebuild it fresh here instead.
- [ ] First commit happens **after** `.gitignore` is in place (see below) — not before.

## Secrets

- [ ] Run a **secret scan before the first push** (e.g. `gitleaks`, `trufflehog`, or GitHub's built-in scanning) on the whole repo, and re-run it after any big edit.
- [ ] Grep the full text for anything key-shaped: `key`, `token`, `secret`, `password`, `Bearer`, `sk-`, long random-looking strings.
- [ ] All example configs use **placeholders only** (`.env.example` with fake values). No real key has ever been typed into any file in this repo — including "just to test it."
- [ ] **If a real credential ever lands anyway: revoke it, don't just delete it.** Generate a replacement. Deletion does not un-leak a credential; revocation does.

## Names, places, and machines (the mosaic check)

- [ ] Every name is fictional: assistant, host machines, people, rooms. Do a find for any real name, nickname, or household member — **zero hits allowed**.
- [ ] No real hostnames, device names, or computer names anywhere — including inside example commands, logs, and screenshots.
- [ ] No network identifiers: no real IP addresses, no tailnet/mesh-network names (a network name is a persistent identifier), no Wi-Fi SSIDs, no MAC addresses, no Bluetooth device names.
- [ ] No real worked examples. Every consult, routine, schedule, and anecdote was **invented from scratch** — not edited down from a real one. Individually harmless details add up to a picture of a real home; the defense is fictional material, not redaction.
- [ ] No production config files (compose files, automation configs) — patterns and placeholders only.

## Files and media

- [ ] **No real screenshots**, or: every screenshot has been through a redaction pass (account emails, device/room names, network names blurred) **and** a metadata strip (EXIF and document properties removed).
- [ ] No real audio — no voice clips of real people, ever.
- [ ] No real prompt logs, chat transcripts, or usage dashboards. Invent examples instead.
- [ ] File metadata scrubbed on any exported documents/images included in the repo.

## Framing and terms

- [ ] README states clearly: **unofficial** community guide, not endorsed by the platform or any provider.
- [ ] Disclaimer present: as-is, no warranty, **reader is responsible for their own API costs**, support is **best-effort only**.
- [ ] Muse's terms and **each model provider's terms** checked for anything that restricts publishing setup documentation like this — checked recently, not from memory.
- [ ] A license file is chosen deliberately (e.g. MIT / Apache-2.0) and included.
- [ ] Issue template (or README equivalent) warns readers **never to post their own secrets**, in plain language, at the top.
- [ ] No proprietary material shipped: no one else's prompts, weights, or license-restricted client code.

## Accuracy

- [ ] Versions pinned: exact model version strings recorded in the README's maintenance table.
- [ ] Guide is **date-stamped** ("Tested as of …") or clearly marked as a draft.
- [ ] Steps tested — ideally followed end-to-end in a fresh environment, not just read over.
- [ ] Secure defaults throughout: local-first examples, authenticated access, least-privilege permissions, **no public port-forwarding instructions**, human approval stated for consequential actions.
- [ ] Cost guidance present, including: multi-model calls multiply spend, caps/auto-reload advice, and "agreement ≠ correctness."

## Final pass

- [ ] Read the whole repo once as a stranger: *could someone reconstruct a real household, network, or person from this?* If yes, fictionalize further.
- [ ] `git log` reviewed — commit messages are clean too (they leak just like files do).
- [ ] Someone (or a second model) other than the author has done one scrub read.
