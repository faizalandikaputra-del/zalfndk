# zalfndk

A single-page website for After Effects editors. Visitors land on a home page and pick a menu entry below it:

- **Extension**: a virtual After Effects host. Drop an extension `.zip` and its panel opens at its manifest size in the browser. CSInterface and `__adobe_cep__` get mock answers, and ExtendScript calls run against a mock composition. No After Effects needed.
- **Editing pack**: opens the editing pack folder on Google Drive in a new tab.

## Files

- `index.html`: the whole site (HTML, CSS and JS in one file).
- `assets/jszip.min.js`: JSZip 3.10.1, self-hosted.
- `assets/fonts/`: JetBrains Mono (latin 400, 500, 700, woff2), self-hosted.
- `assets/LICENSE-jszip.txt`, `assets/fonts/OFL.txt`: licenses for the two bundled libraries.
- `favicon.svg`, `404.html`, `robots.txt`: site basics. The 404 page links to `/`, so it assumes the site lives at the domain root.
- `_headers` (Netlify, Cloudflare Pages) and `vercel.json` (Vercel): security and cache headers.
- `.nojekyll`: tells GitHub Pages to serve the files as they are.
- `README.md`: this file.

## Deploy

It is a static site. No build step, no server code, no environment variables. Upload the contents of this folder (so `index.html` sits at the root of the site).

- **Netlify:** open app.netlify.com/drop and drag the folder in. Or connect a Git repo, leave the build command empty and set the publish directory to `/`.
- **Cloudflare Pages:** create a project, choose Direct Upload and upload the folder. Or connect a Git repo with no build command and output directory `/`.
- **Vercel:** run `vercel` inside the folder, or import a Git repo. Framework preset: Other. No build command.
- **GitHub Pages:** push the files to a repository, then Settings > Pages > Deploy from a branch > `main` and `/ (root)`. All asset paths are relative, so a project site such as `user.github.io/repo/` works too (only the 404 page link needs the domain root).
- **Any other static host (nginx, Apache, S3):** copy the folder and serve `index.html`.

After deploying, check: the access gate shows, **> EXECUTE / ENTER** starts the music, the Extension view loads a zip (use **Load demo panel** for a quick test), and **Editing pack** opens Google Drive. Use HTTPS (every host above does by default).

No third-party requests are made by the site itself. The only outside link is the Google Drive folder, which opens in a new tab.

## Notes

- Extension panels run inside a same-origin sandboxed iframe so the mock CEP bridge can talk to them. Only load zips you trust.
- No Content-Security-Policy header is set, because the site is a single file with inline CSS and JS and runs extension code from `srcdoc` frames. Tighten it only if you move the code into separate files.
- To change the Drive link, search `index.html` for `drive.google.com` (it appears in the HUD bar and the editing pack card).

## Rules for any change

0. **Keep the site self-contained.** Fonts and libraries stay in `assets/`. Do not add CDN links, Google Fonts or other third-party requests.
1. **All website text must be in English.** This covers headings, menu labels, buttons, placeholders, console messages, errors and the page `<title>`. The page uses `<html lang="en">`. Do not add other languages.
2. **Keep the UI style below.** Strictly monochrome (greys on near-black). No color accents, gradients or rounded-card kits.
3. Keep the entry gate: the site opens on a boot screen and visitors must click **> execute** to enter. Do not auto-skip it.
4. Keep the home page first, with the menu under it. Every new tool becomes a new menu entry.
5. Keep the layout responsive down to phone width, keep visible keyboard focus, and respect `prefers-reduced-motion`.
6. Audio stays chill (lofi only), starts only after the visitor clicks **> EXECUTE / ENTER**, and keeps the synthesized approach below. Default music volume is 35%.
7. **No native (browser-default) UI.** Every control must use the themed components listed under Custom controls. Do not add plain `<select>`, checkbox, radio, date picker, `alert`, `confirm`, `prompt` or `title=` tooltips. Build a themed version instead.

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

## HUD look

The home page follows an eDEX-UI style HUD, inspired by the structure of leylin.love (no content, images or audio were copied).

- **Access gate:** panel with a system checklist (`[v] SYSTEM CHECK OK` rows), a user/role/status block, and the **> EXECUTE / ENTER** button. After the click: `ACCESS GRANTED / STARTING SYSTEM...`.
- **HUD bar:** `> zalfndk` brand, uppercase nav (Home, Extension, Editing pack with a `> ` marker on hover) and a live local clock with GMT offset.
- **Hero:** `// HELLO, WORLD.` label, a typed three-line headline with a block cursor, and a terminal window that shows a `neofetch` block and then types `host.jsx` snippets.
- **Modules:** section title `// MODULES 02 OBJECTS`. Each card has a small uppercase status row (`01 / extension`, `STATUS READY`), a title like `Extension // Browser host`, a description, tag chips and an `Open / ...` action.
- **Footer:** `cava --mono` style bar visualizer with 40 bands (static under reduced motion).
- **Micro labels:** 10.5px, uppercase, letter-spaced, muted grey. Use them for status text only.

## Limits

- The host is a mock. Real ExtendScript APIs are simulated, so results are not those of a real project.
- Binary After Effects files such as `.aep` are not rendered. Only CEP extension panels run.
- The Google Drive folder cannot be embedded, so the menu entry is a link.
