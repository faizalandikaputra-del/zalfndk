# zalfndk

> **AI assistants and developers: read this whole file before changing anything.** It is the handoff document for this project. Keep it up to date with every change (see "Working rules").

## Quick facts

- **What it is:** a static website for After Effects editors. A home page with a menu leads to two things: (1) a virtual After Effects host that runs a CEP extension `.zip` in the browser, and (2) a link to the owner's editing pack on Google Drive.
- **Name:** `zalfndk`. Never rename it. The site zip must always be called **`zalfndk.zip`** and must contain one top-level folder called **`zalfndk/`** with `index.html` inside it. The owner's update script depends on both names.
- **Version:** 1.3.1 (2026-10-11).
- **Hosting:** GitHub Pages, deployed from a Git repository the owner controls. No build step, no backend, no environment variables.
- **Owner:** speaks Indonesian and talks to assistants in Indonesian. Likes short, direct replies. **All text inside the website itself must be English.** This README and `PANDUAN.md` in the deploy kit are the only Indonesian-friendly docs.
- **Style:** monochrome terminal HUD (near-black, greys, JetBrains Mono), inspired by the structure of leylin.love and built on the owner's earlier "panelview" site. See "UI style" and "HUD look".
- **Self-contained:** fonts and JSZip live in `assets/`. The only outside link is the Google Drive folder.

## Working rules (read before editing)

0. **Keep the site self-contained.** Fonts and libraries stay in `assets/`. Do not add CDN links, Google Fonts or other third-party requests.
1. **All website text must be in English.** Headings, menu labels, buttons, placeholders, console messages, errors and the page `<title>`. The page uses `<html lang="en">`. Do not add other languages.
2. **Keep the UI style below.** Strictly monochrome (greys on near-black). No color accents, gradients or rounded-card kits.
3. **Keep the entry gate.** The site opens on a boot screen and visitors must click **EXECUTE / ENTER** to enter. Do not auto-skip it, and do not start audio before that click.
4. **Keep the home page first, with the menu under it.** Every new tool becomes a new menu entry.
5. **Keep the layout responsive** down to phone width, keep visible keyboard focus, and respect `prefers-reduced-motion`.
6. **Audio stays chill** (lofi only), starts only after the execute click, and stays synthesized (see "Audio"). Default music volume is 35%.
7. **No native (browser-default) UI.** Use the themed components under "Custom controls". Do not add plain `<select>`, checkbox, radio, date picker, `alert`, `confirm`, `prompt` or `title=` tooltips. Build a themed version instead.
8. **Do not rename the zip or its top folder.** Output is always `zalfndk.zip` containing `zalfndk/`.
9. **Update this README in the same change** (Quick facts, file map, changelog, anything that became untrue). Bump the version.
10. Do not copy content, images or audio from other websites. Taking structure or mood as inspiration is fine.
11. **Every button and link gets an icon** from the inline sprite (see "Motion and icons"). A new button needs a new or existing `#i-*` symbol. Keep icons monochrome (`currentColor`).
12. **Motion follows the tokens in "Motion and icons".** Use the shared easing and durations, never `linear` for movement, and keep the `prefers-reduced-motion` fallbacks working.

## How to make a change and ship it

1. Edit `index.html` (and files in `assets/` if needed).
2. Run the checks:
   - JavaScript syntax: extract the last inline `<script>` and run `new Function(code)` in Node.
   - Every `src`, `href` and `url(...)` that points to a local file exists in the zip.
   - No `http(s)://` links other than the Google Drive folder (search for `cdnjs`, `googleapis`, `gstatic` to be sure).
   - All visible text is English. No native controls were added.
3. Update this README: version, changelog, any changed facts.
4. Build the zip with a top-level `zalfndk/` folder and the same file name, for example:

```python
import zipfile, os
out = "zalfndk.zip"
if os.path.exists(out): os.remove(out)
with zipfile.ZipFile(out, "w", zipfile.ZIP_DEFLATED) as z:
    for root, _, names in os.walk("zalfndk"):      # folder that holds index.html, assets/, README.md ...
        for n in names:
            p = os.path.join(root, n)
            z.write(p, p)                          # keeps the zalfndk/ prefix
```

