# Dexter Meme Factory - Screen-by-Screen Implementation Plan

**Status:** Design and interaction specification, not an implemented UI\
**Primary devices:** Mac desktop browser; responsive Android/iPhone-class mobile browser\
**Named players:** Andrew and Greyson only\
**Design direction:** dark editorial comedy studio; dense personality, spare controls, very little friction

## 0. Product navigation contract

The app consists of seven navigable surfaces and one pass-and-play subflow. The **primary CTA is always a way to make a meme**. Private photos, joke authoring and storage are supporting features, not the first thing the user must configure.

```text
HOME
  -> JUDGMENT -> EDITOR -> EXPORT -> HOME
  -> SITUATION -> JUDGMENT/EDITOR -> EXPORT
  -> COMBAT SETUP -> PLAYER 1 -> HANDOFF -> PLAYER 2
                    -> REVEAL -> VOTE -> VERDICT -> NEXT/END
  -> PHOTO VAULT -> (SELECT PHOTO -> previous activity)
  -> SETTINGS/BACKUP -> EXPORT PACK or RESTORE PACK
```

**State carried between screens:** selected photo, tone, current situation and draft ID; these persist in a normal editor flow. The combat flow owns a separate, sealed per-player draft scope. A browser back action must not reveal a hidden combat turn.

**Shared UX language:** `Make one`, `Judge this`, `Set the scene`, `Challenge Greyson`, `Edit`, `Reroll`, `Save PNG`, `Back up`, `Restore`. Do not substitute decorative, ambiguous labels for critical actions.

### Shared frame

- **Mobile header:** Dexter Meme Factory title, compact `Offline/Online/Preparing` indicator, menu button. No huge masthead consuming the editing viewport.
- **Desktop:** narrow navigation rail; single central content workspace; contextual settings panel only on screens that need it.
- **Mobile:** bottom navigation or easily reachable home/back controls, sticky primary action where needed; one-column vertical arrangement.
- **UI visuals:** charcoal/ink foundation, warm parchment accents, restrained brass highlights and one rich comic accent for primary actions. Avoid generic neon dashboards, excessive glowing paws or busy decorative borders.
- **Humor:** dry, varied, specific. Text needs to fit in a meme, not a three-paragraph dialogue box.
- **Motion:** optional short reveal/press feedback, never mandatory and never replacing textual status; respect reduced motion.

---

## 1. Home: `/` (first interaction)

**Primary question:** What can I make immediately?

**Hierarchy:**

1. Huge `JUDGE THIS` button with one-line explanation: `Dexter has an opinion. You might survive it.`
2. Two large secondary cards: `Set the scene` and `Meme Combat`.
3. Mini recent-meme strip (only when users actually have saved drafts).
4. `Dexter photos` and `Back up / Restore` accessible without a hunt.

**Default:** if photos are imported, show one authentic photo preview; no auto-generating on load. **Empty:** show a plain instruction: `Dexter needs a real photo. Import one to begin.` with one import control. It is acceptable to explore joke examples without a photo, but do not falsely label a generic dog as Dexter or allow a fake final export.

**Click results:**
- `JUDGE THIS` -> one draft using an existing photo or the first-photo prompt.
- `Set the scene` -> Situation screen.
- `Meme Combat` -> Combat setup.
- `Recent` item -> editor, except drafts concealed within an active combat round.

**Acceptance:** returning home never wipes a draft; accessible from keyboard and tap, action labels remain visible in offline mode.

## 2. Photo Vault: `/photos`

**Purpose:** protect authentic Dexter source material and make photo selection quick.

**Components:** local import button; grid of actual imported photos; expression-tag chips; favorite filter; per-photo details drawer; `Use in current meme` action. Show count and storage health, not surveillance-style activity analytics.

**Per-photo controls:** `Use`, `Favorite`, `Tags`, `Rename`, `Delete`. Deletion requires confirmation if the photo is referenced by a draft and explains the consequences. Do not remove an original merely because the user changed a meme crop.

