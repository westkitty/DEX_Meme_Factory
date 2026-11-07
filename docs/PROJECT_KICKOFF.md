# Dexter Meme Factory - Project Kickoff and Build Packets

**Project:** Dexter Meme Factory\
**Repository:** https://github.com/westkitty/DEX_Meme_Factory\
**Date:** 2026-11-07\
**Workstream:** New independent project, not a submodule of `Dex_Scheduler`\
**Current deliverable:** Planning and developer handoff documents, not application code\
**First release goal:** A private, shareable, offline-first Dexter meme game for Andrew and Greyson

## 1. Project declaration

Build a local-first comedy tool around **four experiences**: (1) one-press Dexter judgments, (2) a voice-consistent Comedy Brain, (3) a coherent Situation Machine, (4) two-player pass-and-play Meme Combat. Support those experiences with a real-photo vault, minimal meme editor, standalone PNG exports and a reversible local pack import/export.

The point is to create funny, **useful output**, not a static page of imagined features. Dexter's authentic photo likeness is a functional invariant. v1 cannot generate or replace Dexter with an AI dog. No account, cloud dependency, network multiplayer or remote image API belongs in the baseline.

**Two users:** Andrew and Greyson. **One device for Combat:** both players take turns on the same device. Other devices work independently until a user manually transfers a private pack.

## 2. Source-of-truth order

1. This project's **actual code, tests and runtime behavior** once implementation exists.
2. [DEVELOPER_HANDOFF.md](DEVELOPER_HANDOFF.md): technology, privacy, domain and testing contracts.
3. [SCREEN_BY_SCREEN_PLAN.md](SCREEN_BY_SCREEN_PLAN.md): UI/UX behavior and screen states.
4. This file: phases, handoffs, acceptance and known risks.
5. [README.md](../README.md): product summary and scope.

A later verified decision should update the relevant file explicitly instead of silently violating these contracts. Never assume an agent's verbal `implemented`, `tested`, `pushed` or `deployed` claim is proof; inspect commits and runnable results.

## 3. Release boundary

### Required for v1

- [ ] Photo Vault with at least 20 real Dexter photos, manual expression tags, favorite and auto-match.
- [ ] One-press Judgment using local captions and an actual imported photo.
- [ ] Comedy Brain with six distinct tone modes, at least 120 reviewed captions, exclusions and last-ten anti-repeat behavior.
- [ ] Situation Machine: compatible role/location/problem choices, individual locks and rerolls, meaningful joke influence.
- [ ] Meme editor with editable text, crop/fit and export of **one standalone PNG** per action.
- [ ] Andrew/Greyson best-of-three Meme Combat: separate hidden turns, reveal, manual vote/tie, verdict and local score.
- [ ] Explicit pack export/import including photos and editable drafts, with safe failure handling.
- [ ] Offline-ready app shell after successful caching; browser storage reliability and recovery clearly signaled.
- [ ] Mobile/desktop, accessibility and reduced-motion test evidence.

### Explicitly excluded

AI image generation, OpenAI API integration, internet-required jokes, generated Dexter lookalikes, photo uploads in public Git, cloud storage/sync, accounts, password screens, live two-device duels, mandatory timers, GIF/video export, automatic background removal, third-party tracking or ads.

### Critical truth about locations

Situation Machine can **invent** a location in text, but v1 does not magically move Dexter into a photograph of that location offline. To get new photorealistic locations later, commission an explicit optional AI feature with a protected backend and user-chosen photo upload. This is future work, not a hidden dependency.

## 4. Milestone slices with concrete packets

Each slice should land as a narrow change set, include tests and preserve previously passing behavior. Do not create a monolithic all-at-once commit.

### M0 - Repository and privacy foundation

**Deliverable:** project skeleton, docs wired, instructions for development, ignore rules for local photo packs/secrets, basic test runner.

