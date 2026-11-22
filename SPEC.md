# misc_utils_webapp: specification

This document describes the app in enough detail to rebuild it from scratch, or something very close to it. It always describes the current version. Whenever `index.html` changes, this file changes with it (see the rules in `CLAUDE.md`).

## 1. Overview

- A personal collection of everyday utilities in one web app, with one utility per page. Pages are switched with tabs.
- The whole app is a single static file, `index.html`, with all HTML, CSS and JavaScript inline.
- There is no backend, database, user login or build step, and no external dependencies (no frameworks, CDNs or web fonts). The app works offline, opened directly from disk (`file://`) or served from any static host.
- Nothing is sent anywhere. A page may remember its own settings in the browser's `localStorage` (see 3.4). Nothing else is stored.
- Target browsers: current Chrome, Safari, Firefox and Edge, on desktop and mobile. Plain modern JavaScript (ES2015+) is fine.

## 2. Structure of `index.html`

In order:

1. `<head>`: `charset` utf-8, viewport `width=device-width, initial-scale=1`, an empty `<title></title>` that the shared script fills in, and one `<style>` block with two parts: shared styles, then page-specific styles.
2. `<header class="app-header">` containing a `.container` with an empty `<nav class="tabs" id="tabs" aria-label="Pages">` that the shared script fills. The app has no name, so the header shows only the tabs.
3. `<main class="container">` containing the pages, one `<section class="page">` each.
4. The shared navigation `<script>`.
5. Page scripts, one `<script>` element per page that needs one.

## 3. Page system

### 3.1 Page markup

Each page is a `<section>` that is a direct child of `<main>`:

```html
<section class="page" id="unit-converter" data-title="Unit converter">
  <h1>Unit converter</h1>
  ...
</section>
```

- `id`: a slug of lowercase letters, digits and dashes, unique in the file. It is the page's address in the URL.
- `data-title`: the tab label, also used as the browser tab title. If it is missing, the id is used instead.
- The content starts with an `<h1>` holding the page name. Since the app has no name of its own, this is the page's main heading.
- Tabs appear in the same order as the sections in the file. The first section is the default page.

### 3.2 Navigation

The shared script, wrapped in an IIFE, runs at the end of `<body>`:

1. It collects every `main > section.page`. If there are none, it stops.
2. For each page, it appends `<a href="#/<id>">`, with the page's title as its text, to `<nav id="tabs">`.
3. It shows the page named in the URL, and does so again on every `hashchange` event (tab clicks, back/forward, a typed URL).

Showing a page:
- The id is read from the hash by stripping a leading `#` or `#/`. Both `#/page-2` and `#page-2` work, but links always use `#/<id>`.
- If there is no hash, or no page has that id, the first page is shown. The URL is not rewritten.
- The shown section gets the class `is-active`. CSS hides every other page with `.page:not(.is-active) { display: none !important; }`. The `!important` stops a page's own `display` rule from un-hiding it.
- The matching tab link gets `aria-current="page"`, and the attribute is removed from the other links.
- The document title (the label on the browser tab) becomes the page's title, with nothing added to it. One exception: while a Work Timer countdown runs, the Work Timer script replaces the title with the time left (see 5.1).

Links use `#/<id>` rather than `#<id>` because a fragment that matches an element id makes the browser scroll to that element. With the `/`, no element matches.

### 3.3 Adding a page

1. Copy an existing `<section class="page">` block and put it where its tab should appear.
2. Give it a new `id` and `data-title`, and update its `<h1>`.
3. Add any page-specific CSS in the "Page-specific styles" part of the `<style>` block, with every rule scoped under the page's id (for example `#unit-converter .result { ... }`).
4. Add any page-specific JavaScript in a new `<script>` element after the shared script, wrapped in `(() => { ... })();`. This keeps its variables private, and an error in it can't break other pages or the tab bar.
5. Start the page's own ids and classes with a short prefix (the Work Timer uses `wt-`), so they can't clash with another page's.
6. Document the page in section 5 of this file and in `README.md`.

The shared script needs no changes.

### 3.4 Saved settings

- A page may remember its settings in `localStorage`, under the key `misc-utils:<page id>`, as one JSON object.
- Every read and write is wrapped in `try`/`catch`. Missing, unreadable or out-of-range values fall back to the page's defaults, and if storage is unavailable the page simply works with its defaults.
- Nothing else is stored: no cookies, IndexedDB, files or accounts. Clearing the browser's site data returns every page to its defaults.