5. Hand the zip to the owner. The owner puts it in the deploy kit folder (replacing the old `zalfndk.zip`) and runs `update-web.bat` (Windows) or `update-web.sh` (Mac/Linux). See "Deploy" below.

## File map

- `index.html`: the whole site (HTML, CSS and JS in one file, about 65 KB).
- `assets/jszip.min.js`: JSZip 3.10.1, self-hosted.
- `assets/fonts/`: JetBrains Mono (latin 400, 500, 700, woff2), self-hosted.
- `assets/LICENSE-jszip.txt`, `assets/fonts/OFL.txt`: licenses for the two bundled libraries.
- `favicon.svg`, `404.html`, `robots.txt`: site basics. The 404 page links to `/`, so it assumes the site lives at the domain root.
- `_headers` (Netlify, Cloudflare Pages) and `vercel.json` (Vercel): security and cache headers. GitHub Pages ignores them.
- `.nojekyll`: tells GitHub Pages to serve the files as they are. Keep it.
- `README.md`: this file.

The owner also has a separate **deploy kit** (not part of this zip): `update-web.bat`, `update-web.sh`, `pages.yml`, `PANDUAN.md` (Indonesian guide) and a copy of `zalfndk.zip`.

## How `index.html` is organized

Page structure, in document order: `#rain` (matrix canvas), the SVG icon sprite, `#boot` (access gate), `.pl` (audio widget), `#home` (home view: HUD bar, hero, modules, workflow, capabilities, footer), `#ext` (extension view, hidden until opened), then `assets/jszip.min.js` and one inline script.

Views switch with `go('home' | 'ext')`, which plays a short exit and enter transition (skipped under reduced motion). The body gets class `work` in the extension view, which dims the rain.

The inline script, in order:

1. **Mock After Effects model:** `mk()` builds Proxy objects that log every call. `app`, `comp`, `layer`, plus `setupHost()` which injects `app`, `$`, `Folder`, `File`, `system` and the AE classes into a hidden host iframe. `hostEval()` runs ExtendScript text there.
2. **Bridge `window.__pv`:** `eval`, `env` (CEP host environment and theme colors), `err`, `on`/`emit` (events), `frame` (nested frames), `open`/`close` (floating windows), `dlg` (themed alert/confirm/prompt toast).
3. **Floating windows:** `openExt(id)` implements `requestOpenExtension` as draggable, resizable windows.
4. **`SHIM_T`:** the script injected into every panel iframe. It defines `__adobe_cep__`, `cep`, a small `require` shim, the CSInterface-facing calls and the alert/confirm/prompt overrides.
5. **Zip to panel:** `res`, `css`, `buildHtml` (inlines CSS, JS and images from the zip into `srcdoc`), `buildPanel`, `start`.
6. **Loading:** `loadZip` reads `CSXS/manifest.xml` (bundle id, name, version, host, `MainPath`, `ScriptPath`, size, multiple `Extension` entries), falls back to `index.html` when there is no manifest, then evaluates every `.jsx` in the hidden host. `pick` handles the file input and drag and drop. `demo` builds a demo zip in memory.
7. **Panel UI:** `apply`/`setSize` (size, auto fit, zoom), resize grip, theme toggle (Dark/Light panels), reload, and the console input `#cmd`.
8. **Gate and views:** `leave` (execute click), `go`.
9. **Matrix rain** (grey glyphs on `#rain`).
10. **Home animations:** `SN` snippets, `hl` highlighter, `typeLoop` (typing in the terminal window), `heroType` (headline), `clk` (clock), the 40-band visualizer.
11. **Audio:** `ac`, `tone`, `noise`, `kick`, `p8`, `sched`, `playM`, `stopM`, `sfx`, `ui`. Preferences are stored in `localStorage` under `zf-music` and `zf-sfx`.
12. **Themed controls:** `toast`, range fill (`fill`, `fillAll`), number steppers.
13. **Icons and reveal:** `ic(name)` returns an icon `<svg>`; an `IntersectionObserver` adds `.in` to `.sr` elements as they scroll into view; a `pointermove` listener feeds `--mx`/`--my` to the card glow.

Matrix rain and the visualizer run on `requestAnimationFrame` (no `setInterval`) and pause when the tab is hidden.