**Tasks:**
- Establish the actual branch, `main`, and preserve any existing contents (this repo was empty at initial inspection on 2026-11-07).
- Add `app/` shell and no-dependency module boundaries as proposed in handoff.
- Add `.gitignore` for `.env*`, local packs, private asset directories, generated exports, logs and tool caches. Never commit real Dexter photos even if Git does not flag them.
- Add deterministic development/test commands once tooling is chosen. No need for a cloud build system on day one.

**Acceptance:** repo builds/opens from a clean clone; no secrets or user photographs in tracked files; README status updated from `planning` only after real code exists.

### M1A - Real-photo Vault

**Deliverable:** import real photos, tag, favorite, select; use IndexedDB with an explicit volatile-storage fallback.

**Dependencies:** M0.\
**Tasks:** safe image decoding and rejection, nondestructive originals and thumbnails, structured photo records, gallery empty/error states, browser reload proof, source-photo selection callback.

**Acceptance:** import and tag at least 20 normal photos; select photo; reload and see originals plus tags on a stable-origin browser; if storage is unsupported, display loss warning and give an export path. No generic dog substitute.

**Stop condition:** no detectable original-photo import/persistence route. Do not proceed to the joke UI by inventing static photos.

### M1B - Judgment + minimal editor + one PNG

**Deliverable:** one-button locally authored joke attached to a selected real photo; edit and export.

**Dependencies:** M1A.\
**Tasks:** initial small curated joke set, tone picker, photo/tag filter, canvas crop/fit, top/bottom text, three styles, position controls, deterministic PNG renderer, error recovery.

**Acceptance:** `open -> Judge -> Edit -> Save PNG` works with airplane mode enabled (once app shell is available offline). Export exactly one decoded 1080x1080 PNG with source image and readable edited caption, matching preview. Draft remains after export.

**Stop condition:** blank or composite output, unreadable text, non-local photos or changes lost on error.

### M2A - Full Comedy Brain

**Deliverable:** substantive comedy content and repeatable selection algorithm.

**Dependencies:** M1B.\
**Tasks:** at least 20 distinct reviewed captions for each of six tones, theme metadata, anti-repeat history of 10, exclusions including `Treats`, mood/photo matching and explicit fallback when no eligible captions remain.

**Acceptance:** a test suite demonstrates tone-specific output, recent-history behavior when enough eligible jokes exist and persisted exclusions. Editorial review rejects obvious filler, triple-name banners and a disproportionate concentration of snack gags.

**Stop condition:** more templates are being counted as quality without visible differences in voice or topic.

### M2B - Situation Machine

**Deliverable:** roles, locations and complications with individually lockable rerolls and joke influence.

**Dependencies:** M2A.\
**Tasks:** curate at least 30 representative coherent combinations, add compatibility logic, implement three-slot controls, bridge situation tags to Comedy Brain.

**Acceptance:** lock a role, reroll everything else five times, role never changes; invalid combinations are rejected and explained; changing the premise changes eligible captions or actual tokens; user can export resulting meme.

**Stop condition:** the premise appears on screen but has zero effect on the meme output.

### M3 - Meme Combat

**Deliverable:** private two-player, one-device best-of-three game.

**Dependencies:** M1B, M2B.\
**Tasks:** setup with Andrew/Greyson labels, frozen shared photo/premise, protected player 1 editing, opaque device handoff, player 2 editing, reveal, manual winner/tie selection, scores, local resume and separately exportable memes.

**Acceptance:** complete a three-round session disconnected; player 2 cannot access player 1's caption before reveal by back navigation, recents, page reload or UI routing; a tie does not award a win; scores are correct; each player's meme downloads as its own PNG.

**Stop condition:** any premature leak of hidden draft or irreversible loss during the handoff.

### M4 - Offline shipping and recovery

**Deliverable:** PWA install/cache, local backup/import and cross-device verification.

**Dependencies:** M1A-M3.\
**Tasks:** service worker app-shell precache, offline readiness state, manifest/icons, compatible IndexedDB migrations, `.dexpack.json` export/preview/restore with readable-photo warning, import safety/atomicity tests.