## 4. Visual style

- Font: `system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`, line height 1.5. No web fonts.
- Light and dark themes follow the operating system setting through `prefers-color-scheme`. `:root` sets `color-scheme: light dark` so built-in form controls match the theme.
- Colors are CSS custom properties on `:root`, overridden inside a `prefers-color-scheme: dark` media query:

| Variable | Light | Dark | Used for |
|---|---|---|---|
| `--bg` | `#f6f7f9` | `#14171c` | Page background |
| `--surface` | `#ffffff` | `#1d2128` | Header and page cards |
| `--text` | `#1d2330` | `#e6e9ee` | Body text |
| `--muted` | `#5f6b7a` | `#9aa4b2` | Secondary text, placeholders |
| `--border` | `#dde1e7` | `#2e343d` | Borders |
| `--accent` | `#2f6fde` | `#5b8ef0` | Active tab, hover borders |
| `--accent-text` | `#ffffff` | `#0d1117` | Text on accent backgrounds |

- A page may add its own color variables on its section, scoped like the rest of its styles, with dark values in its own `prefers-color-scheme: dark` media query. The Work Timer's timer colors are an example (5.1).
- `--radius: 10px` for card corners. `box-sizing: border-box` on all elements. `body` has no margin.
- `.container`: max width 960px, centered, 16px padding left and right. Used by the header's inner wrapper and by `<main>`.
- Header: `--surface` background with a 1px `--border` bottom border. It holds only the tab bar, a wrapping flex row with 8px gaps and 12px padding above and below.
- Tabs: pill-shaped links with 6px × 14px padding, a 1px `--border` border, 999px radius, `--text` color, no underline, and no line breaks inside a label. On hover, the border turns `--accent`. The active tab has an `--accent` background and border, and `--accent-text` text.
- `<main>`: 24px padding above and 48px below.
- Page card: `--surface` background, 1px `--border` border, `--radius` corners, 20px padding. The page's `<h1>` is 1.5rem, with no top margin and 20px below.
- `.placeholder`: text in `--muted`, used for "nothing here yet" notes.

## 5. Pages

| Tab | id | Purpose | Content |
|---|---|---|---|
| Work Timer | `work-timer` | Work and rest countdowns, with random reminder dings during work | See 5.1 |
| Page 2 | `page-2` | Not decided yet | Placeholder: `<h1>Page 2</h1>` and a `.placeholder` paragraph, "Nothing here yet." |
| Page 3 | `page-3` | Not decided yet | Same placeholder, titled Page 3 |

### 5.1 Work Timer

A countdown for work and one for rest on a single page, each with its own color: blue for work and green for rest. The time sits in the middle of a ring whose arc runs down as the session goes. While the work timer runs, a soft ding plays at random moments as a reminder to stay on task. On a computer, the timer can also pop out into a small floating window that stays on top of other windows. The page's ids and classes start with `wt-`. Its styles sit under `/* ---------- Work Timer (#work-timer) ---------- */` in the page-specific styles, and its script is the first page script.

#### Markup

Inside `<section class="page" id="work-timer" data-title="Work Timer">`, in order:

1. `<h1>Work Timer</h1>`.
2. `button#wt-float.wt-icon-btn.wt-float` with `aria-pressed="false"`, `aria-label="Floating window"` and `title="Floating window"`, starting `hidden`. It holds a 20px inline SVG on a 24×24 viewBox: a window outline (`rect` at x 3, y 5, 18 × 14, corner radius 2, no fill, 2px `currentColor` stroke) with a small filled window in its lower right (`rect` at x 12, y 11, 6 × 5, corner radius 1, `currentColor` fill).
3. `button#wt-gear.wt-icon-btn.wt-gear` with `aria-label="Settings"`, `title="Settings"`, `aria-expanded="false"` and `aria-controls="wt-settings"`. It holds a 20px inline SVG gear filled with `currentColor`, drawn on a 24×24 viewBox: eight teeth with one pointing straight up, tips on a circle of radius 10.5, roots on radius 7.9, each tooth 0.4 of its 45° pitch wide at the tip and 0.62 at the root (arcs along the tip and root circles), and a round hole of radius 3.2 cut out with `fill-rule="evenodd"`.
4. `div.wt-body`, holding:
   - `div.wt-main`, the timer column:
     - `div.wt-tabs` with `role="tablist"` and `aria-label="Timers"`, holding two `role="tab"` buttons, `#wt-tab-work` "Work" and `#wt-tab-rest` "Rest", both with `aria-controls="wt-panel"`.
     - `div#wt-panel.wt-panel` with `role="tabpanel"`, holding:
       - `div#wt-ring.wt-ring`, the ring (see Ring below), holding an inline `svg.wt-ring-svg` with `viewBox="0 0 100 100"` and `aria-hidden="true"`, then `div.wt-ring-text`. The SVG holds `circle.wt-ring-track` (center 50, 50, radius 45) and then `path#wt-ring-arc.wt-ring-arc` with `d="M50 5a45 45 0 1 1 0 90a45 45 0 1 1 0-90z"`: the same circle drawn as two half-circle arcs, starting at the top and going clockwise. `div.wt-ring-text` holds `div#wt-time.wt-time` (`role="timer"`) and `div#wt-ring-label.wt-ring-label`, which the script fills with the name of the timer on show ("Work" or "Rest").
       - `p#wt-info.wt-info`.
       - `div.wt-controls` with three buttons: `#wt-start.wt-btn.wt-btn-primary` "Start", `#wt-pause.wt-btn` "Pause" and `#wt-reset.wt-btn` "Reset".
     - `button#wt-wake.wt-wake` "Keep screen on", with `aria-pressed="false"`, starting `hidden`.
     - `p#wt-wake-note.wt-wake-note` (`role="status"`), empty unless keeping the screen on failed.
     - `p#wt-live.wt-sr` (`role="status"`), a visually hidden live region for announcements.
   - `div#wt-away.wt-away`, starting `hidden`, shown in the timer's place while the timer is in the floating window: `<p>The timer is in a floating window.</p>` and `button#wt-back.wt-back` "Bring it back".
   - `div#wt-backdrop.wt-backdrop`, starting `hidden`.
   - `aside#wt-settings.wt-settings` with `aria-labelledby="wt-settings-title"`, starting `hidden`:
     - `div.wt-settings-head`: `h2#wt-settings-title` "Settings" and `button#wt-close.wt-icon-btn.wt-close` (`aria-label` and `title` "Hide settings") holding a 16px × made of two 2px round-capped strokes.
     - Three `div.wt-group` elements with `role="group"`, each labelled by its `h3`:
       - "Durations": two `label.wt-field` rows, "Work" and "Rest". Each wraps a `span.wt-num` holding a number input followed by " min". Inputs `#wt-work-min` and `#wt-rest-min` have `min="0.5" max="999" step="0.5" inputmode="decimal"`.
       - "Random reminder": `label.wt-check` with the checkbox `#wt-remind-on` and the text "Ding at random times during work", then `label.wt-field` rows "Shortest gap" (`#wt-gap-min`) and "Longest gap" (`#wt-gap-max`), built the same way with `min="0.5" max="60" step="0.5"`.
       - "Sounds": three `div.wt-sound` rows, "Reminder ding", "Session start" and "Session end". Each ends with a `button.wt-icon-btn.wt-play` whose `data-sound` is `reminder`, `start` or `end`, with `title="Play"`, an `aria-label` such as "Play the reminder ding", and a 14px play triangle (`M9 5.5v13l10-6.5z`, filled).
     - `p.wt-hint`: "Settings are saved in this browser. Clearing the browser's site data brings back the defaults."

#### Timers