**Photo tagging:** manual selection from `judgmental`, `suspicious`, `concerned`, `royal`, `sleepy`, `delighted`, `unimpressed`, `confused`, `heroic`, `chaotic`; optional custom tags. Auto-pick ranks matching tags, then favorite, then fair rotation. User can override without asking permission.

**Empty:** `No Dexter photos here yet.` plus `Import photos`. **Invalid input:** show filename and reason; valid files in the same batch still import. **Storage denied/volatile:** show `Photos may not persist here; export a pack before closing` and offer download. **20-photo grid:** fast scrolling and thumbnail rendering, no full-resolution blobs decoded for every thumbnail simultaneously.

**Acceptance:** 20 usable photos survive reload in supported stable-origin storage; their original bytes and tags survive backup round-trip.

## 3. Judgment Button: `/judge`

**Purpose:** instant, actually usable meme generation.

**Layout:** large preview of chosen Dexter photo; `JUDGE THIS` action; small tone picker; caption preview; unobtrusive `Reroll joke`, `Different photo`, `Edit` and `Save PNG` actions. Do not present all six tones in a massive controls wall.

**Interaction:** click Judge -> Comedy Brain chooses photo/joke pairing -> preview appears quickly -> user edits or exports. The selected tone and topic exclusions persist. `Reroll joke` must keep the chosen photo unless `Shuffle photo` is selected.

**Empty:** prompt to import a photo; users may preview text-only jokes, but no deceptive faux-Dexter output. **No eligible captions:** `Those filters ruled out every joke. Unban a topic or try another tone.` Provide `Review filters`. **Saving error:** keep draft and explain fallback.

**Required punchline quality:** each tone has distinct structure, not merely different labels. For example:

- Deadpan: `THE PLAN WAS REVIEWED. / THE PLAN WAS RETURNED.`
- Petty: `I HAVE BEEN VERY PATIENT. / THAT WAS NOT A COMPLIMENT.`
- Royal: `THE REQUEST IS DENIED. / APPEALS MAY BE SUBMITTED TO THE FLOOR.`
- Concerned Elder: `YOU LOOK CONFIDENT. / THAT IS WHAT WORRIES ME.`
- Cosmic Witness: `THE UNIVERSE IS VAST. / AND YET YOU CHOSE THIS.`
- Full Stinkweasel: `I COULD EXPLAIN THE PROBLEM. / BUT THEN WE WOULD BOTH BE TIRED.`

Examples are seeds for editorial direction; the 120-line catalog still needs substantive writing and review.

**Acceptance:** a real photo + readable joke + one-click PNG path, offline; last ten exact captions do not repeat when enough are eligible.

## 4. Situation Machine: `/situation`

**Purpose:** inspire different kinds of jokes without requiring user prompt engineering.

**Three generator slots:** `Dexter's job`, `Where this goes wrong`, `What happened`. Each displays chosen value, `Reroll` and `Lock` buttons. A single `Surprise me` rerolls all **unlocked** slots. Provide `Make a meme from this` and `Use in a duel` actions.

**Example card:**

> **Role:** night museum curator\
> **Location:** abandoned submarine\
> **Problem:** an unauthorized goose insists it is management

The selected premise is fed to the comedy engine as structured data; it changes wording/theme and not just hidden metadata. Show a small `Premise used in caption` status or highlight which parts matter. Don't infer the actual photo depicts a submarine: this is a comedic fictional situation superimposed on Dexter's real photo.

**Invalid lock state:** if all allowable combinations are incompatible, show `These three choices do not belong in the same universe. Unlock one.` with a specific `Unlock problem` action. Never silently discard the lock.

**Acceptance:** each slot rerolls independently, locked values survive, combining parts remains coherent, output meaningfully changes after a premise change.

## 5. Meme Editor: `/editor/:draftId` (logical route)