Important element ids: `#boot #exec #bootmsg`, `#home #nExt #toExt #hl #code #cstat #modules #clk #viz`, `#ext #toHome #demoBtn #drop #file #tools #iw #ih #fit #tDark #tLight #reload #size #sliders #rw #rh #stage #dock #panel #grip #info #log #cmd`, `.pl #pst #plb #vol #sfxb`, `#rain`. Scripts look elements up by these ids, so keep them when you edit the markup.

The Google Drive link appears twice (HUD bar and editing pack card). Search `index.html` for `drive.google.com` to change it.

## UI style

Minimal monochrome terminal look, all in one monospace family.

- **Font:** JetBrains Mono (400, 500, 700), 13px body, line-height 1.6. Headings use weight 700 and tight letter-spacing (-0.03em). Home title is large (up to 112px).
- **Dark theme (default):** background `#101010`, text `#e9e9e9`, muted text `#8b8b8b`, lines `#272727`, card `#171717`, error `#ff8a80`.
- **Light theme:** background `#f3f3f3`, text `#161616`, muted `#666`, lines `#d4d4d4`, card `#e9e9e9`, error `#b3261e`.
- **Shapes:** 1px solid borders, radius 5px on buttons and 6px on panels and menu cards, no heavy shadows. Drop zone uses a dashed border.
- **Motion:** easing `cubic-bezier(.2,0,0,1)`, short transitions (about 150ms) on hover and press.
  - Boot gate: terminal lines appear one by one, then the **> execute** button. Nothing happens until it is clicked. Then the screen fades up and the home page enters.
  - Home entrance: title, intro, code window, menu title and the two menu cards rise 16px and fade in, staggered 70ms apart (about 450ms in total).
  - Code window: `host.jsx` types ExtendScript snippets character by character with a blinking caret, pauses, then moves to the next snippet in a loop.
  - Matrix rain: a fixed canvas behind the whole site, grey glyphs at low opacity (30% on the home page, 12% inside the Extension view so the panel stays readable). It pauses when the tab is hidden.
  - Reduced motion: rain and typing stop (the first snippet shows as static text) and transitions shrink to near zero.
- **Layout:** header bar with `> zalfndk`, then content with a 340px side column (extension info and host console) on wide screens. It collapses to one column below 900px. The stage behind the panel has a subtle dot grid.
- **Voice:** short, plain, sentence case. Buttons say what they do ("Open extension", "Open Google Drive", "Load demo panel").

## HUD look

The home page follows an eDEX-UI style HUD, inspired by the structure of leylin.love (no content, images or audio were copied).

- **Access gate:** panel with a system checklist (`[v] SYSTEM CHECK OK` rows), a user/role/status block, and the **EXECUTE / ENTER** button. After the click: `ACCESS GRANTED / STARTING SYSTEM...`.
- **HUD bar:** `> zalfndk` brand, uppercase nav (Home, Extension, Editing pack with a `> ` marker on hover) and a live local clock with GMT offset.
- **Hero:** `// HELLO, WORLD.` label, a typed three-line headline with a block cursor, and a terminal window that shows a `neofetch` block and then types `host.jsx` snippets.
- **Modules:** section title `// MODULES 02 OBJECTS`. Each card has a small uppercase status row (`01 / extension`, `STATUS READY`), a title like `Extension // Browser host`, a description, tag chips and an `Open / ...` action.
- **Footer:** `cava --mono` style bar visualizer with 40 bands (static under reduced motion).
- **Micro labels:** 10.5px, uppercase, letter-spaced, muted grey. Use them for status text only.

## Custom controls

The page was audited for browser-default UI. Everything below is themed to match the monochrome HUD.