- There are two timers, Work and Rest, each with a duration from the settings (defaults 25 and 5 minutes). Each one is idle, running or paused.
- Time shows as `MM:SS`, rounded up to the whole second, so a fresh 25-minute timer shows 25:00 and reaches 00:00 only at the very end. Minutes can go past 59 (for example 90:00).
- Time is kept with the wall clock: a running timer stores the moment it will end (from `Date.now()`), so the display is right whenever it is redrawn, even after the page was in the background or the computer slept.
- **Start** begins a fresh session at the full duration, with the start sound. On a paused timer it reads **Resume** and carries on from where it paused, with no sound. **Pause** freezes the timer. **Reset** stops it and puts it back to its full duration, silently.
- Button states for the timer on show: Start is enabled unless that timer is running, Pause only while it runs, Reset only once it has started (running or paused).
- One timer at a time: pressing Start on one timer first resets the other, silently, if it is running or paused. Switching between the Work and Rest tabs only changes which timer is on show and never stops anything.
- When a session reaches zero, the end sound plays, that timer goes back to its full duration (idle), and the page shows the other timer, ready but not started. The info line then reads "Work session done. Start your rest when you’re ready." or "Rest is over. Start the next work session when you’re ready." until the next press of Start or Reset or a Work/Rest tab switch. The same text goes to the live region for screen readers.
- The info line (`#wt-info`) shows the first that applies: the end-of-session message; "Paused" if the timer on show is paused; on the Work tab, "Random reminder every 2–5 min" (a single number when both gaps are equal, for example "every 6 min") or "Random reminder off"; on the Rest tab, nothing. It keeps its height when empty.
- The tab of a running timer shows a small dot in that timer's color. The time is drawn in the color of the timer on show while running, `--muted` while paused and `--text` otherwise (see Timer colors).
- A changed duration shows at once on an idle timer. A timer that has started keeps its current session length, and the new duration applies from its next reset or new session. To do this, each timer stores the length of its current session (`total`). It is set to the duration when a fresh session starts, when the timer is reset or finishes, and when its duration changes while it is idle.
- **Browser tab title:** while a timer runs, the title is the time left and the timer's name, `18:42 · Work` or `04:12 · Rest`, updated along with the display, whichever app page is on show. When no timer is running, the title goes back to the shown page's title (its `data-title`, or its id). The page script also listens for `hashchange`. Its listener runs after the shared script's, so it puts the countdown back right after a page switch.

#### Ring

- The time sits in the middle of a ring, with the name of the timer on show ("Work" or "Rest") under it in smaller `--muted` text. The ring is a circle in a much paler shade of the timer's color (the track), with an arc in the solid color on top that shows how much of the session is left.
- The arc starts at the top (12 o'clock) and runs clockwise. Its length is the share of the session left: time left ÷ session length (`total`). So the ring is full while a timer is idle, and as the time runs down, the arc's end moves back counter-clockwise toward the top, with nothing left at zero. When a session ends, the page shows the other timer with a full ring.
- It always shows the timer on show, and is redrawn along with the time: on every tick and every render. Because it uses the session length, a duration changed mid-session doesn't make it jump.
- The arc is drawn with the path's `stroke-dasharray`: `none` when the share is 1, otherwise `<share × C> <C>` with 2 decimals, where C = 2π × 45 ≈ 282.74 (the circumference). A share of 0 draws nothing, since the stroke ends are flat (butt). The attribute is written only when it changes.
- Colors: the track is `--wt-color-pale` and the arc `--wt-color`, the colors of the timer on show (see Timer colors). While that timer is paused, the whole ring turns grey, matching the time: the track is `--border` and the arc `--muted`. The script sets the classes `is-running` and `is-paused` on `#wt-ring` for the timer on show, and the colors of the time and the ring follow from them.
- In the floating window the ring adapts to the window's size, and in a very short window it becomes a small ring beside the time (see Styles).

#### Timer colors

Each timer has its own color, so the two look distinct: a soft blue for Work and a soft green for Rest, never pure `#0000ff` or `#00ff00`. Each color also has a much paler shade, used for the ring's track. They are custom properties on `#work-timer`, overridden inside a `prefers-color-scheme: dark` media query like the shared colors:

| Variable | Light | Dark | Used for |
|---|---|---|---|
| `--wt-work` | `#2f6fde` | `#5b8ef0` | Work timer (the same blue as `--accent`) |
| `--wt-work-pale` | `#dae5f9` | `#2c3c5a` | Work timer's ring track |
| `--wt-rest` | `#318454` | `#53ae77` | Rest timer |
| `--wt-rest-pale` | `#dae9e0` | `#2b443c` | Rest timer's ring track |

- The green has about the same lightness as the blue (OKLCH lightness 0.55 against 0.56 in light, 0.68 against 0.66 in dark) but less chroma (0.11 to 0.12), which keeps it calm. Each pale shade is its solid color mixed into `--surface`, 18% in light and 25% in dark, so in the dark theme the track is a dim version of the color.
- Contrast: the Start button's `--accent-text` label is at least 4.6:1 on every timer color, and every timer color is at least 4.6:1 against `--surface`.
- `--wt-color` and `--wt-color-pale` hold the colors of the timer an element belongs to. `#work-timer` sets them to the Work colors, and `#wt-tab-rest` and `#wt-panel.is-rest` set them to the Rest colors. Every render toggles the class `is-rest` on `#wt-panel`, so it's there while the Rest timer is on show.
- They color the ring's arc and track, the time while running, the running dot on each tab (each in its own timer's color, whichever is on show), the background and border of the Start/Resume button (`.wt-btn-primary`), and the hover border of the timer buttons. The rest of the page keeps `--accent`: the gear and floating window buttons, the settings sidebar with its play buttons, Keep screen on, and the focus outline.