**Purpose:** turn a draft into a deliberate, standalone meme with almost no learning curve.

**Desktop geometry:** canvas on left or center; caption text controls in right panel. **Mobile:** canvas above bottom sheet, controls below; touch dragging must not conflict with page scroll. Keep Save PNG visible without obscuring the picture.

**Controls v1:**

- Edit top text and bottom text; add one optional free text line.
- Three typography treatments: classic high-contrast meme, understated editorial, dramatic announcement.
- Resize text with accessible slider and numeric/readable value; align left/center/right.
- Drag text within bounds; keyboard arrow nudge when focused.
- Pan and crop photo with `Fit Dexter` preset; restore original crop anytime.
- Undo/redo within draft, `Reset captions` (confirm if edits will be lost), `Save PNG`.

**Default:** text appears in safe upper/lower regions. No automatic heading repeating Dexter's name. Avoid facial obscuration, especially ears, eyes and muzzle; offer a visible warning when text overlays the photo's central face area if feasible. Preserve the source photograph.

**States:** exporting (button disabled and accessible status), export success, export failed (re-enable without losing draft), image missing (relink/reimport photo), disk/download denied (show `Try Share` if supported or another browser-specific save method), resize/rotation change (canvas and controls remain aligned).

**Acceptance:** the exact text, positions and crop from preview appear in one 1080x1080 PNG; no panel grid, watermark, extra output or blank canvas. Text remains editable after reopening saved draft.

## 6. Meme Combat: `/combat` (multi-screen state flow)

**Purpose:** an actual game for two people who are physically sharing one device.

### 6A. Setup

Controls: player labels **Andrew** and **Greyson**, `Same photo` or `Same situation`, optional timer toggle, `Start best of three`. Default to names but let users edit locally. Use no accounts, logins, matchmaking or third participants.

### 6B. Player 1 edit

Show challenge and clear `Andrew's turn`. Use ordinary editor controls, but write to private draft A not the global recent list. `Finish turn` takes user to a sealed handoff overlay.

### 6C. Handoff

Full opaque screen: `Pass the device to Greyson`. `I'm Greyson - begin` starts player 2. No photo thumbnail with Andrew's caption, no recent-draft shortcuts, no back navigation that reveals player 1's content. On refresh, resume at a safe handoff checkpoint.

### 6D. Player 2 edit

Same locked challenge/photo, independently edited draft B. `Finish turn` ends writing and unlocks reveal.

### 6E. Reveal and vote

Show both separate meme previews with labels `Andrew` and `Greyson` and actions: `Andrew wins`, `Greyson wins`, `Tie`. Side-by-side comparison is acceptable **inside the app**; downloading a meme still exports each separately. Tie triggers a replayed round without awarding a win, with a clear next action.

### 6F. Verdict and score

Dexter delivers a short, randomly selected **decorative** judgment after the human vote. Show `Andrew 1 - Greyson 0`, `Next round` or `Finish`; first to two wins best-of-three. Don't let Dexter's joke overwrite the actual vote. End screen gives both individual PNG downloads and `Play again`.

**Game recovery:** if storage fails, keep current drafts in memory and offer export before restart; no silently vanished turn. Timer expiration never discards work, and timer defaults off. Avoid requiring a third-party real-time connection.

**Acceptance:** complete a best-of-three offline with no hidden-turn leakage, correct tie/replay and scoreboard, separately exportable memes and safe reload points.

## 7. Settings + Backups: `/settings`

**Purpose:** preserve work, respect privacy and make the offline nature obvious.

**First section:** `Your factory stays on this device.` Show storage support (`Local storage active`, `Temporary session`, `Storage limited`), approximate photo count and offline-ready status. Do not display `Private and encrypted` unless encryption exists.

**Actions:** `Back up my factory`, `Restore a pack`, `Exclude topics`, `Default tone`, `Manage custom captions`, `Reset local data`. Backups use `.dexpack.json` and disclose that photos inside are readable; Greyson receives them only through intentional private sharing. `Reset` requires typed/clear confirmation and warns to back up first.

