# 🌸 Sakura for Claude — v1.0.0

A Chrome extension that reskins claude.ai with a sakura theme: pixel-art
Mt. Fuji background, falling petals, a draggable sun/moon day/night toggle,
and a settings popup. Built as the first theme in a series (Vikings, Cloudy
Sky, etc. are stubbed out and ready to plug in later).

## Install it (unpacked, for testing / your own use)

1. Unzip this folder somewhere permanent (don't delete it after loading —
   Chrome loads unpacked extensions directly from the folder).
2. Go to `chrome://extensions`.
3. Turn on **Developer mode** (top-right toggle).
4. Click **Load unpacked** and select the `sakura-extension` folder.
5. Open **claude.ai** — the theme applies automatically. Click the
   extension icon in your toolbar to open the settings popup.

`Alt+Shift+S` flips day/night from anywhere on the page.

## What's actually in the box

```
manifest.json          Manifest V3 config
src/
  theme-engine.js       defaults, background layer, mode/class switching
  petals.js             canvas particle system for falling petals
  toggle.js             draggable sun/moon orb
  content.js            bootstrap — wires the three together, syncs storage
styles/
  inject.css            all injected CSS (background, petals, toggle, tint)
popup/
  popup.html            settings UI (markup)
  popup.css             settings UI (style)
  popup.js              settings UI (logic) — reads/writes chrome.storage.sync
assets/
  backgrounds/          sakura-day.webp / sakura-night.webp (your reference art)
  icons/                16/32/48/128px pixel-sakura icon
```

Settings live in `chrome.storage.sync` under the key `sakuraSettings`, so
they follow the user across signed-in Chrome installs. The popup and the
content script both read/write that same key and stay in sync live
(`storage.onChanged`) — toggle a slider in the popup and it updates the
open claude.ai tab immediately, no reload.

## What's guaranteed to work everywhere

The background image, the falling petals, the floating toggle, and the
whole-page colour wash are all **new elements the extension adds on top of
the page** — they don't depend on claude.ai's internal markup at all, so
they'll keep working even if Claude changes its UI.

## What's "best effort": making chat panels see-through

To let the background actually show *through* Claude's own sidebar and
message bubbles (rather than just tinting on top of them), `inject.css`
targets a handful of selectors that reflect Claude's UI as of this build:
`.font-user-message`, `.font-claude-message`, `[data-testid="chat-input"]`,
`nav`, `aside`. These are exactly the kind of thing that can change without
notice on any web app — that's true of *any* extension that restyles a
site it doesn't control, not something specific to this build.

**If a Claude update stops matching:**
1. Right-click a message bubble on claude.ai → **Inspect**.
2. Note the class name(s) on the bubble container.
3. Open `styles/inject.css`, find the `see-through panels` block near the
   top, and add the new class to the selector list.
4. Reload the extension at `chrome://extensions` (the little refresh icon).

Nothing else in the extension needs to change when this happens.

## Adding a new theme later (Vikings, Cloudy Sky, ...)

Right now there's only one theme, so it's wired directly into
`theme-engine.js`/`inject.css` rather than a separate plugin system —
that was the right call for one theme, but isn't the shape you want once
you're selling several. To add the second theme:

1. Duplicate the day/night background art into `assets/backgrounds/` with
   a new prefix (e.g. `vikings-day.webp`).
2. In `popup/popup.js`, flip that theme's `locked: false` in the `THEMES`
   array once its assets exist.
3. In `theme-engine.js`, generalize `BG_URLS` into a lookup keyed by
   `settings.theme` instead of the hardcoded `day`/`night` pair, and add
   the new theme's palette (petal hue, wash gradient colours) as its own
   block in `inject.css`, gated on `[data-sakura-theme="vikings"]` the same
   way `.sakura-ct-day`/`.sakura-ct-night` gate the sakura palette now.
   That's the one piece of real refactoring — everything else (petals
   engine, toggle, popup shell) is already theme-agnostic.

## Known limitations to be upfront about

- Pixel heading font (`Press Start 2P`) is loaded via `@font-face` from
  Google Fonts; if claude.ai's Content-Security-Policy blocks external
  fonts it'll silently fall back to a monospace stack instead — still
  readable, just not pixel-styled.
- This hasn't been tested against a live claude.ai session by me (no
  browser access in this environment) — the panel-recolouring selectors
  are a best-effort guess based on commonly-referenced Claude.ai DOM hooks,
  not a live-verified list. Background/petals/toggle needed no such
  guessing and should just work.
- Not yet packaged for the Chrome Web Store (that needs a developer
  account, a privacy-practices disclosure since it reads claude.ai pages,
  and store-listing assets) — this is an unpacked/developer-mode build.

## Handoff prompt (paste this to continue with another AI)

> I have a Manifest V3 Chrome extension called "Sakura for Claude" that
> reskins claude.ai with a sakura theme. Architecture: `theme-engine.js`
> (background layer + day/night class switching via CSS custom
> properties), `petals.js` (canvas-based falling petal particle system,
> density/speed configurable), `toggle.js` (draggable sun/moon orb that
> flips day/night), `content.js` (bootstrap, reads/writes
> `chrome.storage.sync` key `sakuraSettings`, listens for
> `storage.onChanged` and runtime messages `sakura:burst`/`sakura:flip`),
> and a `popup/` (html/css/js) settings panel with theme cards (only
> "Sakura" unlocked so far — Vikings and Cloudy Sky are stubbed for
> later), day/night/auto segmented control, and sliders for petal
> density/speed and background dim/blur/panel-solidity. All injected
> styling lives in `styles/inject.css`. Chat-panel see-through recolouring
> uses best-effort selectors (`.font-user-message`, `.font-claude-message`,
> `[data-testid="chat-input"]`, `nav`, `aside`) that may need updating if
> claude.ai's DOM changes — instructions for that are in the README.
> Next steps I want help with: [describe what you need — e.g. "verify the
> panel selectors against the live DOM and fix any that don't match",
> "build the Vikings theme following the README's theming guide", "package
> for the Chrome Web Store"].
