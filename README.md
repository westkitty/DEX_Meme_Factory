# Dexter Meme Factory

**A small offline-first comedy studio starring the real Dexter.**

Make a sharp, shareable meme in seconds. Invent absurd situations, let Dexter issue a ruling, or challenge a friend to a meme duel. The app is designed for Andrew and Greyson, with no account, subscription, tracking, or cloud service required for core use.

> **Project status (2026-11-07):** Planning and implementation handoff only. This repository currently contains product and build documents, **not** a functioning app or deployed site. Do not mistake planned capabilities for completed features.

## The pitch

Dexter is not a generic dog mascot. He is the household's compact, floppy-eared, famously unimpressed authority. His real photographic likeness is the point. The comedy should be fast, distinctive, deadpan and occasionally absurd; not a stream of repetitive treat jokes.

The first release has four game-like experiences plus the small set of editing and storage capabilities needed to make their output useful:

| Mode | What it does | First-release boundary |
| --- | --- | --- |
| **Judgment Button** | One press assembles a Dexter photo, verdict and editable caption. | Curated offline jokes; no API call. |
| **Comedy Brain** | Six authored voices, situational captions, recent-history de-duplication and topic exclusions. | Rule-based local library, not an LLM. |
| **Situation Machine** | Mixes role, location and complication; reroll or lock individual parts. | Generates a premise and matching text, **not** a newly synthesized background. |
| **Meme Combat** | Andrew and Greyson make competing memes from one prompt in pass-and-play rounds. | One device, two players, manual winner, no network multiplayer. |

Supporting tools: a private Dexter Photo Vault, a small caption editor, individual PNG export and a portable backup/import pack.

## The everyday loop

1. Open the factory and choose **Judge**, **Situation**, or **Duel**.
2. See a draft using a real, locally imported photo of Dexter.
3. Reroll the joke, swap photos or edit the caption.
4. Save **one** full-resolution PNG, or challenge Greyson.

The home screen must remain immediately usable after setup; no onboarding maze or dashboard full of unused controls.

## Non-negotiable product rules

- **Real Dexter only.** The v1 meme canvas draws from user-imported actual photos, retaining his floppy ears, dark coat, cream brow/cheek markings and pale muzzle/chest. No generated lookalikes or substituted dogs.
- **Photos stay private.** Do not commit personal photo assets to this public repository, put them in the deployed site's asset folder, or send them to analytics/cloud services. Users must explicitly choose any future upload.
- **Offline means tested offline.** Core creation, import, editing, combat and PNG export must work with the network disabled after initial setup. No runtime CDNs or remote fonts.
- **One meme is one image.** Exports are independent PNG files. No accidental grids, montages, watermarks or extra posters.
- **Funny before flashy.** Six deliberately different voices, strong premises and actual editorial review beat a giant list of interchangeable template strings.
- **Only Andrew and Greyson.** Those are the two named players for v1. Player labels may be edited locally; no third account, account creation or server identity model.
- **No mandatory AI.** Later optional image generation can have its own protected API backend. Never place a secret key inside browser HTML or a public repository.

## Offline and sharing model

**Preferred first-run distribution:** a lightweight static PWA, optionally served on GitHub Pages after implementation. Install/load once while online, confirm the service worker has cached the app shell, then use it offline. App state and user photos live on that device in IndexedDB; they do **not** automatically synchronize.

**Portable backup:** an explicit export/import `.dexpack.json` format that includes selected photos, captions, settings and memes. Give Greyson a copy of the app and a privately shared pack; importing the pack is a deliberate separate action. Sensitive images do not pass through the repository.

**Desktop fallback:** a downloadable self-contained HTML package may run directly from disk for basic editing, but `file://` storage behavior varies by browser. Do not claim reliable persistence for that mode without a browser-specific test; make backup/export available even when durable storage is not.

This split is intentional: service workers require an appropriate secure origin and do not make a brand-new website magically offline before caching. See [MDN service workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) and [MDN localStorage limitations](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).

## v1 content commitment

- Six tone families: **Deadpan**, **Petty**, **Royal Decree**, **Concerned Elder**, **Cosmic Witness**, **Full Stinkweasel**.
- At least **120 reviewed captions** (at least 20 per tone); stored locally as editable data, not hidden in UI conditionals.
- At least **30 coherent situations** in the initial curated set, generated from individually selectable roles, locations and problems with compatibility constraints.
- No exact caption repeats in the preceding **10** generations when the available eligible pool permits it; provide an honest fallback when filters reduce the pool.
- A **Treats** topic exclusion and topic-frequency balance so food jokes do not dominate the default experience.
- One-device, pass-and-play **best-of-three** Meme Combat, with private turns, reveal and a manual vote.

## User stories that define success

- **Andrew:** "I want a genuinely funny Dexter meme with one click, without losing his likeness or editing twenty settings."
- **Greyson:** "I want to open my copy, use the photos that were shared with me and make/export a meme without an account or internet."
- **Together:** "We want to challenge one another using the same prompt without seeing each other's work before the reveal."
- **Both:** "If a browser clears its storage, I can restore from a pack rather than lose all our photos and drafts."

## Build milestones

| Milestone | Deliverable | Gate |
| --- | --- | --- |
| **M1: Playable core** | Photo Vault, local jokes, Judgment, editor and PNG export | Real photo -> one-click draft -> one verified PNG, offline. |
| **M2: Replayability** | Six tones, situation controls, exclusions, anti-repeat | Premises affect jokes and controls remain coherent after reload. |
| **M3: Shared fun** | Pass-and-play Meme Combat, local scoring, portable pack | Both players finish a best-of-three, offline, with import/export recovery. |
| **Release candidate** | Responsive UI, service-worker cache, accessibility and storage tests | Cross-browser offline and backup/restore acceptance matrix passes. |

## Repository documents

1. [Developer handoff and technical contracts](docs/DEVELOPER_HANDOFF.md) - architecture, data models, modules, APIs, security, tests and do-not-break rules.
2. [Screen-by-screen implementation plan](docs/SCREEN_BY_SCREEN_PLAN.md) - navigation, actions, interaction states, responsive layouts and accessibility.
3. [Project kickoff and build packets](docs/PROJECT_KICKOFF.md) - phased tasks, dependencies, acceptance checklist, risks and first coding assignment.

This README is the concise product brief. The other three files are complementary, not copies of it.

## Planned tech posture

- Dependency-light HTML/CSS/JavaScript, Canvas 2D for final PNG rendering.
- IndexedDB for photos, drafts, templates and session records; Cache Storage for static app-shell assets.
- Local, structured and reviewed humor data. No AI call in v1.
- A small test harness that validates the meme engine, pack round-trip, combat state machine, cropping/lettering and offline behavior.
- Static hosting is an **optional deployment step**, not required for local development.

## How to start development

Read [Project Kickoff](docs/PROJECT_KICKOFF.md), implement its M1 task packet, then use [Developer Handoff](docs/DEVELOPER_HANDOFF.md) as the contracts and [Screen Plan](docs/SCREEN_BY_SCREEN_PLAN.md) as the UX acceptance oracle. Do not replace the product with a generic meme maker, upload private Dexter photos, introduce accounts, or bolt on a premature API integration.

## License and assets

No license has been selected yet; do not assume third-party reuse rights. Real Dexter images are private user-provided assets and are **not** bundled into this public repository. Any future fonts, illustrations or packages must have their provenance and redistribution terms checked before inclusion.