**Import flow:** pick pack -> parse and validate in memory -> preview counts/compatibility and merge-vs-replace -> confirm -> transactional apply -> verify result. On parse/size/schema failure, show action-specific reason and leave existing collection unchanged. Never execute imported code or silently overwrite originals.

**Accept:** export/import of 20 normal photos and relevant draft metadata round-trips; failed import does not mutate local data; excluded topics survive restart.

---

## 8. Navigation and state map

| From | Action | Destination | Preserve / guard |
| --- | --- | --- | --- |
| Home | Judge | Judgment | Tone, photo selection |
| Home | Set scene | Situation | Existing locks only if resuming scene |
| Home | Combat | Setup | Do not import private combat drafts into recents |
| Judgment | Edit | Editor | Exact joke ID and chosen photo |
| Situation | Make meme | Judgment / Editor | Coherent situation and its filters |
| Situation | Use in duel | Combat Setup | Shared challenge frozen when round begins |
| Editor | Save PNG | Same screen | Draft retained after success/failure |
| Editor | Photo picker | Vault -> Editor | Text remains unchanged |
| Combat Turn1 | Finish | Handoff | Protect draft A |
| Combat Handoff | Confirm player | Turn2 | Draft A not inspectable via UI |
| Combat Turn2 | Finish | Reveal | Stop editing; show both only now |
| Combat Vote | Winner / tie | Verdict | Human selection controls scoring |
| Settings | Restore pack | Settings | Preview before mutation |

## 9. Responsive and accessible behavior

- **Phone portrait:** no canvas smaller than practical finger editing area; zoom/pan or edit panel collapse available. Sticky actions respect safe-area insets and do not obscure bottom caption.
- **Tablet/desktop:** controls and canvas side by side when there is space; no scattered controls in huge unused margins.
- **Keyboard:** all primary actions reachable and named; arrow movement for selected text, Escape to close overlays, predictable focus return after close.
- **Touch:** minimum comfortable targets, drag handles distinct from text input, no drag-only action.
- **Screen reader:** the finished meme needs an editable accessible text description made from its caption and photo label, not only raw canvas pixels.
- **Reduced motion:** reveal uses a static status and same action buttons without animations.
- **Contrast:** text and outlined captions legible on light/dark photos; use non-color cues for selection, offline and success states.
- **Error recovery:** show explicit `Retry`, `Restore`, `Use different photo` or `Export now` depending on failure; never reset the draft as a generic recovery step.

## 10. Screen acceptance checklist

- [ ] Home provides a successful `Judge` journey from an imported photo in no more than a few taps.
- [ ] Photo Vault imports 20 photos and never displays a substitute dog as Dexter.
- [ ] Judgment produces an editable, non-repeating caption with an obvious reroll.
- [ ] Situation locks/rerolls, forms coherent premises and passes them into the actual joke.
- [ ] Editor previews and exports one readable PNG with source photo unchanged.
- [ ] Combat hides turn 1 until reveal, completes best-of-three and keeps individual export files separate.
- [ ] Settings clearly explains local storage, export, import, loss risks and no encryption claim.
- [ ] All primary screens work with the network disabled once cache is ready.
- [ ] Browser back, reload, orientation switch, storage error, oversized image and malformed pack have sensible outcomes.
- [ ] Keyboard, mobile touch and reduced-motion flows have been checked on actual supported browsers.

## 11. Implementation order

Use the milestone tasks in [PROJECT_KICKOFF.md](PROJECT_KICKOFF.md). Build shared Photo Vault and storage first, Judgment and editor second, Situation and catalog third, Combat fourth, then offline/install QA. **Do not implement a gallery, a game and an AI backend simultaneously.** Use [DEVELOPER_HANDOFF.md](DEVELOPER_HANDOFF.md) for data types, safety and validation details.