**Acceptance:** load once online, verify offline-ready, disable network, reopen and exercise all four modes and export; import a 20-photo pack in a clean browser profile and compare photos/tags/drafts; malformed import leaves prior data untouched.

**Stop condition:** relying on `file://` browser storage as guaranteed persistent, or showing offline-ready before caching succeeds.

## 5. Task dependency map

```text
M0 repository hygiene
  -> M1A authentic photo import & storage
     -> M1B Judgment + editor + PNG
        -> M2A Comedy Brain full catalog
           -> M2B Situation Machine
              -> M3 pass-and-play Combat
                 -> M4 offline packaging, import/export and release gate
```

Export/backup plumbing should begin in M1A if storage risk warrants it; its full restore test is owned by M4. Technical architecture can evolve, but the evidence gates and private-photo rule must survive any refactor.

## 6. Acceptance test ledger

| ID | Test / user journey | Pass oracle |
| --- | --- | --- |
| P01 | First opening with zero photos | Clear import path; never a fake Dexter image. |
| P02 | Import 20 original photos | Stored originals, separate thumbnails, tags and favorites recover. |
| J01 | Judgment one-click | Actual Dexter photo, readable caption and editable draft. |
| J02 | Default comedy diversity | Six visibly different tones, no repetitive default treat fixation. |
| J03 | Repetition/exclusion | No exact repeats in eligible last 10; banned theme never returns. |
| S01 | Locked premise part | Locked component immutable across rerolls; incompatibility handled. |
| S02 | Situation influences joke | Changing premise changes an observable output. |
| E01 | Canvas export | Exactly one 1080x1080 decoded PNG, preview parity, no watermark. |
| E02 | Editor recovery | Crop, text and photo survive safe reload or recover from backup. |
| C01 | Two-player concealed handoff | No player 1 reveal until both submit. |
| C02 | Best-of-three and ties | Correct votes, score and separately downloadable memes. |
| O01 | Network off after precache | All four modes, editor, local import and export still work. |
| O02 | Backup/restore | 20 original photos + draft metadata round-trip between profiles. |
| O03 | Corrupt pack | Import rejected without modifying existing data. |
| A01 | Responsive and keyboard | Primary controls operable on phone and desktop without pointer-only traps. |
| A02 | Reduced motion and contrast | Same state info and controls without animation; captions readable. |
| G01 | Privacy audit | No personal photos/secrets in Git, no runtime network image upload. |
| G02 | Git proof | Clean changes committed, pushed and read back at expected branch. |

Mark each test `PASS`, `FAIL`, `BLOCKED` or `UNVERIFIED` with actual evidence. Missing proof is not a pass. Do not use fabricated screenshots or unexecuted test commands.

## 7. Cross-device and offline proof matrix

| Environment | Online initial load | Cached offline relaunch | Photo persistence | PNG export | Import pack | Combat |
| --- | --- | --- | --- | --- | --- | --- |
| Mac Safari | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED |
| Mac Chrome/Brave | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED |
| Android Chrome/Brave | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED |
| iOS Safari, when available | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED | UNVERIFIED |
| Local `file://` desktop | session-only fallback; **not** a PWA | N/A | unproven, must warn | UNVERIFIED | UNVERIFIED | UNVERIFIED |

No device is certified by writing this matrix. A test operator must perform these paths on actual browsers. Never equate `online page loaded` with `offline relaunch proven`.

## 8. Risks and countermeasures

| Risk | Severity | Control and evidence |
| --- | --- | --- |
| Stale/generic Dexter lookalike | Critical | Original photos only; never ship generated substitutions. |
| Public asset leakage | Critical | No private image files in Git; pre-commit diff/secret review. |
| Browser storage loss / quota | High | Status + explicit pack export; restore test with separate profile. |
| Unfunny/repetitive captions | High | Six edited tone banks; editorial review + last-ten and theme tests. |
| PWA offline falsely advertised | High | Explicit shell-ready lifecycle state, offline relaunch proof. |
| Hidden turn exposure | High | Combat-specific state machine and opaque handoff screen; back/reload tests. |
| Text obscuring Dexter's face | Medium | Default safe caption zones, fit preset, preview/PNG pixel tests. |
| Mobile oversized media and packs | Medium | Thumbnails, bounded decoding, size warnings, transactional import. |
| Unneeded API/secrets added early | High | No backend nor tokens in v1; reject unrelated dependencies in diff review. |
| Four docs diverge during coding | Medium | Actual code/tests first, then update exact controlling contract with evidence. |

