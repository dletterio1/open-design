# SESSION-RESUME — read this first (written 2026-07-02)

The previous session (1e8d08a5) died from context overflow ("Prompt is too long")
mid-execution. This file is the recovered handoff. Everything below was verified
against disk, not memory.

## Operational rules that killed the last session — obey these

1. **Short, single-purpose sessions.** Tooling work and screen work never mix.
   2–3 screens max per session.
2. **Never read raw captures or vendor CSS into context.** Read `spec.json`
   (~5KB), `CHANGES.md`, `brand-spec.md`. Grep the 1.09MB
   `booking/theme/vendor/daisyui-5.6.7.css` — never open it.
3. **Always use absolute paths under this project root.** The 6 failed
   dashboard edits last session were caused by a cwd drift into
   `booking/theme/vendor/`.

## What this project is

Export-to-parity pipeline: the deployed staging apps (Amplify, then Sonik
Booking) exported into standalone, editable, theme-switchable HTML prototypes.
Captures are reference truth (never pasted markup); colors come from scripted
extraction (`scripts/extract-tokens.py`); data comes from harvest
(`scripts/distill.py` → `spec.json`); only composition/interaction are
hand-built. Svelte is the eventual destination; every screen exposes a
`window.__sonikAgentUI` contract.

**Amplify lane: DONE and exported** — 8 token-pure screens, unified `index.html`
demo, committed to the Amplify repo at `src/design-system/static-preview/` on
branch `ux/static-preview`, **PR #491 (open, unmerged)** with CI gates
(`static-preview-gates` workflow, `scripts/check-gates.py`).

**Booking lane: IN PROGRESS** — this is what you are resuming.

## Hard constraints (user-imposed, non-negotiable)

- **Theme-agnostic:** gunmetal-dark is *a* theme, never *the* design. Screens
  bind only to DaisyUI semantic vars (`--color-*`), surface projection
  (`--app-*`), radius/font vars. Zero color literals outside `theme/`
  (= Gate 0). All six registry themes must keep working.
- **`theme/tokens.css` is generated — never hand-edit.** `providers.css` is the
  only hand-maintained color file. `amplify-static-preview-dashboard.html` is a
  render artifact — never edit.
- **BK ledger protocol** (`booking/CHANGES.md`): every deviation from captured
  parity gets a `BK-###` entry with status
  (`proposed → applied → verified/vetoed`) and the dev requirement it implies.
  No silent changes, including your own layout fixes. User vetoes by ID; parity
  content is restored verbatim. Entries append, never rewrite.
- **Cascade order is load-bearing:** vendor daisyui.css loads BEFORE
  tokens.css/effects.css so extracted themes win. Flipping the order silently
  breaks theming.
- Two-lane commits: `chore(design-system)` = generated, `feat(design-system)` =
  hand-authored. Raw `rendered-dom.html`/`content.md` never committed.

## Exact resume point — the A→D shell-upgrade plan (user approved: "execute")

Verbatim from the dead session:

> - **A. Vendor drop** — `booking/theme/vendor/daisyui.css`, pinned version,
>   pulled via analyze-copy-retrofit's manifest-first flow (source preference:
>   booking's own deployed bundle, fallback to the published release). Ledgered.
> - **B. Shell rebuild** — `booking/screens/_shell.{css,js}` on exact DaisyUI
>   structures: `navbar` for the top bar, vertical `menu` (with `menu-title`
>   groups) for the sidebar, `tabs-box` for the context strip, and the megamenu
>   pattern reserved for the super-app top-nav.
> - **C. Dashboard refit + deslop pass** — move the dashboard onto the new
>   shell, then run ai-slop-cleaner discipline over the result, one smell per
>   pass, each cut ledgered.
> - **D. Verify** — gates, render check, before/after shots into the ledger.

Status:

| Step | Status | Notes |
|---|---|---|
| A | **Done on disk, NOT ledgered** | `booking/theme/vendor/daisyui-5.6.7.css` + `MANIFEST.md`. Deployed-bundle URLs 404'd post-redeploy → pinned jsdelivr 5.6.7 fallback (documented in MANIFEST). First action of the new session: ledger this as BK-015. |
| B | Not started | No `booking/screens/_shell.*` exist yet. |
| C | Not started | `booking/screens/dashboard.html` is still the pre-refit version (inline shell, links only tokens.css — no vendor CSS). The 6 attempted edits never landed. |
| D | Not started | |

Side task also approved: `uipro init --ai claude` (UI-UX Pro Max skill).
**This completed** — `uipro` 2.10.0 is installed and the skill exists at
`.claude/skills/ui-ux-pro-max/` in this project. Do not reinstall.

## Booking state (all verified on disk)

- `booking/theme/` — tokens.css / effects.css / theme.js extracted (six themes,
  gunmetal-dark default); fonts in `booking/assets/fonts/`.
- `booking/captures/dashboard/` + `booking/captures/settings/` — harvested once,
  banked ("never go back to the well"). Known nit: settings `spec.json` is
  sparse (content sits outside `<main>`; one-line `distill.py` fix pending).
- `booking/screens/dashboard.html` — upgraded Operations Home: story carousel
  over the 11 real contexts (6 Ultratest test contexts excluded per BK-006),
  Tee Sheet live numbers verbatim, token-pure, `__sonikAgentUI` instrumented.
- `booking/CHANGES.md` — ledger BK-001…BK-014, all `applied`, awaiting user
  veto/verify. Component decomposition queued (start `ContextStatTile`, then
  `ContextHeroStory`, per sonik-component-design contract).

## Queued after A→D (do not start unopposed; confirm order with user)

1. User veto/verify pass over BK-001…BK-014.
2. Component phase (`ContextStatTile` first), ledgered under new BK IDs.
3. `distill.py` `<main>` fix; booking settings re-distill (from banked capture).
4. Amplify: merge PR #491, make gates required; `campaigns.html` shell
   retrofit; Gate-2 VRT; Storybook scaffolder.
5. Decide booking commit destination (sonik-booking-service repo vs handoff
   package) — undecided, user's call.
6. Before any marketplace/super-app work: read `~/Downloads/Amplify Design
   System/HANDOFF.md` + `_ds_manifest.json` (one small read), and at most ONE
   PRD, only in the session building that surface.

## Cleanup owed

- Stray `.omc/` state dirs (tool noise from the cwd drift): one inside
  `booking/theme/vendor/`, one at `src/design-system/static-preview/` in the
  Amplify repo working tree on `main` (that path's canonical copy lives on
  branch `ux/static-preview`; don't commit it from `main`).

## Provenance

Formalized by Open Design from candidate fb14916a-8407-4371-aca9-beba8273c66e.
