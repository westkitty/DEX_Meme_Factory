# Developer Handoff - Dexter Meme Factory v1

**Status:** Specification, not implemented\
**Target repository:** `westkitty/DEX_Meme_Factory`\
**Target release:** v0.1 offline-first playable core, then v1 with all four modes\
**Audience:** developer or coding agent starting from a newly initialized repository\
**Owners / users:** Andrew and Greyson (one device at a time; no accounts)

## 1. Objective and non-goals

Build a private-photo, offline-first, responsive meme maker whose main activities are **Judgment Button**, **Comedy Brain**, **Situation Machine**, and **Meme Combat**. Every generated meme begins with a **real photo of Dexter**, a compact senior tricolor dog with floppy ears. The game chooses captions and layouts, never creates a synthetic Dexter in v1. The ordinary user journey is `open -> create -> adjust -> export PNG` in as few interactions as possible.

**Not v1:** OpenAI image generation, background synthesis, cloud sync, accounts, multiplayer networking, GIF/video export, automatic subject segmentation, facial recognition, server-side storage, social feed, paid products or user telemetry. Do not promote these into M1 just because implementation is convenient.

**Non-negotiable:** This GitHub repository is public. **Do not check in Dexter's actual photos**, internal reference vaults, sample personal archives, API keys, browser storage dumps, uploaded files or secrets. User-chosen private transfer of photos and meme packs is outside GitHub.

## 2. Decision record

| ID | Decision | Why |
| --- | --- | --- |
| ADR-001 | Local first; no API for baseline product | Free, reliable and usable by Greyson without accounts or internet. |
| ADR-002 | Actual user-imported photos | Preserves Dexter's likeness; avoids AI lookalike drift. |
| ADR-003 | Static HTTPS PWA as preferred install/distribution | Allows app-shell caching and stable origin-backed local data stores. |
| ADR-004 | IndexedDB for durable user content, Cache Storage for static shell | Photos/blobs differ from source files and app code; no runtime backend. |
| ADR-005 | Canvas 2D export pipeline | Predictable single-image PNG output with offline text rendering. |
| ADR-006 | Curated comedy content and finite rules, not probabilistic calls | Quality control, predictable runtime and cost. |
| ADR-007 | Pass-and-play on one device | Works offline with no synchronization or personal accounts. |
| ADR-008 | Portable, schema-versioned manual export/import | Browser-managed storage is not a permanent backup. |

### Offline compatibility warning

- PWA offline access requires successful first online load/cache on a secure origin (or localhost for local development). Do not promise the first uncached launch works disconnected.
- A self-contained `file://` HTML package can support useful *session-based* editing, but persistent storage behavior varies by browser; it does not receive service worker caching. Clearly surface unsupported/volatile storage status and encourage export.
- Device storage can be cleared or evicted. Provide visible `Back up my factory` / `Restore a pack` actions and an honest status message.