## 9. Operational rules for a coding agent

1. Read this kickoff, the README, developer handoff and screen plan **before** changing the repository.
2. Inspect the actual branch, existing files and any work in progress; do not assume the repository is still empty after initial planning.
3. State a bounded slice from M0-M4. Build and test only that slice. No unrelated redesign, third-party library spree, API integration, or deployment.
4. Preserve all verified capabilities as further slices land. Add or repair tests when a user-visible contract changes.
5. Prefer direct, reversible commits with descriptive messages. Never force-push or overwrite a user's branch.
6. Record what actually ran, what passed, what remains unverified and the commit SHA. Do not call an untested UI `complete`.
7. **Never add user's real photo originals to public Git.** If a demonstration needs images, use synthetic abstract test fixtures clearly labeled `not Dexter` and never treat them as an authentic likeness.

### Initial Git workflow (empty repository only)

```sh
git init -b main
git remote add origin https://github.com/westkitty/DEX_Meme_Factory.git
git add README.md docs/DEVELOPER_HANDOFF.md docs/SCREEN_BY_SCREEN_PLAN.md docs/PROJECT_KICKOFF.md
git commit -m "docs: bootstrap Dexter Meme Factory build contracts"
git push -u origin main
```

If the repository already has commits, **clone/fetch and branch from its real current head instead**. The command block documents intended behavior, not evidence that any command was executed. For a GitHub-connected agent using the Contents/Git Data API, equivalent Git objects, a fast-forward branch update and readback are acceptable; report the mechanism honestly rather than claiming a terminal push occurred.

## 10. First coding-agent task packet (copy-ready)

> **Repository:** `westkitty/DEX_Meme_Factory`, read its actual current `main` before edits.\
> **Objective:** Implement M0 plus M1A only: an offline-capable static app shell with a local Dexter Photo Vault. Users must be able to import their own genuine photo files, see a grid, tag/favorite/select an image, recover that image and metadata after reload on a stable-origin browser, and see a clear warning when persistent storage is unavailable.\
> **Do not:** implement AI generation, cloud calls, accounts, meme combat, a new image model, public Dexter image files, fake lookalike assets, remote fonts, tracking or deployment. Preserve all existing files and docs.\
> **Contracts:** follow `docs/DEVELOPER_HANDOFF.md` for data/storage/privacy and `docs/SCREEN_BY_SCREEN_PLAN.md` for Home/Photo Vault UI; provide a local in-memory fallback with explicit session-storage warning.\
> **Required proof:** clean-open app, 20-photo import/reload, invalid image rejection, proper original-blob preservation, tags/favorites and private photo audit, keyboard/mobile basic interaction, no runtime network requirement after initial load when cache-ready. Add relevant automated tests and record any browser proof still unverified.\
> **Git:** stage only authorized files; commit a cohesive slice and push to the existing authorized branch only after tests. Report exact commit, changed files, test results, known gaps and the next bounded task. No force push and no pretend deployment.

## 11. Project completion gate

**Documentation delivery** is complete when the four Markdown files exist, cross-link correctly, are staged/committed/pushed to `main` and are read back from GitHub. That does **not** imply that any software exists.

**Product v1 completion** is a separate later gate: P01 through G02 tests supported by actual runtime and repository evidence, including real-photo privacy, one-PNG outputs, both players' full pass-and-play journey, portable local backup and device/network-off acceptance. If any mandatory item is missing, v1 remains incomplete.

**The next action after documentation:** execute the M0 + M1A coding task above, then return with source and test evidence before moving to M1B.