#### Random reminder

- It plays only while the work timer is running, and only when the setting is on (the default).
- Each gap between dings is a fresh random number, evenly spread between the shortest and longest gap (defaults 2 and 5 minutes). The first gap counts from the start of the session. The time to the next ding is never shown.
- Pause remembers how long was left until the next ding, and Resume continues from there. A fresh session draws a fresh first gap.
- No reminder plays in the last 15 seconds of a session, so it never runs into the end sound.
- Changing any reminder setting applies at once: the dings already scheduled are dropped and, if work is running, a fresh gap is drawn from now. If work is paused, the remembered gap is dropped and Resume draws a fresh one.

#### Sounds

The sounds are generated with the Web Audio API, with no sound files. One `AudioContext` is created by the first click that needs sound (Start, Resume or a play button), because browsers allow audio only after a user gesture. Everything plays through a master gain of 0.9.

Every note is a soft bell of four sine partials:

| Partial | Frequency | Peak (× note level) | Fade (× note fade time) |
|---|---|---|---|
| 1 | f | 1 | 1 |
| 2 | 2f | 0.4 | 0.6 |
| 3 | 3f | 0.18 | 0.4 |
| 4 | 4.16f | 0.08 | 0.25 |

Each partial's gain starts at 0, rises linearly to its peak in 5 ms, then falls exponentially to 0.0001 by the end of its fade. Its oscillator stops 50 ms after that.

| Sound | Notes: start, pitch, level, fade time | Plays when |
|---|---|---|
| Reminder | 0 s, A5 (880 Hz), 0.5, 2 s | A random reminder is due |
| Start | 0 s, G5 (783.99 Hz), 0.42, 1.2 s; then 0.16 s, D6 (1174.66 Hz), 0.42, 1.6 s | A fresh work or rest session starts |
| End | 0 s C6 (1046.5 Hz), 0.2 s G5 (783.99 Hz), 0.4 s E5 (659.25 Hz), 0.6 s C5 (523.25 Hz); all at level 0.45; fades 1.4, 1.4, 1.6 and 3 s | A session reaches zero |

So the reminder is one ding, the start is a short rising pair, and the end is a falling run of four notes, the longest and loudest of the three. Their peaks are about −4.6, −4.2 and −3.5 dBFS, well clear of clipping.

#### Keeping time in a background tab

Browsers slow down timers in background tabs (Chrome to once a minute after five minutes hidden), so the page doesn't rely on them for anything that must be on time:

- The end sound and the reminders are scheduled ahead on the audio clock, which keeps running in the background. When a session starts or resumes, the end sound is scheduled for the end time, and reminders are scheduled up to 5 minutes ahead. Every tick extends that window. Each scheduled sound has its own gain node: cancelling one (on Pause, Reset or a reminder setting change) disconnects that node and stops its oscillators. A finished sound disconnects its own gain node.
- A 250 ms `setInterval` redraws the display while a timer runs. Next to it runs a chain of silent one-second `ConstantSourceNode`s (offset 0, connected to the output). Each one's `ended` event calls the same tick and starts the next one, and these events aren't slowed down in background tabs. That keeps the title countdown current and ends the session on time in a background tab. Tick is safe to call any number of times.
- While the floating window is open, a second 250 ms interval runs on the floating window itself (that window's own `setInterval`) whenever a timer runs. The floating window stays on screen, so browsers don't slow its timers even while the app's tab is hidden or covered, and the floating display stays current to the second. Stopping the timer clears it, and it goes away with the window.
- On each tick, if the wall clock and the audio clock have drifted a second or more apart since the sounds were scheduled (the audio clock stalls while a computer sleeps), everything is scheduled again from the wall-clock times, keeping the next reminder's time. Reminders missed while asleep are skipped, not played in a burst.
- At the end of a session, if the scheduled end sound is still more than 0.5 s in the future (because the audio clock is behind), it is cancelled and, if audio is running, the end sound plays at once instead. Otherwise it is left to ring out.
- The audio context is suspended 5 seconds after a timer stops or a preview plays, if no timer is running then. The next Start, Resume or play button wakes it again.

#### Settings sidebar

- The gear opens and closes the sidebar. The close button, the backdrop (narrow screens) and Escape (while the page is shown) also close it. Opening moves focus to the close button, and closing returns it to the gear. While the sidebar is open, the gear has the accent background. The sidebar starts closed on every load.
- Wider than 720px, `div.wt-body` gets the class `is-settings-open` and becomes two columns, `minmax(0, 1fr) 260px`, with the sidebar beside the timer. At 720px or narrower, the sidebar is a panel fixed to the right edge of the window, `min(320px, 88vw)` wide and full height, scrolling if needed, with a `--surface` background, only a left border, no rounded corners and a shadow (`-8px 0 24px rgba(0, 0, 0, 0.25)`). It sits over a fixed full-window backdrop of `rgba(0, 0, 0, 0.4)`.
- Entries are checked on `change`. A number is rounded to the nearest 0.5 and clamped to its limits (0.5–999 minutes for durations, 0.5–60 for gaps). Anything that isn't a number puts the previous value back. Raising the shortest gap above the longest raises the longest to match, and lowering the longest below the shortest lowers the shortest to match. With the reminder off, the two gap inputs are disabled and their rows turn `--muted`.
- The play buttons play their sound at once, whatever the timers are doing.
- Every change is saved to `localStorage` under `misc-utils:work-timer`, as JSON like `{"workMin":25,"restMin":5,"remindOn":true,"gapMinMin":2,"gapMaxMin":5}`. On load, each of the four numbers is used only if it is a finite number, rounded and clamped as above, and `remindOn` only if it is a boolean. Anything else falls back to its default. If the shortest gap then ends up above the longest, both gaps go back to their defaults.

#### Keep screen on

- It uses the Screen Wake Lock API, and the button appears only if `navigator.wakeLock` exists.
- The button toggles (`aria-pressed`). On, it requests a screen wake lock. Off, it releases it. Browsers drop the lock whenever the page is hidden, so it is requested again when the page becomes visible while the button is on. If a request fails, the button turns off and `#wt-wake-note` says "This browser didn’t allow keeping the screen on."
- It is not saved: it is off on every load.

#### Floating window

On a computer, the timer can move into a small window that stays on top of other windows, so it can sit in a corner of the screen. Ordinary browser windows can't be made narrower than about 400–500 px, and a page can't change that, which is why this uses the Document Picture-in-Picture API.

- The button `#wt-float` appears only if `documentPictureInPicture` exists in `window`. It is a toggle: `aria-pressed="true"` while the floating window is open.
- **Opening:** the button calls `documentPictureInPicture.requestWindow({ width: 240, height: 120 })` from its click, since browsers allow this only after a user gesture. The button is disabled while the request is pending, and if the browser refuses, nothing changes. The browser decides where the window opens and may adjust its size. The user can move and resize it.
- The new window starts empty. The script copies every `<style>` element into its `<head>`, sets its title to the page's `data-title` ("Work Timer"), and adds a `div` with the page's id (`work-timer`) and the class `is-floating` to its body, so the page's scoped rules apply there too. It then moves `div.wt-main` into that `div`. The moved elements keep their event listeners and the page script keeps running them, so it is the same timer, not a copy: the sounds, the settings, the tab-title countdown and the live region carry on as before. On the page, `#wt-away` is shown in the timer's place.
- **Closing:** pressing the button again, pressing "Bring it back", and closing the floating window itself all end in the floating window's `pagehide` event. Its handler moves `div.wt-main` back in front of `#wt-away`, hides `#wt-away` and sets `aria-pressed="false"`. If focus was in `#wt-away`, it moves to the floating window button. The browser also closes the floating window when the app's tab closes or reloads.
- While the timer floats, the rest of the app works as usual: the settings sidebar still opens on the page, and other app pages can be shown.
- How the floating window looks at different sizes is under Styles. The Keep screen on button moves with the timer but is hidden in the floating window.
- It is not saved: the timer is back on the page on every load.

#### Styles

Every rule is scoped under `#work-timer` and uses the shared color variables and the timer colors above.

- `#work-timer` is `position: relative`, and `[hidden]` inside it is forced to `display: none`. Buttons inherit the font and color. Focus shows a 2px `--accent` outline with a 2px offset.
- `.wt-icon-btn`: a 36px circle with a 1px `--border` border, `--surface` background and the icon centered; the border turns `--accent` on hover. `.wt-gear` is placed absolutely, 20px from the card's top and right, level with the `h1`, and `.wt-float` sits 20px from the top and 64px from the right, just left of the gear. While pressed, `.wt-float` has an `--accent` background and border and an `--accent-text` icon, like the gear while the sidebar is open. `.wt-close` and `.wt-play` are 32px, and `.wt-play` icons are `--accent`.
- `.wt-body` is a grid with a 20px gap. `.wt-main` is a centered flex column with a 16px gap and `container-type: inline-size`.
- `.wt-tabs` is a segmented control: a pill with a `--bg` background, 1px `--border` border, 3px padding and 2px gap. Each tab is pill-shaped, at least 92px wide, with 6px × 20px padding and `--muted` text (`--text` on hover). The selected tab has a `--surface` background, a `--border` border, `--text` color and weight 600. The running dot is a 6px `--wt-color` circle, 8px from the tab's right edge and vertically centered.
- `.wt-panel` is a centered flex column, full width, with an 8px gap. `.wt-info`: `--muted`, 0.95rem, at least 1.5em tall, no margin.
- `.wt-ring` is a size container (`container-type: size`) and a square, `min(300px, 100cqi)` on each side (`cqi` here is the width of `.wt-main`). It is a one-cell grid with `grid-template: minmax(0, 1fr) / minmax(0, 1fr)`, so the cell is always the ring's own size and its content can't stretch it. Both children sit in that cell (`grid-area: 1 / 1`), centered. The SVG fills the cell (100% × 100%, `overflow: visible`), and its viewBox makes the circle as large as fits, centered. Track and arc have no fill and a stroke width of 6 viewBox units, so the ring's outer edge is at 96% of its box and the stroke is about 6% of its size.
- `.wt-ring-text` is a centered flex column. `.wt-time`: `font-size: 22cqmin` (22% of the ring's smaller side), weight 600, `font-variant-numeric: tabular-nums`, letter spacing −0.02em, line height 1.1. `.wt-ring-label`: `--muted`, `font-size: max(11px, 7cqmin)`, line height 1.3. On the page the ring is at most 300px wide, so the time is up to 66px and the name 21px.
- `.wt-controls` is a flex row with an 8px gap, full width up to 360px, and an 8px top margin. `.wt-btn`: equal widths (`flex: 1 1 0`), 9px × 10px padding, pill-shaped, 1px `--border` border, `--surface` background, weight 600; the border turns `--wt-color` on hover; disabled buttons are at 45% opacity. `.wt-btn-primary` has a `--wt-color` background and border and `--accent-text` text.
- `.wt-wake` is a small pill (5px × 14px padding, 0.875rem, `--muted` text, 1px `--border` border) with an 8px ring before its label (1.5px `currentColor` border). When on, its border and text are `--accent` and the ring is filled. `.wt-wake-note`: `--muted`, 0.85rem, pulled 8px closer to the button, and not shown when empty.
- `.wt-settings`: 14px 16px 16px padding, 1px `--border` border, `--radius` corners, `--bg` background. `.wt-settings-head` is a flex row with the heading on the left and the close button on the right, vertically centered. The `h2` is 1.1rem with no margin. Each `.wt-group` has a 14px top margin; every group after the first also has a 1px `--border` rule above it and 14px top padding. Group `h3`s: 0.8rem, weight 600, uppercase, letter spacing 0.04em, `--muted`. `.wt-field` and `.wt-sound` rows are flex rows with space between, a 12px gap and 4px vertical padding. `.wt-num` holds the input and "min" in `--muted`, 6px apart. Number inputs: 4.5em wide, 5px × 8px padding, 1px `--border` border, 8px radius, `--surface` background, `--text` color, right-aligned text; disabled ones at 50% opacity. The checkbox uses `accent-color: var(--accent)`. `.wt-hint`: `--muted`, 0.85rem, 14px top margin.
- `.wt-away`: a centered flex column with a 12px gap, 32px padding above and below, and centered `--muted` text; its `p` has no margin. `.wt-back` is a small pill: 5px × 14px padding, 1px `--border` border, no background, `--text` color and 0.875rem text, with the border turning `--accent` on hover.
- **Floating window:** `#work-timer.is-floating` is fixed to fill the window (`inset: 0`), with `overflow: hidden`, a `--surface` background and `container: wt-float / size`, so the container queries below, and the `cqw`/`cqh` units that size the ring's box, measure the whole window. Its `.wt-main` stops being a container (`container-type: normal`), takes the full height with 8px padding and a 10px gap, and centers its content vertically. At every size:
  - The Keep screen on button and note are hidden, and so is every disabled timer button, so only buttons that do something show: Start alone when idle, Pause and Reset while running, Resume and Reset when paused.
  - `.wt-panel` takes the height the tabs leave (`flex: 1 1 0`, `min-height: 0`), centers its content vertically, and has a 6px gap.
  - The ring's box is the panel's full width and takes the height the info line and buttons leave (`flex: 1 1 0`, `min-height: 0`), but is never taller than the window's width minus 16px (`max-height: calc(100cqw - 16px)`), so in a tall, narrow window the buttons stay right under the ring. The circle is drawn as large as the box allows, centered, and the time and name inside scale with it, since their sizes are in `cqmin` of the ring.
  - The info line is 0.8rem on a single line (`white-space: nowrap`, at most the panel's width), cut off with an ellipsis when too long.
  - `.wt-controls` shrinks to its buttons, with no top margin. Each `.wt-btn` is as wide as its label (`flex: none`), with 5px × 14px padding and 0.875rem text.

  Then, by window size (container queries on `wt-float`):
  - 360px tall or more and 230px wide or more: the full layout, with the Work/Rest tabs, the ring, the info line and the buttons, one under the other.
  - Shorter than 360px or narrower than 230px: the Work/Rest tabs and the info line are hidden, leaving the ring with the buttons under it. The name inside the ring still says which timer is on show. The buttons have 3px × 12px padding and 13px text.
  - The same, and more than 1.4 times as wide as tall (`aspect-ratio > 7 / 5`): `.wt-panel` becomes a centered row with a 12px gap, with the buttons in a column beside the ring (each as wide as the widest). The ring's box is a square, `min(100cqh - 16px, 100cqw - 112px)` on each side. This is the layout at the default size, 240 × 120.
  - 100px tall or less: the time wouldn't fit inside the ring at a readable size, so a small ring sits beside the time instead, with the buttons on the same line. `.wt-panel` becomes a centered row with a 10px gap. The ring stops being a container and becomes a flex row at its natural size (vertically centered, 0.25em gap) with `font-size: min(13cqw, 60cqh)`. Its SVG is 0.8em square with a stroke width of 10, the time is 1em with line height 1, and the name is hidden. The buttons sit in a row, with 2px × 8px padding and 12px text.
  - 100px tall or less and 280px wide or less: Reset is hidden as well.

#### Accessibility

- The Work and Rest tabs follow the tabs pattern: `aria-selected`, a roving `tabindex`, the Left and Right arrow keys switch tabs, and Home and End go to Work and Rest.
- `role="timer"` keeps screen readers from reading out every second. The end-of-session message is announced through `#wt-live`, which is emptied first and filled 100 ms later, so a repeated message is read again.
- When Start, Pause or Reset disables the button that had focus, focus moves on: from Start to Pause, from Pause to Start, from Reset to Start. Focus is checked in the button's own document, since the button may be in the floating window.
- The ring's SVG is hidden from screen readers (`aria-hidden="true"`). The time and the name under it are plain text.
- In the smaller floating window layouts, where the Work/Rest tabs are hidden, the name inside the ring (`#wt-ring-label`) says which timer is on show. Only the one-line strip (100px tall or less) has no name. There, only the timer's color tells them apart.

#### Known limits

- Phones usually suspend a page when the screen locks or the browser goes to the background, so dings can't play then. The time is still right on return. "Keep screen on" prevents the screen lock.
- On an iPhone, silent mode may mute the sounds.
- In a browser without Web Audio, the timers work silently.
- The floating window needs the Document Picture-in-Picture API: desktop Chrome and Edge 116 or later, and Firefox 151 or later. Safari and phone browsers don't have it, so its button doesn't appear there. The browser decides where the window opens and sets its own smallest and largest sizes.