- **Sliders** (music volume, panel width and height): flat 2px track that fills with the light color, small square thumb that inverts on hover and focus. Styled for both WebKit and Firefox.
- **Number fields** (panel W and H): native spinners hidden, centered text, with themed `-` and `+` buttons (step 10). Arrow keys still work.
- **Scrollbars:** thin, square, dark grey thumb on a transparent track (page, host console, stage, side panes).
- **Text selection:** inverted colors (light background, dark text). Caret and placeholder colors follow the palette.
- **Tooltips:** native `title` tooltips removed. Controls use `aria-label` for screen readers.
- **Dialogs:** `alert`, `confirm` and `prompt` calls from the panel or from ExtendScript show a themed toast in the top right corner (labeled `ALERT / HOST` and so on) and are also written to the host console. A toast does not block, so `confirm` returns true and `prompt` returns its default value.
- **Page frame:** `color-scheme: dark`, no tap highlight on touch devices.
- **Cannot be themed:** the operating system file picker that opens from the zip drop zone. The drop zone itself is themed, and dragging a file onto it avoids the picker.

## Audio

All sound is generated in the browser with the Web Audio API. There are no audio files, no external audio hosts and no licensing to track.

- **Music:** a looping lofi track at 76 BPM: soft maj7 and min7 chords (F, Em, Dm, C), a low bass, a gentle kick, a muted snare, hats, a sparse pentatonic melody and vinyl crackle, all through a low-pass filter. It starts after the execute click and can be paused.
- **Player widget (bottom right):** status text, Play/Pause, a volume slider (default 35%) and an SFX on/off switch. Choices are remembered in `localStorage`.
- **SFX:** soft UI click on buttons and links, a faint hover blip (mouse only), key ticks while the headline types, an access-granted chime on execute and when a zip loads, and a sweep when switching between Home and Extension.
- Browsers block sound until a user gesture, which is why the gate button starts the audio.
- To use a real song instead, embed a track you own or have a license for (as a data URI or an asset), keep it under the 16 MB page limit, and keep the same player controls.

## Motion and icons

**Motion personality:** corporate and precise. Short, decisive, no bounce.

- **Easing:** one signature curve, `cubic-bezier(.2,0,0,1)` (`--ease`). Exits use the accelerating curve `cubic-bezier(.3,0,1,1)`.
- **Durations:** `--t1` 150ms (hover, press, color), `--t2` 300ms (glow, small state changes), `--t3` 450ms (entrances). View exit is 180ms, view enter 350ms.
- **Entrance:** elements rise 16px and fade in. Home hero items stagger 50ms apart after the execute click (`.rise`, variable `--i`). Sections below the fold reveal once on scroll (`.sr` plus `.in`, variable `--d` at 60ms steps). Stagger totals stay under 500ms.
- **View change:** home and Extension cross-fade with a small vertical shift (`.vout`, `.vin`).
- **Hover and press:** menu cards lift 2px and show a soft pointer-following glow; the icon tile inverts; the action arrow slides 3px. Buttons darken or invert in 150ms and scale to 0.97 on press. HUD nav links get a 1px underline that slides in.
- **Ambient:** grey matrix rain (60fps canvas, one glyph per cell, per-column speeds), a 40-band visualizer that eases toward its targets, blinking caret, and a 3-bar equalizer in the audio widget while music plays.
- **Smooth scrolling:** `scroll-behavior: smooth` for anchor links.
- **Reduced motion:** rain and typing stop, reveals and entrances show instantly, smooth scrolling is off.

**Icons:** one inline SVG sprite at the top of `<body>`. Each icon is a `<symbol id="i-name" viewBox="0 0 24 24">`, drawn with 1.6px strokes, square caps, mitered joins, no fill, `currentColor`. Use it like `<svg class="ic" aria-hidden="true"><use href="#i-name"/></svg>`. In JavaScript use `ic('name')`. Current names: term, home, box, folder, down, right, ext, play, pause, vol, zap, zapoff, wave, upload, dock, land, win, fit, max, moon, sun, reload, plus, minus, x, info, plug, code, layers.

- Buttons and links are inline-flex with a 7px gap between icon and label.
- The menu cards and workflow/capability cells use a 34px bordered icon tile (`.cico`).
- Icon assignments: Home = home, Extension = box, Editing pack = folder, Load demo = play, Dock/Landscape/Floating = dock/land/win, Auto fit = fit, Dark/Light = moon/sun, Reload = reload, Play/Pause = play/pause, SFX = zap/zapoff, Volume = vol, drop zone = upload, console and terminal = term, external link = ext.

## Deploy

It is a static site. Upload the contents of this folder so `index.html` sits at the root of the site.

