---
name: design
description: Squadron design guidelines, for the app and for extension panels and settings. Load before any mockup, artifact, html block, extension panel page or style change in a Squadron repo.
---

# Squadron design

Design has two contracts. First find out which one applies:

- `src/app.css` and `DESIGN.md` at the root: **the Squadron app**.
- `extension.json` at the root: **an extension**.

## In the app

1. Read `DESIGN.md` in full, at least "Rules", "Mockups and artifacts" and the section of the component being touched, then the `:root` of `src/app.css`. `app.css` is authoritative for values, `DESIGN.md` says what they are for.
2. Start from the CSS of the closest existing component (file cited in "Components"). Reuse `StatusIcon`, `ModeIcon`, `Icon`, `Avatar`, `SidePanel`, `Dialog`, `Confirm`, `Select` and the `.btn*`, `.field`, `.option-*`, `.effect-*` classes without redefining them.
3. A token, hard-coded color, size or visible component added or changed: `DESIGN.md` is updated in the same commit. `scripts/design.test.ts` fails otherwise.

Mistakes already made, not to repeat:

- 8px radius on a card: it is 6px. 8px = dialogs and conversation cards with a tone bar, 4px = buttons and fields.
- `--background` taken for a background: it is the blue of background tasks. Ink on copper is `--accent-ink`.
- Copper on an ordinary action: copper signals what is waiting for the user. The main button is light (`.btn-primary`, `--text` on `--ink`, 700).
- Status ring redrawn by hand instead of `StatusIcon`.
- Outline as a `border`: it is `box-shadow: inset 0 0 0 1px …`, except zone separators.

## In an extension

- **Squadron zones** (items of `contributes.items`, declared settings): no freedom. The extension gives an `en` + `fr` label, an icon from the guide's `icon` list and an action; Squadron draws. No CSS, no color.
- **Its pages** (panels, settings page): free HTML/CSS/JS in an isolated iframe, but only these variables are guaranteed by the API: `--bg-main`, `--surface-1`, `--surface-2`, `--surface-3`, `--border`, `--border-focus`, `--text`, `--text-strong`, `--text-muted`, `--text-dim`, `--ink`, `--accent`, `--accent-link`, `--ok-text`, `--err-text`, `--sans`, `--mono`. The app's other tokens (`--wait-*`, `--option*`, `--surface-4`, `--menu`…) and its classes do not exist there.
- Start from `<link rel="stylesheet" href="/_squadron/kit.css" />` (background, text, links, headings, `button`, `button.primary`, fields, `.muted`) and style only what is missing, with the guaranteed variables.
- Still follow the spirit of the "Rules" of Squadron's `DESIGN.md`: copper (`--accent`) reserved for what is waiting for the user, light main button, 6/8/4px radii, outlines as `inset box-shadow`. Read Squadron's `DESIGN.md` at https://raw.githubusercontent.com/Jgrasp/squadron/main/DESIGN.md (WebFetch).
- No network, `localStorage` or popup (`alert`, `confirm`) in the page: state lives in the WASM.
- `contributes.theme` recolors Squadron's own variables: reserved for a theme extension, never for an extension's color.

## Mockups

- **```html block in the conversation**: gets the app's tokens and fonts, not its classes; no network or script. Copy the class rules from `app.css`.
- **Artifact** (`$SQUADRON_ARTIFACTS`): `<link rel="stylesheet" href="/__squadron/theme.css">` gives all of `app.css`. Do not copy the tokens. For an extension panel, use only the guaranteed variables above.
- Pitfall: app CSS loaded after `/__squadron/theme.css` (a copy of `app.css` rules, a dump of the dev app's stylesheets) brings back `html, body { height: 100%; overflow: hidden }` and the page no longer scrolls. Load nothing of the app after theme.css, or end with `html, body { height: auto !important; overflow: auto !important }`.
- At real size (conversation at 740px, widths from "Layout" in `DESIGN.md`).
- Variants stacked one under the other on the same page, the current state first, each titled, same data, each at real width.
- Interactive: the controls of the mockup work (segments, switches, menus, fields, buttons that move or reveal things) with a small inline script in the artifact, so Quentin can try each variant. Html blocks in the conversation have no script: use an artifact as soon as a mockup needs interaction.
- Pitfall: interactive means every control in every variant, the current state included. Never disable clicks (`pointer-events: none`) on part of the mockup. Before showing it, scroll to the bottom and click one control per variant.
- Mockup text in English.
- Show, wait for the choice, redo. Integrate or commit nothing before an explicit order.