Authoritative references: [MDN Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API), [web.dev offline data](https://web.dev/learn/pwa/offline-data), [MDN localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).

## 3. Proposed project structure (to create during implementation)

```text
/
  README.md
  docs/
    DEVELOPER_HANDOFF.md
    SCREEN_BY_SCREEN_PLAN.md
    PROJECT_KICKOFF.md
  app/
    index.html
    styles.css
    main.js
    manifest.webmanifest
    sw.js
    icons/                     # original, redistributable app icons only
    data/
      captions.json           # >=120 authored lines
      situations.json         # curated roles/locations/problems/rules
    modules/
      state.js                # state machine / router
      photo-vault.js          # local import, tags, thumbnails, storage
      joke-engine.js          # pure deterministic selection
      situation-engine.js     # compatible combinations and locks
      meme-canvas.js          # preview, layout and PNG export
      combat.js               # pass-and-play transitions
      pack-io.js              # versioned backup and import
      storage.js              # IndexedDB and migrations
  tests/
    joke-engine.test.js
    situation-engine.test.js
    combat.test.js
    pack-io.test.js
    canvas-export.test.js
    browser-offline.spec.js
```

The file tree is a proposed future implementation layout. **None of the `app/` or `tests/` files is claimed to exist now.** Use plain JS and browser APIs before adding dependencies. If a bundler is introduced, document why and ensure all runtime dependencies are local, buildable and pinned. Use relative paths so GitHub Pages subpath deployment works.

## 4. Component boundaries and contracts

| Module | Inputs | Output | Side effects |
| --- | --- | --- | --- |
| `photo-vault` | Local image files, user tags, chosen ID | `PhotoRecord` plus safe preview URL | IndexedDB photo+metadata storage |
| `joke-engine` | Tone, tags, situation, exclusions, recent IDs, RNG seed | `JokeCandidate` | None; caller stores history |
| `situation-engine` | role/location/problem catalogs, locks, exclusions, RNG seed | `Situation` plus compatibility metadata | None |
| `meme-canvas` | Photo blob, crop, caption blocks, style, size | On-screen preview and PNG Blob | Download/share user gesture only |
| `combat` | Players, round challenge, drafts, vote event | Transition to next state / winner verdict | Store session snapshot locally |
| `pack-io` | Selected local records, manifest version | Downloadable pack / safe import preview | Transactional import after confirmation |
| `state` | User actions and stored state | Selected screen/view model | Explicit, bounded persistence |

Never let the comedic text engine directly change photo objects, remote destinations or browser file permissions. Never let preview text execute as HTML.

## 5. Domain model and data ownership

Use a versioned IndexedDB database named `dexter-meme-factory`, schema version `1`. Suggested stores: `photos`, `drafts`, `captionTemplates`, `situations`, `combatSessions`, `settings`, `recentJokes`. These are implementation targets, not existing stores.

```ts
// Documentation-only shapes. IDs are locally generated stable strings.
type PhotoRecord = {
  id: string; name: string; importedAt: string; mime: string;
  originalBlob: Blob; thumbnailBlob?: Blob;
  tags: string[]; favorite: boolean; cropPreset?: CropRect;
};
type CropRect = { x: number; y: number; width: number; height: number }; // 0..1

type CaptionTemplate = {
  id: string; tone: ToneId; theme: string;
  top: string; bottom: string; photoTags?: string[];
  compatibleRoleTags?: string[]; compatibleProblemTags?: string[];
};
type ToneId = 'deadpan'|'petty'|'royal'|'elder'|'cosmic'|'stinkweasel';

type Situation = {
  roleId: string; locationId: string; problemId: string;
  locked: { role: boolean; location: boolean; problem: boolean };
};
type TextBlock = {
  id: string; text: string; x: number; y: number;
  size: number; align: 'left'|'center'|'right'; styleId: string;
};
type MemeDraft = {
  id: string; photoId: string; mode: 'judge'|'situation'|'combat';
  tone: ToneId; jokeId?: string; situation?: Situation;
  crop: CropRect; blocks: TextBlock[]; createdAt: string; updatedAt: string;
};
type CombatSession = {
  id: string; players: [string, string]; bestOf: 3;
  roundIndex: number; wins: [number, number];
  state: 'setup'|'turn1'|'handoff'|'turn2'|'reveal'|'vote'|'verdict'|'complete';
  challenge: { photoId: string; situation?: Situation };
  drafts: [MemeDraft|null, MemeDraft|null];
};
```

Do not put image bytes in localStorage. Do not store object URLs as permanent identifiers; recreate URLs from blobs and revoke them on cleanup. Photos remain local unless a user specifically exports a pack. Do not commit examples of personal data to tests; construct synthetic pixel fixtures instead.

## 6. The Comedy Brain algorithm

**Editorial minimum:** 120 distinct caption pairs, at least 20 per tone, with a reviewed mood and topic taxonomy. No recycled triple-name header. Default lines should not be dominated by food/treat themes. Include dry bureaucracy, petty domestic judgments, cosmic resignation, technical failure, royal administration and absurd scenarios.

**Selection pipeline:**

1. Read tone, current photo's human-assigned tags, optional situation and persisted topic exclusions.
2. Filter templates by tone, exclusion tags and compatible scene/photo tags.
3. Exclude the last **10** generated joke IDs where possible.
4. Weight underused themes and appropriate photo expressions. Use a deterministic injectable RNG for testability.
5. Select one candidate, substitute **safe text tokens** from the chosen situation only where explicitly supported, then check length and forbidden words again.
6. Return candidate ID, top/bottom text, tone, reason tags and optional alternatives. The editor owns user-facing wording changes.
7. If filtering makes the pool smaller than required, offer `Loosen filters` or a clearly labeled fallback; never silently reintroduce banned topics or infinite-loop on reroll.

`Generate` is not an LLM endpoint. Tone selection must visibly change construction/rhythm; merely changing labels is a failure. Add editorial snapshots to tests so joke quality can be reviewed rather than inferred from passing type checks.

## 7. The Situation Machine

Catalog data consists of roles, locations and problems. Each item has an `id`, user-friendly `label`, semantic tags and disallowed combinations or allowed families. Generate a **coherent premise**; do not randomly concatenate three grammatically incompatible strings.

Example: `role=maritime inspector`, `location=haunted aquarium`, `problem=the fish have filed a complaint`. A compatible joke might be `THE FISH HAVE FILED THEIR COMPLAINT. / IT HAS BEEN FORWARDED TO THE OCEAN.`

- Reroll `role`, `location` or `problem` independently.
- Locking a field survives `Reroll all` and later caption changes.
- If locks make every combination invalid, explain the conflict and offer to unlock one field. Do not silently override locks.
- Feeding a premise to Judgment must change joke candidate filtering or token content, not merely place metadata in an unused object.
- Offline v1 **does not** place Dexter photographically inside the imagined location. User can still caption the photo, and an optional designed border/background can convey the role without falsifying Dexter's identity.

## 8. Photo Vault and likeness protection

Accepted input: local JPEG, PNG and WebP photos (test actual browser decode support). Validate file bytes through image decode, MIME and size before saving; set a practical per-image limit and show an actionable error when it is exceeded. Target **at least 20** normal phone-sized photos without crashing on target devices.

- Show `No photos yet` on first use, with a real Import button. Never ship a fake/generic Dexter photo placeholder as if authentic.
- Let user set one or more tags, rename a photo and mark favorites. Tag-based auto-selection means matching manually assigned metadata, not face recognition.
- Preserve the original imported blob; generate a small thumbnail only for gallery rendering. Any meme crop is non-destructive metadata.
- Prevent export controls from cropping ears or crucial facial markings by offering `Fit Dexter` as default. Never *force* a crop when users intentionally choose otherwise.
- If a photo cannot be decoded, report it individually and continue importing other valid photos.
- Imported photos stay isolated in the active browser profile. Backups are encrypted **only if encryption is actually implemented and verified**; otherwise say the pack contains photos in readable form and advise private transfer.

## 9. Canvas editor and PNG output contract

Render **one image per export action**: full-frame Dexter photograph with caption blocks, never a collage unless the user explicitly designs one. Use device-independent output pixels (for example, 1080x1080 initial square preset), while editor rendering is responsive. Support square and portrait presets if simple; one solid square preset is preferable to a broken format menu.

Requirements:

- Type editable top/bottom caption and optionally one free text block; select one of three legible typography treatments.
- Drag and keyboard-nudge blocks with bounded positions; text remains on canvas and supports multiline wrapping.
- Reuse the *same* drawing rules for screen preview and high-resolution PNG export. Verify expected pixel dimensions and nonblank alpha/color content.
- Use bundled/system fonts; no remote requests. Use Canvas `toBlob` or equivalent and revoke object URLs.
- Only include desired text. Default captions should not overlay Dexter's face; offer a simple safe text area or adjust crop to protect facial features.
- Export filename `dexter-meme-YYYYMMDD-HHMMSS.png`; collision-safe unique suffix when necessary.
- Prefer click-to-download where available; on mobile, provide a visible Share/Save fallback that handles unsupported Web Share gracefully.

## 10. Meme Combat: explicit state machine

`setup -> turn1 -> handoff -> turn2 -> reveal -> vote -> verdict -> (setup next round | complete)`.

- Setup names default to **Andrew** and **Greyson**. Editable locally, no user accounts.
- The same photo ID or situation is fixed for both players in a round.
- After player 1 taps `Done`, show an opaque handoff screen. Do not reveal their draft in thumbnails, recent-meme list, navigation backstack or the next player's editor.
- Player 2 gets a separate draft and identical challenge. A timer is optional and can be disabled without penalty.
- Reveal presents **two independently rendered memes** side by side *within the game UI*, but exports remain separate PNG objects.
- Player(s) manually choose a winner or tie; Dexter's random verdict is decorative, not a hidden scoring algorithm.
- Track best-of-three and support restart. A reload mid-match resumes safely without exposing player 1's concealed draft or corrupting score.
- Accessibility: no timers that silently delete drafts; keyboard/touch voting, reduced-motion verdict and clear `Who is playing?` labels.

## 11. Portable pack contract

Use `.dexpack.json` for v1, because it can be produced with built-in browser APIs and carries app data without a ZIP dependency. Pack `schemaVersion: 1`, `exportedAt`, `appVersion`, `manifest` (counts + bytes), `photos` with base64-encoded binary and validated media type, `drafts`, `captions` (custom only), `settings` and optionally `combatSessions`.

- Explicitly warn that the backup contains private photos in readable form. It is **not** encryption or cloud sync.
- Preview photo count and estimated size before export; warn about large mobile packs and optionally exclude originals or history.
- Parse as **untrusted data**: maximum size and count, schema validation, MIME and decoded image checks, field length caps, no script/HTML execution, reject unknown required schema versions.
- Preview import conflicts and choose `Merge` (deduplicate by stable IDs and content hash where feasible) or `Replace` (confirmation required; transactional). A failed pack must leave existing vault unmodified.
- Round-trip requirement: export and import preserves photo count, tags, captions, crops and drafts for a representative 20-photo test set, byte-for-byte original photos when `Include originals` is selected.

Future ZIP support may replace the transport, but it must retain an explicit schema version and pass the same tests.

## 12. PWA/static hosting

The service worker caches only the exact versioned app shell and bundled non-sensitive assets. Do not cache user photos at public URLs or let an update erase IndexedDB contents. The app should show a reliable `Ready offline` signal only after precache success and control is established; otherwise report `Preparing offline` or `Offline unavailable`.

GitHub Pages is a possible distribution method, **not yet deployed**. Ensure paths resolve under `/DEX_Meme_Factory/`. A plain local server is sufficient for development. Add a desktop HTML fallback only after checking file-origin behavior; local storage is a convenience, not the backup.

## 13. Security, privacy and accessibility

- No credentials, cookies for accounts, trackers, analytics, network fetches in the core runtime, or implicit image uploads.
- Treat captions, filenames and imported pack fields as untrusted text. Use `textContent`/canvas drawing, never `innerHTML` from user content.
- Clear user intent for any replace/reset/import action; confirmation and safe recovery.
- Visible focus styles, proper labels, keyboard controls, touch targets at least comfortably finger-sized, high-contrast caption defaults and reduced-motion parity.
- No flashing, mandatory sound or time pressure. All humor modes usable without audio.
- Document limits and unsupported browser behaviors in the UI, not merely hidden console logs.

## 14. Validation plan

**Pure tests:** catalog schema, 120+ captions and 6 tones, exclusions, nonrepeat last 10, deterministic RNG, lock behavior, coherent compatibility, safe fallback, combat state transitions, scoring, import validation and non-destructive failures.

**Browser tests:**

1. Empty vault -> meaningful import path; no fabricated Dexter art.
2. Import 20 photos; tag, favorite, reload; all persist if storage is supported.
3. Judge ten times, no exact duplicate where eligible; `Treats` exclusion survives reload.
4. Situation Machine lock one field; reroll all; locked value unchanged and premise alters joke output.
5. Export a 1080x1080 PNG; verify one independent decoded image, caption readable and nonblank.
6. Offline after cache-ready: refresh/reopen and perform Judge, Situation, edit, export and Combat with all network disabled.
7. Play a best-of-three with two concealed turns; verify no premature reveal, correct winner and scoreboard.
8. Export a 20-photo pack; import in a fresh browser profile/device; compare records and original media bytes.
9. Import malformed/oversized pack; vault remains unchanged and error is actionable.
10. Keyboard-only and mobile viewport traversal with reduced motion; no unusable controls or loss of draft.

Test target at least **Mac Safari**, **Mac Chromium-family browser (Chrome or Brave)** and **Android Chromium-family browser**; add iOS Safari if available. If a target is not actually tested, label it `UNVERIFIED`, not PASS. Test online-first cache, app restart offline and storage loss as separate scenarios.

## 15. Definition of done and implementation handoff

A milestone is complete only with source files, a successful clean build/open, tests or documented manual proof, plus a Git commit identifying the actual change. The **planning docs alone do not satisfy a product milestone**.

The v1 release is ready only if each of the four modes works, photo vault remains local, 20-photo backup/restore succeeds, Meme Combat completes offline, final PNGs match preview, and the offline/responsive matrix passes. Failing checks remain explicit blockers rather than being downgraded to polish.

**First coding task:** Implement `app/` shell, local data model and photo import/gallery with temporary in-memory fallback and clear persistence detection. Add tests for 20 photos, failed decode and storage recovery. Stop after a demonstrated original-photo -> local gallery -> reload/restore path. Do **not** add AI calls, deploy or publish private assets as part of that task.