- **GitHub Pages (what the owner uses):** Settings > Pages > Deploy from a branch > `main` and `/ (root)`. Every push to `main` redeploys automatically in 1 to 2 minutes. Asset paths are relative, so project sites such as `user.github.io/repo/` work too.
- **Netlify:** drag the folder to app.netlify.com/drop. **Cloudflare Pages:** Direct Upload. **Vercel:** run `vercel` in the folder. No build command anywhere.
- **Custom domain:** only set one if the owner actually owns a domain. The value must be a full domain with an ending, for example `example.com`. A bare word such as `zalfndkk` makes GitHub show "custom domain is not properly formatted". If that error appears, clear the Custom domain field in Settings > Pages and delete the `CNAME` file from the repo.

**Owner's update flow:** replace `zalfndk.zip` in the deploy kit folder, then run `update-web.bat` or `update-web.sh`. The script unzips into a local copy of the repo, mirrors the zip contents into it (files missing from the zip are deleted; `.git`, `.github` and `CNAME` are kept), commits and pushes. That is why the zip name `zalfndk.zip` and the top folder `zalfndk/` must not change. After a deploy, hard refresh with Ctrl+Shift+R.

Quick check after any deploy: the gate shows, **EXECUTE / ENTER** starts the music, **Load demo panel** works in the Extension view, **Editing pack** opens Google Drive.

## Changelog

- **1.3.1 (2026-10-11):** browser history. Switching between Home and Extension now pushes a history entry (`#home`, `#ext`), so the mouse back button, back swipe and Alt+Left return to the previous view. `popstate` calls `go(v,true)` (no new entry). The Home link in the HUD bar no longer adds duplicate entries.
- **1.3.0 (2026-10-11):** bottom tab bar (`#tabbar`, class `.dk`) with Home, Extension and Editing pack. Auto-hides after 3s without interaction and slides up from the bottom on any pointer, touch, key, wheel or scroll event. A small handle at the bottom edge (`#dkh`) shows it again. Hovering or keyboard-focusing the bar keeps it open. A sliding highlight marks the active tab. The audio widget moves up while the bar is visible. Appears only after the entrance click. Note: `#dock` is the extension panel frame, do not reuse that id. Taps inside the extension iframe do not reach the page, so use the bottom handle there.
- **1.2.0 (2026-10-11):** haptic feedback on touch devices. Every button, link and toast fires a short vibration through the same `sfx()` cues (`click`, `tick`, `sweep`, `granted`). Android uses `navigator.vibrate`; iOS 18+ uses a hidden `<input type=checkbox switch>` toggled by a label click. A `HAPTIC ON/OFF` button in the audio widget (shown only on touch devices that support it) saves to `localStorage` key `zf-hap`. Desktop is unchanged.
- **1.1.0 (2026-10-11):** inline icon sprite and an icon on every button, link and card; new Workflow (3 steps) and Capabilities (6 items) sections so the home page is fuller; smoother motion (view transitions, scroll reveal, pointer glow on cards, sliding nav underline, equalizer in the audio widget, rain and visualizer on `requestAnimationFrame`, smooth scrolling); the gate button now shows an icon instead of the `>` character.
- **1.0.0 (2026-10-11):** current release. Home page with menu (Extension, Editing pack), virtual After Effects host built from the owner's panelview code, access gate with execute button, matrix rain in grey, typed code terminal and neofetch block, HUD style inspired by leylin.love, synthesized lofi music and SFX with a player widget, themed controls (no native UI), self-hosted assets, deploy files and update scripts, handoff README.

## Limits

- The host is a mock. Real ExtendScript APIs are simulated, so results are not those of a real project.
- Binary After Effects files such as `.aep` are not rendered. Only CEP extension panels run.
- The Google Drive folder cannot be embedded, so the menu entry is a link.

## Notes

- Extension panels run inside a same-origin sandboxed iframe so the mock CEP bridge can talk to them. Only load zips you trust.
- No Content-Security-Policy header is set, because the site is a single file with inline CSS and JS and runs extension code from `srcdoc` frames. Tighten it only if you move the code into separate files.
- To change the Drive link, search `index.html` for `drive.google.com` (it appears in the HUD bar and the editing pack card).

