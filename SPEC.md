# misc_utils_webapp: specification

This document describes the app in enough detail to rebuild it from scratch, or something very close to it. It always describes the current version. Whenever `index.html` changes, this file changes with it (see the rules in `CLAUDE.md`).

## 1. Overview

- A personal collection of everyday utilities in one web app, with one utility per page. Pages are switched with tabs.
- The whole app is a single static file, `index.html`, with all HTML, CSS and JavaScript inline. The Work Timer's sound recordings and the Eye Exercise's recording are embedded in it too, as base64 (see 5.1, Sounds, and 5.3, Recording), and so are the Exercise page's routines, as plain text (see 5.2, Routines).
- There is no backend, database, user login or build step, and no external dependencies (no frameworks, CDNs, web fonts or files loaded at run time). The app works offline, opened directly from disk (`file://`) or served from any static host.
- Nothing is sent anywhere. A page may remember its own settings in the browser's `localStorage` (see 3.4). Nothing else is stored.
- Target browsers: current Chrome, Safari, Firefox and Edge, on desktop and mobile. Plain modern JavaScript (ES2015+) is fine.

## 2. Structure of `index.html`

In order:

1. `<head>`: `charset` utf-8, viewport `width=device-width, initial-scale=1`, an empty `<title></title>` that the shared script fills in, and one `<style>` block with two parts: shared styles, then page-specific styles. The Work Timer script adds a `<link rel="icon">` at the end of `<head>` when it runs (see 5.1, Tab icon).
2. `<header class="app-header">` containing a `.container` with an empty `<nav class="tabs" id="tabs" aria-label="Pages">` that the shared script fills. The app has no name, so the header shows only the tabs.
3. `<main class="container">` containing the pages, one `<section class="page">` each.
4. The shared navigation `<script>`.
5. Page scripts, one `<script>` element per page that needs one: the Work Timer's, then the Exercise page's, then the Eye Exercise's, then the Breathing page's.
6. Exercise routines, under a comment headed `Exercise routines`: one `<script type="text/plain" class="ex-routine">` element per routine, holding a copy of its file's text (see 5.2, Routines). This is readable text, a few KB per routine.
7. Embedded data, under a comment headed `Embedded data`: the Work Timer's recordings, one `<script type="text/plain" class="wt-recording">` element each (see 5.1, Sounds), then the Eye Exercise's recording, in a `<script type="text/plain" class="ee-recording">` element (see 5.3, Recording). Each is one very long line of base64, about 400 KB in all, so they come last and the code above reads without scrolling past them.

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
- The id is read from the hash by stripping a leading `#` or `#/`. Both `#/exercise` and `#exercise` work, but links always use `#/<id>`.
- If there is no hash, or no page has that id, the first page is shown. The URL is not rewritten.
- The shown section gets the class `is-active`. CSS hides every other page with `.page:not(.is-active) { display: none !important; }`. The `!important` stops a page's own `display` rule from un-hiding it.
- The matching tab link gets `aria-current="page"`, and the attribute is removed from the other links.
- The document title (the label on the browser tab) becomes the page's title, with nothing added to it. One exception: while a Work Timer countdown runs, the Work Timer script replaces the title with the time left, or during a short break with the break's time left (see 5.1).

Links use `#/<id>` rather than `#<id>` because a fragment that matches an element id makes the browser scroll to that element. With the `/`, no element matches.

### 3.3 Adding a page

1. Copy an existing `<section class="page">` block and put it where its tab should appear.
2. Give it a new `id` and `data-title`, and update its `<h1>`.
3. Add any page-specific CSS in the "Page-specific styles" part of the `<style>` block, with every rule scoped under the page's id (for example `#unit-converter .result { ... }`).
4. Add any page-specific JavaScript in a new `<script>` element after the shared script, wrapped in `(() => { ... })();`. This keeps its variables private, and an error in it can't break other pages or the tab bar.
5. Start the page's own ids and classes with a short prefix (the Work Timer uses `wt-`), so they can't clash with another page's.
6. Put any large embedded data, such as sounds or images, at the end of `<body>` after the page scripts, in elements the browser neither runs nor shows, such as `<script type="text/plain">`. The Work Timer's recordings are an example (5.1, Sounds).
7. Document the page in section 5 of this file and in `README.md`.

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
- Page card: `--surface` background, 1px `--border` border, `--radius` corners, 20px padding. The page's `<h1>` is 1.5rem, with no top margin and 20px below. A page may give its card its own background: the Work Timer's card takes a very soft shade of its timer's color, or a bright green during a short break (5.1).
- `.placeholder`: text in `--muted`, used for "nothing here yet" notes.

## 5. Pages

| Tab | id | Purpose | Content |
|---|---|---|---|
| Work Timer | `work-timer` | Work and rest countdowns, with random short breaks during work | See 5.1 |
| Exercise | `exercise` | Follow-along exercise routines: get-ready countdowns, hold countdowns and rep counters, with dings | See 5.2 |
| Eye Exercise | `eye-exercise` | Plays an eye exercise recording five times in a row, then a big ding | See 5.3 |
| Breathing | `breathing` | Breathing exercises (box and 4-7-8) for a chosen time: a ring in each part's color and a ding every second | See 5.4 |

### 5.1 Work Timer

A countdown for work and one for rest on a single page, each with its own color: blue for work and green for rest. The card behind the timer takes a very soft shade of the color of the timer on show. The time sits in the middle of a ring whose arc runs down as the session goes. While the work timer runs, short breaks come at random moments: a sound starts each one and another ends it, and in between the card turns a bright green of its own and counts the break down, so the break is noticed even with the sound off. Each start and end, of a session or a break, has its own recorded sound, embedded in the file, with generated dings as an option. On a computer, the timer can also pop out into a small floating window that stays on top of other windows. The page's ids and classes start with `wt-`. Its styles sit under `/* ---------- Work Timer (#work-timer) ---------- */` in the page-specific styles, and its script is the first page script.

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
       - "Durations": two `label.wt-field` rows, "Work" and "Rest". Each wraps a `span.wt-num` holding an input followed by " min": `#wt-work-min` and `#wt-rest-min`.
       - "Random breaks": `label.wt-check` with the checkbox `#wt-breaks-on` and the text "Take short breaks at random times during work"; a `label.wt-field` row "Break length" whose `span.wt-num` holds the input `#wt-break-sec` followed by " sec"; then `label.wt-field` rows "Shortest gap" (`#wt-gap-min`) and "Longest gap" (`#wt-gap-max`), built like the durations.
       - These five inputs are typed into by hand: each is `<input type="text" inputmode="numeric" autocomplete="off">`, not a number input, so it has no up/down arrows, and neither the mouse wheel nor the arrow keys change it. Phones show a number keypad. They have no `min`, `max` or `step`; the script checks the ranges (see Settings sidebar).
       - "Sounds": two `label.wt-check` rows with radio buttons named `wt-sounds`, "Recorded sounds" (value `recorded`) and "Dings" (value `dings`). Then three `div.wt-sound` rows, "Work", "Rest" and "Break". Each ends with a `span.wt-plays` holding two `button.wt-play`, "Start" and "End". Their `data-sound` is the moment they play (`work_begin` and `work_end`, `rest_begin` and `rest_end`, `break_begin` and `break_end`), and their `aria-label` says it, for example "Play the work start sound". Each label follows a 10px play triangle: the path `M9 5.5v13l10-6.5z`, filled, on a `7 5 14 14` viewBox that centers it.
     - `p.wt-hint`: "Settings are saved in this browser. Clearing the browser's site data brings back the defaults."

#### Timers

- There are two timers, Work and Rest, each with a duration from the settings, in whole minutes: 10 to 120 for Work (default 25) and 3 to 30 for Rest (default 5). Each one is idle, running or paused.
- Time shows as `MM:SS`, rounded up to the whole second, so a fresh 25-minute timer shows 25:00 and reaches 00:00 only at the very end. Minutes can go past 59 (for example 90:00).
- Time is kept with the wall clock: a running timer stores the moment it will end (from `Date.now()`), so the display is right whenever it is redrawn, even after the page was in the background or the computer slept.
- **Start** begins a fresh session at the full duration, with the timer's start sound (`work_begin` or `rest_begin`, see Sounds). On a paused timer it reads **Resume** and carries on from where it paused, with no sound. **Pause** freezes the timer. **Reset** stops it and puts it back to its full duration, silently.
- Button states for the timer on show: Start is enabled unless that timer is running, Pause only while it runs, Reset only once it has started (running or paused).
- One timer at a time: pressing Start on one timer first resets the other, silently, if it is running or paused. Switching between the Work and Rest tabs only changes which timer is on show and never stops anything.
- When a session reaches zero, its end sound plays (`work_end` or `rest_end`), that timer goes back to its full duration (idle), and the page shows the other timer, ready but not started. The info line then reads "Work session done. Start your rest when you’re ready." or "Rest is over. Start the next work session when you’re ready." until the next press of Start or Reset or a Work/Rest tab switch. The same text goes to the live region for screen readers.
- The info line (`#wt-info`) shows the first that applies: the end-of-session message; "Paused" if the timer on show is paused; "Take a short break" during a break (with the Work timer on show); on the Work tab, "10 s break every 2–5 min" (the break length in seconds, then the shortest and longest gaps, which are never equal) or "Random breaks off"; on the Rest tab, nothing. It keeps its height when empty.
- The tab of a running timer shows a small dot in that timer's color. The time is drawn in the color of the timer on show while running, `--muted` while paused and `--text` otherwise (see Timer colors).
- A changed duration shows at once on an idle timer. A timer that has started keeps its current session length, and the new duration applies from its next reset or new session. To do this, each timer stores the length of its current session (`total`). It is set to the duration when a fresh session starts, when the timer is reset or finishes, and when its duration changes while it is idle.
- **Browser tab title:** while a timer runs, the title is the time left and the timer's name, `18:42 · Work` or `04:12 · Rest`, updated along with the display, whichever app page is on show. During a break it is the time left in the break instead, `00:08 · Break`, whichever Work/Rest tab is on show. When no timer is running, the title goes back to the shown page's title (its `data-title`, or its id). The page script also listens for `hashchange`. Its listener runs after the shared script's, so it puts the countdown back right after a page switch.

#### Ring

- The time sits in the middle of a ring, with the name of the timer on show ("Work" or "Rest") under it in smaller `--muted` text. The ring is a circle in a lighter shade of the timer's color (the track), with an arc in the solid color on top that shows how much of the session is left.
- The arc starts at the top (12 o'clock) and runs clockwise. Its length is the share of the session left: time left ÷ session length (`total`). So the ring is full while a timer is idle, and as the time runs down, the arc's end moves back counter-clockwise toward the top, with nothing left at zero. When a session ends, the page shows the other timer with a full ring.
- It always shows the timer on show, and is redrawn along with the time: on every tick and every render. Because it uses the session length, a duration changed mid-session doesn't make it jump.
- **During a break** (with the Work timer on show), the ring shows the break instead: the time in the middle counts down the time left in the break (`00:10` … `00:01`, rounded up like the timers), the arc is the share of the break left (time left ÷ break length, so it is full when the break starts and runs down to nothing), and the name under the time reads "Break". The work clock keeps running underneath, so when the break ends, the work time and its arc come back, the break's length further on.
- The arc is drawn with the path's `stroke-dasharray`: `none` when the share is 1, otherwise `<share × C> <C>` with 2 decimals, where C = 2π × 45 ≈ 282.74 (the circumference). A share of 0 draws nothing, since the stroke ends are flat (butt). The attribute is written only when it changes.
- Colors: the track is `--wt-color-pale` and the arc `--wt-color`, the colors of the timer on show, or of the break during one (see Timer colors). While that timer is paused, the whole ring turns grey, matching the time: the track is `--border` and the arc `--muted`. The card behind keeps its soft shade. The script sets the classes `is-running` and `is-paused` on `#wt-ring` for the timer on show, and the colors of the time and the ring follow from them.
- In the floating window the ring adapts to the window's size, and in a very short window it becomes a small ring beside the time (see Styles).

#### Timer colors

Each timer has its own color, so the two look distinct: a soft blue for Work and a soft green for Rest, never pure `#0000ff` or `#00ff00`. Each color comes in three shades: the solid color, a lighter shade (pale) for the ring's track, and an even lighter, very soft shade (soft) for the card behind the timer. A short break during work has a green of its own, also in three shades (see Break colors below). They are custom properties on `#work-timer`, overridden inside a `prefers-color-scheme: dark` media query like the shared colors:

| Variable | Light | Dark | Used for |
|---|---|---|---|
| `--wt-work` | `#2f6fde` | `#5b8ef0` | Work timer (the same blue as `--accent`) |
| `--wt-work-pale` | `#c1d4f5` | `#2e4266` | Work timer's ring track |
| `--wt-work-soft` | `#eef3fc` | `#222a38` | The card while Work is on show |
| `--wt-rest` | `#318454` | `#53ae77` | Rest timer |
| `--wt-rest-pale` | `#c1dacc` | `#2f4a38` | Rest timer's ring track |
| `--wt-rest-soft` | `#ecf6ef` | `#1f2e24` | The card while Rest is on show |
| `--wt-break-fill` | `#87d576` | `#316a23` | The card during a break |
| `--wt-break-track` | `#d5f2cf` | `#22441a` | The ring's track during a break |
| `--wt-break-ink` | `#26511c` | `#d3f2cd` | The ring's arc, the time and the card's grey text during a break |

- The green has about the same lightness as the blue (OKLCH lightness 0.55 against 0.56 in light, 0.68 against 0.66 in dark) but less chroma (0.11 to 0.12), which keeps it calm.
- Pale (the track): in light, the solid color mixed 30% into `--surface` (white). In dark, a dim version of the color at OKLCH lightness 0.38, in the color's own hue, between the card and the solid color.
- Soft (the card): the two colors share one OKLCH lightness and chroma, each in its own hue, so the two cards look equally soft: lightness 0.96 and chroma 0.013 in light, 0.28 and 0.028 in dark. The blue ones are the solid blue mixed 8% into `--surface`. The dark shades keep each color's own hue because mixing green into the bluish dark `--surface` would turn it teal.
- Contrast: the Start button's `--accent-text` label is at least 4.6:1 on every timer color. Each timer color is at least 4.1:1 against its own card shade (the running time is large text, which needs 3:1). `--muted` text is at least 4.8:1 on every card shade, and the arc is at least 3.1:1 against its track.
- `--wt-color`, `--wt-color-pale` and `--wt-color-soft` hold the shades of the timer on show. `#work-timer` sets them to the Work shades, and `#work-timer.is-rest` to the Rest shades. Every render toggles the class `is-rest` on the page's `section#work-timer` and, while the floating window is open, on its `div#work-timer`, so the class is there while the Rest timer is on show, and everything inside follows. The Work/Rest tabs are the exception: `#wt-tab-work` and `#wt-tab-rest` set `--wt-color` to their own timer's color, so each tab's running dot keeps its own color whichever timer is on show.
- `--wt-color-soft` is the background of `#work-timer`, replacing the shared `--surface`, so the page's card and the floating window take it. The other shades color the ring's arc and track, the time while running, the running dot on each tab, the background and border of the Start/Resume button (`.wt-btn-primary`), and the hover border of the timer buttons. The rest of the page keeps `--accent`: the gear and floating window buttons, the settings sidebar with its play buttons, Keep screen on, and the focus outline. The controls on the card keep their own backgrounds: `--surface` for the buttons, and `--bg` for the Work/Rest switch and the settings sidebar.
- The card's shade changes whenever the timer on show does: on a Work/Rest tab switch, and when a session ends and the page shows the other timer. It also changes when a break starts or ends. While the timer floats, the page's card (behind "The timer is in a floating window.") keeps following the timer on show.

Break colors:

- The break's green is its own, distinct from Rest's: brighter and more vivid, leaning a little toward yellow (OKLCH hue 140, where Rest's is 155). Its shades swap roles with the timers': the card fills with the strong shade, and the ring and the time take a contrasting one (ink).
  - Light: the fill is a bright green at OKLCH lightness 0.80 and chroma 0.15, so the page's dark text stays dark. The ink is a deep green (0.39, 0.095), and the track is paler than the fill (0.93, 0.056).
  - Dark: the fill is a deep green (0.47, 0.12), so the page's light text stays light. The ink is a very light green (0.93, 0.06), and the track is dimmer than the fill (0.35, 0.08).
  - Either way, the card changes a lot from the timers' near-white (or near-black) soft shades, which catches the eye even in a small window in a corner of the screen.
- Contrast during a break, light and dark: `--text` on the fill is 8.8:1 and 5.4:1. The ink on the fill is 5.2:1 and 5.4:1, enough for the small text under the time and on the info line. The arc against its track is 7.7:1 and 9.1:1. The Start button's `--accent-text` label on the ink is at least 9:1.
- `#work-timer.is-break`, after the `.is-rest` rule, sets `--wt-color`, `--wt-color-pale` and `--wt-color-soft` to the break's ink, track and fill. So the card's background (and the floating window's), the ring, the running time, the Start button and the timer buttons' hover border all follow. Every render toggles `is-break` on the page's `section#work-timer` and, while the floating window is open, on its `div#work-timer`, so the class is there while a break is in progress and the Work timer is on show. The Work tab's running dot stays blue.
- During a break, the elements that are otherwise `--muted` take `--wt-break-ink`: `.wt-ring-label`, `.wt-info`, `.wt-wake`, `.wt-wake-note` and `.wt-away`. So do the border of `.wt-wake` on hover or when on, and the focus outline. The Work/Rest switch, the buttons and the settings sidebar keep their own backgrounds and colors.

#### Random breaks

A short break during work, meant for stopping for a moment: standing up, stretching, looking away.

- Breaks happen only while the work timer is running, and only when the setting is on (the default).
- **A break:** the `break_begin` sound plays at its start and `break_end` at its end, the break length apart (default 10 seconds, settable from 5 to 15 in whole seconds). In between, the page shows the break (see Ring, Timer colors and Tab icon).
- **The work clock keeps running** through a break, so a session still ends at its set time (25 minutes by default).
- **Gaps:** each gap is the work time between two breaks, from the end of one to the start of the next, so breaks never overlap, whatever the lengths. Each gap is a fresh random time, evenly spread between the shortest and longest gap (defaults 2 and 5 minutes, see Settings sidebar). The settings are whole minutes, but a gap isn't rounded. The first gap counts from the start of the session. The time to the next break is never shown.
- **Near the end:** a break starts only if it can end at least 15 seconds before the session does, so its end sound never runs into the session's end sound.
- **Pause:** pausing during work remembers the work time left until the next break, and Resume continues from there. Pausing during a break ends the break silently (the pause is a break in itself), and Resume draws a fresh gap. A fresh session draws a fresh first gap.
- **Ending early:** Reset, starting the Rest timer, or the session ending also ends a break in progress silently. A break's sounds that haven't started are cancelled, and one already playing plays out.
- **Changing any break setting** (on/off, length, gaps) applies at once. The planned breaks are dropped (one in progress ends silently), and if work is running, a fresh gap is drawn from now. If work is paused, the remembered gap is dropped and Resume draws a fresh one.
- **When a break starts**, the page shows the Work timer if the Rest tab was on show. Switching to the Rest tab during a break shows the Rest timer as usual, while the browser tab's title and icon still show the break. Switching back to Work shows the break again.
- **Screen readers** hear "Short break" when a break starts and "Back to work" when it ends by itself, through `#wt-live`. A break ended by Pause, Reset or a settings change isn't announced.
- **How it works:** breaks are planned ahead in wall-clock time, separately from their sounds, so they show on screen even in a browser without Web Audio. Each planned break keeps its start (`at`), its end (`end`, the start plus the break length) and the handles of its two sounds. The script keeps the next break not planned yet (`nextBreakAt`), and the break in progress as last shown (`shownBreak`). On every tick while work runs, it plans breaks up to 5 minutes ahead (by start time), then finds the break whose start ≤ now < end. If that differs from `shownBreak`, a break has started or ended: it updates `shownBreak`, switches to the Work timer when one starts, makes the announcement and re-renders. Pause, Reset, finishing and a break setting change clear `shownBreak`.

#### Tab icon

- When it runs, the Work Timer script appends a `<link rel="icon">` to the document's `<head>`. During a break (whichever Work/Rest tab is on show), its `href` is a green dot. Otherwise it is a blank, transparent icon.
- Both are SVG data URLs: `data:image/svg+xml,` followed by `encodeURIComponent` of `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16">…</svg>`. The dot is `<circle cx="8" cy="8" r="7" fill="#479d32"/>`, a mid green at least 3:1 against white and dark tab bars. The blank icon has nothing inside. The `href` is written only when it changes, on every render.
- The icon is blank rather than missing, because browsers don't reliably bring back their default icon when a page removes its own, so a removed dot could stay on show. As a result, the app's browser tab shows no icon apart from the dot during a break.

#### Sounds

Six moments have a sound, each named after its file in `timer_sound/`:

| Moment | Plays when | Recording | Ding |
|---|---|---|---|
| `work_begin` | A fresh work session starts (Start, not Resume) | `work_begin.m4a`, 4.0 s | Start |
| `work_end` | A work session reaches zero | `work_end.m4a`, 2.0 s | End |
| `rest_begin` | A fresh rest session starts | `rest_begin.m4a`, 2.0 s | Start |
| `rest_end` | A rest session reaches zero | `rest_end.m4a`, 4.0 s | End |
| `break_begin` | A break starts | `break_begin.m4a`, 1.9 s | Break ding |
| `break_end` | A break ends by itself | `break_end.m4a`, 1.8 s | Break ding |

- **Choice:** the "Sounds" setting picks what plays: **Recorded sounds** (`recorded`, the default), the owner's recordings, or **Dings** (`dings`), three sounds generated with the Web Audio API, which were the app's only sounds before the recordings. A new choice applies at once: the sounds already scheduled (the end sound, and the planned breaks' sounds that haven't started) are scheduled again with it, and one already playing plays out.
- **Previews:** each play button in the sidebar plays its moment's sound with the current choice, at once, whatever the timers are doing.
- **Playback:** everything plays through the Web Audio API. One `AudioContext` is created by the first click that needs sound (Start, Resume or a play button), because browsers allow audio only after a user gesture. Everything plays through a master gain of 0.9. Each sound gets its own gain node (its bus) into the master, and disconnects it once all of its sources have ended. The script keeps the set of sounds scheduled or playing, so that the audio context isn't suspended while one still plays (see Keeping time in a background tab).

Recordings:

- `work_begin.m4a` and `rest_end.m4a` are the same file, and so are `work_end.m4a` and `rest_begin.m4a`. All are AAC in an `.m4a` container, 48 kHz stereo.
- **Embedding:** they are embedded in `index.html`, so that it stays one self-contained file. The app never reads `timer_sound/`. Each recording is a `<script type="text/plain" class="wt-recording">` element at the end of `<body>` (see 2), whose text is the file's bytes in base64, on one line. Its `data-files` attribute names the files it stands for, without `.m4a` and separated by spaces, and these are the moments it plays at. Identical files share one element, so there are four: `work_begin rest_end`, `work_end rest_begin`, `break_begin` and `break_end`. The `text/plain` type keeps the browser from running them.
- **Decoding:** creating the `AudioContext` also starts decoding every recording, once for the page's lifetime: `atob` turns an element's text into bytes, and `decodeAudioData` turns those into an `AudioBuffer`, kept for each moment its `data-files` names. If the page is still loading then (the first click came early), decoding waits for `DOMContentLoaded`. A recorded sound asked for while decoding is under way waits for it, then starts at its time, or 30 ms later if its time has passed. So the very first sound after loading can come a fraction of a second late (about 0.15 s in Chrome on a Mac).
- **Level:** a recording plays through an `AudioBufferSourceNode` at its own level, with only the master gain. After it, the recordings peak between about −4.5 and −11 dBFS and sound a little quieter than the dings. `break_end` is the quietest.
- **Fallback:** a moment whose recording can't be decoded, or that no `data-files` names, plays its ding instead.
- **Changing a recording:** replace its file in `timer_sound/`, then put the new file's base64 (for example from `base64 -i timer_sound/break_end.m4a`) in its element, as one line. If the element stands for two files and only one changed, give the changed one its own element and take its name out of the old element's `data-files`. AAC (`.m4a`) and MP3 are safe formats: every target browser decodes them.

Dings:

Every note is a soft bell of four sine partials:

| Partial | Frequency | Peak (× note level) | Fade (× note fade time) |
|---|---|---|---|
| 1 | f | 1 | 1 |
| 2 | 2f | 0.4 | 0.6 |
| 3 | 3f | 0.18 | 0.4 |
| 4 | 4.16f | 0.08 | 0.25 |

Each partial's gain starts at 0, rises linearly to its peak in 5 ms, then falls exponentially to 0.0001 by the end of its fade. Its oscillator stops 50 ms after that.

| Ding | Notes: start, pitch, level, fade time | Moments |
|---|---|---|
| Break ding (`ding`) | 0 s, A5 (880 Hz), 0.5, 2 s | `break_begin`, `break_end` |
| Start (`start`) | 0 s, G5 (783.99 Hz), 0.42, 1.2 s; then 0.16 s, D6 (1174.66 Hz), 0.42, 1.6 s | `work_begin`, `rest_begin` |
| End (`end`) | 0 s C6 (1046.5 Hz), 0.2 s G5 (783.99 Hz), 0.4 s E5 (659.25 Hz), 0.6 s C5 (523.25 Hz); all at level 0.45; fades 1.4, 1.4, 1.6 and 3 s | `work_end`, `rest_end` |

So the break ding is one ding, the start is a short rising pair, and the end is a falling run of four notes, the longest and loudest of the three. Their peaks are about −4.6, −4.2 and −3.5 dBFS, well clear of clipping.

#### Keeping time in a background tab

Browsers slow down timers in background tabs (Chrome to once a minute after five minutes hidden), so the page doesn't rely on them for anything that must be on time:

- The end sound and the break sounds are scheduled ahead on the audio clock, which keeps running in the background: a recording through its `AudioBufferSourceNode`'s `start(time)`, a ding through its oscillators'. When a session starts or resumes, the end sound is scheduled for the end time, and breaks are planned up to 5 minutes ahead, each with its two sounds scheduled at its start and end. Every tick extends that window. Each scheduled sound has its own gain node: cancelling one (on Pause, Reset, a break setting change, or a new sound choice, which schedules it again) stops its sources and disconnects that node. A break's sounds are cancelled only if they haven't started yet, so one already playing plays out. A finished sound disconnects its own gain node.
- A 250 ms `setInterval` redraws the display while a timer runs. Next to it runs a chain of silent one-second `ConstantSourceNode`s (offset 0, connected to the output). Each one's `ended` event calls the same tick and starts the next one, and these events aren't slowed down in background tabs. That keeps the title countdown current and ends the session on time in a background tab. Tick is safe to call any number of times.
- A break's look, title and tab icon switch on the first tick after it starts or ends: within 250 ms while the page or the floating window is on screen, and within about a second in a background tab (through the silent chain).
- While the floating window is open, a second 250 ms interval runs on the floating window itself (that window's own `setInterval`) whenever a timer runs. The floating window stays on screen, so browsers don't slow its timers even while the app's tab is hidden or covered, and the floating display stays current to the second. Stopping the timer clears it, and it goes away with the window.
- On each tick, if the wall clock and the audio clock have drifted a second or more apart since the sounds were scheduled (the audio clock stalls while a computer sleeps), the end sound and the sounds of the planned breaks are scheduled again from their wall-clock times. Sounds whose time has passed are dropped. Breaks missed while asleep are skipped, not played in a burst.
- At the end of a session, if the scheduled end sound is still more than 0.5 s in the future (because the audio clock is behind), it is cancelled and, if audio is running, the end sound plays at once instead. Otherwise it is left to ring out.
- The audio context is suspended 5 seconds after a timer stops or a preview plays, if no timer is running then and no sound is still playing. While one is (a long recording, say), it checks again every 5 seconds until the last one has ended. The next Start, Resume or play button wakes it again.

#### Settings sidebar

- The gear opens and closes the sidebar. The close button, the backdrop (narrow screens) and Escape (while the page is shown) also close it. Opening moves focus to the close button, and closing returns it to the gear. While the sidebar is open, the gear has the accent background. The sidebar starts closed on every load.
- Wider than 720px, `div.wt-body` gets the class `is-settings-open` and becomes two columns, `minmax(0, 1fr) 260px`, with the sidebar beside the timer. At 720px or narrower, the sidebar is a panel fixed to the right edge of the window, `min(320px, 88vw)` wide and full height, scrolling if needed, with a `--surface` background, only a left border, no rounded corners and a shadow (`-8px 0 24px rgba(0, 0, 0, 0.25)`). It sits over a fixed full-window backdrop of `rgba(0, 0, 0, 0.4)`.
- The five numbers are typed in by hand, as whole numbers, in these ranges:

| Setting | Input | Range | Default |
|---|---|---|---|
| Work | `#wt-work-min` | 10 to 120 min | 25 |
| Rest | `#wt-rest-min` | 3 to 30 min | 5 |
| Break length | `#wt-break-sec` | 5 to 15 s | 10 |
| Shortest gap | `#wt-gap-min` | 1 to 5 min | 2 |
| Longest gap | `#wt-gap-max` | 2 to 10 min | 5 |

- Entries are checked on `change`, which fires when an entry is committed (Enter, or leaving the field). An empty field (spaces only, too) or anything `Number()` can't read, such as "abc" or "25 min", puts the previous value back. A number is rounded to the nearest whole number (12.5 becomes 13, 12.4 becomes 12) and clamped to its range (200 minutes of work becomes 120). An entry that ends up at the value already set changes nothing, so it isn't saved and doesn't redraw the planned breaks.
- The shortest gap always stays below the longest, at least a minute apart. A shortest gap typed at or above the longest moves the longest to 1 minute above it, and a longest gap typed at or below the shortest moves the shortest to 1 minute below it. For example, with gaps of 2 and 5, typing 5 as the shortest makes the longest 6, and typing 2 as the longest makes the shortest 1. The moved value always stays in its range, because the shortest gap is at most 5 and the longest at least 2.
- With the breaks off, the break length and gap inputs are disabled and their rows turn `--muted`.
- Choosing Recorded sounds or Dings applies at once, even to the sounds already scheduled (see Sounds). The play buttons play their moment's sound with the current choice, at once, whatever the timers are doing.
- Every change is saved to `localStorage` under `misc-utils:work-timer`, as JSON like `{"workMin":25,"restMin":5,"remindOn":true,"gapMinMin":2,"gapMaxMin":5,"breakSec":10,"sounds":"recorded"}`. `remindOn` is the breaks switch. Its name dates from when the breaks were single reminder dings, and it is kept so that saved settings carry over. `sounds` is the sound choice. On load, each of the five numbers is used only if it is a finite number, rounded and clamped as above, `remindOn` only if it is a boolean, and `sounds` only if it is `"recorded"` or `"dings"`. Anything else falls back to its default, so settings saved before the sound choice existed get Recorded sounds. Numbers saved by earlier versions, which allowed half minutes and wider ranges, are rounded and clamped into the current ranges. If the shortest gap then isn't below the longest, both gaps go back to their defaults.

#### Keep screen on

- It uses the Screen Wake Lock API, and the button appears only if `navigator.wakeLock` exists.
- The button toggles (`aria-pressed`). On, it requests a screen wake lock. Off, it releases it. Browsers drop the lock whenever the page is hidden, so it is requested again when the page becomes visible while the button is on. If a request fails, the button turns off and `#wt-wake-note` says "This browser didn’t allow keeping the screen on."
- It is not saved: it is off on every load.

#### Floating window

On a computer, the timer can move into a small window that stays on top of other windows, so it can sit in a corner of the screen. Ordinary browser windows can't be made narrower than about 400–500 px, and a page can't change that, which is why this uses the Document Picture-in-Picture API.

- The button `#wt-float` appears only if `documentPictureInPicture` exists in `window`. It is a toggle: `aria-pressed="true"` while the floating window is open.
- **Opening:** the button calls `documentPictureInPicture.requestWindow({ width: 240, height: 120 })` from its click, since browsers allow this only after a user gesture. The button is disabled while the request is pending, and if the browser refuses, nothing changes. The browser decides where the window opens and may adjust its size. The user can move and resize it.
- The new window starts empty. The script copies every `<style>` element into its `<head>`, sets its title to the page's `data-title` ("Work Timer"), and adds a `div` with the page's id (`work-timer`) and the class `is-floating` to its body, so the page's scoped rules apply there too. It then moves `div.wt-main` into that `div`, keeps a reference to the `div` so that every render can toggle its `is-rest` and `is-break` classes (see Timer colors), and renders once, so the window opens in the colors of the timer on show, or of a break in progress. The moved elements keep their event listeners and the page script keeps running them, so it is the same timer, not a copy: the sounds, the settings, the tab-title countdown and the live region carry on as before. On the page, `#wt-away` is shown in the timer's place.
- **Closing:** pressing the button again, pressing "Bring it back", and closing the floating window itself all end in the floating window's `pagehide` event. Its handler drops the reference to the floating `div`, moves `div.wt-main` back in front of `#wt-away`, hides `#wt-away` and sets `aria-pressed="false"`. If focus was in `#wt-away`, it moves to the floating window button. The browser also closes the floating window when the app's tab closes or reloads.
- While the timer floats, the rest of the app works as usual: the settings sidebar still opens on the page, and other app pages can be shown.
- How the floating window looks at different sizes is under Styles. The Keep screen on button moves with the timer but is hidden in the floating window.
- It is not saved: the timer is back on the page on every load.

#### Styles

Every rule is scoped under `#work-timer` and uses the shared color variables and the timer colors above. The rules for a break's look are under Timer colors (Break colors).

- `#work-timer` is `position: relative`, with the card's soft shade, `--wt-color-soft`, as its background, and `[hidden]` inside it is forced to `display: none`. Buttons inherit the font and color. Focus shows a 2px `--accent` outline with a 2px offset.
- `.wt-icon-btn`: a 36px circle with a 1px `--border` border, `--surface` background and the icon centered; the border turns `--accent` on hover. `.wt-gear` is placed absolutely, 20px from the card's top and right, level with the `h1`, and `.wt-float` sits 20px from the top and 64px from the right, just left of the gear. While pressed, `.wt-float` has an `--accent` background and border and an `--accent-text` icon, like the gear while the sidebar is open. `.wt-close` is 32px.
- `.wt-body` is a grid with a 20px gap. `.wt-main` is a centered flex column with a 16px gap and `container-type: inline-size`.
- `.wt-tabs` is a segmented control: a pill with a `--bg` background, 1px `--border` border, 3px padding and 2px gap. Each tab is pill-shaped, at least 92px wide, with 6px × 20px padding and `--muted` text (`--text` on hover). The selected tab has a `--surface` background, a `--border` border, `--text` color and weight 600. The running dot is a 6px `--wt-color` circle, 8px from the tab's right edge and vertically centered.
- `.wt-panel` is a centered flex column, full width, with an 8px gap. `.wt-info`: `--muted`, 0.95rem, at least 1.5em tall, no margin.
- `.wt-ring` is a size container (`container-type: size`) and a square, `min(300px, 100cqi)` on each side (`cqi` here is the width of `.wt-main`). It is a one-cell grid with `grid-template: minmax(0, 1fr) / minmax(0, 1fr)`, so the cell is always the ring's own size and its content can't stretch it. Both children sit in that cell (`grid-area: 1 / 1`), centered. The SVG fills the cell (100% × 100%, `overflow: visible`), and its viewBox makes the circle as large as fits, centered. Track and arc have no fill and a stroke width of 6 viewBox units, so the ring's outer edge is at 96% of its box and the stroke is about 6% of its size.
- `.wt-ring-text` is a centered flex column. `.wt-time`: `font-size: 22cqmin` (22% of the ring's smaller side), weight 600, `font-variant-numeric: tabular-nums`, letter spacing −0.02em, line height 1.1. `.wt-ring-label`: `--muted`, `font-size: max(11px, 7cqmin)`, line height 1.3. On the page the ring is at most 300px wide, so the time is up to 66px and the name 21px.
- `.wt-controls` is a flex row with an 8px gap, full width up to 360px, and an 8px top margin. `.wt-btn`: equal widths (`flex: 1 1 0`), 9px × 10px padding, pill-shaped, 1px `--border` border, `--surface` background, weight 600; the border turns `--wt-color` on hover; disabled buttons are at 45% opacity. `.wt-btn-primary` has a `--wt-color` background and border and `--accent-text` text.
- `.wt-wake` is a small pill (5px × 14px padding, 0.875rem, `--muted` text, 1px `--border` border) with an 8px ring before its label (1.5px `currentColor` border). When on, its border and text are `--accent` and the ring is filled. `.wt-wake-note`: `--muted`, 0.85rem, pulled 8px closer to the button, and not shown when empty.
- `.wt-settings`: 14px 16px 16px padding, 1px `--border` border, `--radius` corners, `--bg` background. `.wt-settings-head` is a flex row with the heading on the left and the close button on the right, vertically centered. The `h2` is 1.1rem with no margin. Each `.wt-group` has a 14px top margin; every group after the first also has a 1px `--border` rule above it and 14px top padding. Group `h3`s: 0.8rem, weight 600, uppercase, letter spacing 0.04em, `--muted`. `.wt-field` and `.wt-sound` rows are flex rows with space between, a 12px gap and 4px vertical padding. `.wt-num` holds the input and its unit ("min", or "sec" for the break length) in `--muted`, 6px apart. The inputs: 3.5em wide (room for three digits, with no arrows to make room for), 5px × 8px padding, 1px `--border` border, 8px radius, `--surface` background, `--text` color, right-aligned text; disabled ones at 50% opacity. The checkbox and the radio buttons (`.wt-check input`) use `accent-color: var(--accent)`. `.wt-plays` is a flex row with a 6px gap that doesn't shrink. Each `.wt-play` is a small pill: an inline flex row, vertically centered, with a 5px gap, 2px 10px 2px 9px padding, a 1px `--border` border, `--surface` background and 0.85rem text. Its triangle is `--accent`, and its border turns `--accent` on hover. `.wt-hint`: `--muted`, 0.85rem, 14px top margin.
- `.wt-away`: a centered flex column with a 12px gap, 32px padding above and below, and centered `--muted` text; its `p` has no margin. `.wt-back` is a small pill: 5px × 14px padding, 1px `--border` border, no background, `--text` color and 0.875rem text, with the border turning `--accent` on hover.
- **Floating window:** `#work-timer.is-floating` is fixed to fill the window (`inset: 0`), with `overflow: hidden`, the same soft background as the page's card and `container: wt-float / size`, so the container queries below, and the `cqw`/`cqh` units that size the ring's box, measure the whole window. Its `.wt-main` stops being a container (`container-type: normal`), takes the full height with 8px padding and a 10px gap, and centers its content vertically. At every size:
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
- In the smaller floating window layouts, where the Work/Rest tabs are hidden, the name inside the ring (`#wt-ring-label`) says which timer is on show, or "Break". Only the one-line strip (100px tall or less) has no name. There, only the colors (the window's background, the ring and the running time) tell Work, Rest and a break apart.
- Breaks are announced through `#wt-live`: "Short break" when one starts and "Back to work" when it ends by itself (see Random breaks).

#### Known limits

- Phones usually suspend a page when the screen locks or the browser goes to the background, so sounds can't play then. The time is still right on return. "Keep screen on" prevents the screen lock.
- On an iPhone, silent mode may mute the sounds.
- In a browser without Web Audio, the timers work silently, and breaks still show on screen.
- The recordings are AAC, which current Chrome, Edge, Safari and Firefox decode (Firefox through the operating system's decoders, so on Linux it needs the system's FFmpeg). A browser that can't decode a recording plays that moment's ding instead.
- Some browsers, Safari in particular, may not change a page's tab icon after it has loaded, so the green dot may not show there. The tab title still changes.
- The floating window needs the Document Picture-in-Picture API: desktop Chrome and Edge 116 or later, and Firefox 151 or later. Safari and phone browsers don't have it, so its button doesn't appear there. The browser decides where the window opens and sets its own smallest and largest sizes.

### 5.2 Exercise

A routine of exercises to follow along with. A drop-down picks the routine, and Start runs it from beginning to end with no further interaction. A 5-second get-ready countdown comes first, and a 3-second one before every later step. A hold counts down its seconds, and a set of reps counts up at the routine's pace. A ding marks the start and end of each step, and a softer ding marks each new rep. The current exercise's name (large) and description sit beside the ring. A list of the routine's exercises, with a progress bar down its side, shows the way through, and while paused a click on an exercise picks where to go on. The screen is kept on while the routine runs. The routines are embedded in the page as text, so it still works offline as one file. The page's own words are in Thai, like the exercises; only its name, "Exercise", is English. The page's ids and classes start with `ex-`. Its styles sit under `/* ---------- Exercise (#exercise) ---------- */` in the page-specific styles, after the Work Timer's, and its script is the second page script.

#### Markup

Inside `<section class="page" id="exercise" data-title="Exercise">`, in order:

1. `<h1>Exercise</h1>`.
2. `div.ex-pick` with `lang="th"`: `<label for="ex-routine">ชุดท่า</label>` and an empty `<select id="ex-routine">`, which the script fills with the routines.
3. `div.ex-body` with `lang="th"`, holding:
   - `div.ex-main`:
     - `div.ex-stage`, holding `div#ex-ring.ex-ring` and then `div.ex-text`.
       - The ring is built like the Work Timer's: an inline `svg.ex-ring-svg` with `viewBox="0 0 100 100"` and `aria-hidden="true"`, holding `circle.ex-ring-track` (center 50, 50, radius 45) and `path#ex-ring-arc.ex-ring-arc` with `d="M50 5a45 45 0 1 1 0 90a45 45 0 1 1 0-90z"`; then `div.ex-ring-text` holding `div#ex-count.ex-count` (`role="timer"`) and `div#ex-ring-label.ex-ring-label`.
       - `div.ex-text` holds `p#ex-meta.ex-meta`, `h2#ex-name.ex-name` and `p#ex-desc.ex-desc`.
     - `p#ex-info.ex-info`.
     - `div.ex-controls` with two buttons: `#ex-start.ex-btn.ex-btn-primary` "เริ่ม" and `#ex-pause.ex-btn` "หยุดชั่วคราว", starting disabled.
     - `p#ex-live.ex-sr` (`role="status"`), a visually hidden live region for announcements.
   - `nav#ex-list.ex-list` with `aria-label="ลำดับท่า"`, which the script fills (see Exercise list).

The script fills in every text.

#### Routines

- **Files:** each routine is a plain-text file (UTF-8) in `exercise_routines/`, read as follows:
  - The first line is the routine's title, shown in the drop-down.
  - The lines before the first line that starts with `# ` are notes for people (the file's header) and are skipped.
  - A `# ` line starts a section, named by the rest of the line. The exercises after it belong to it.
  - Every other non-blank line is an exercise: `name | description | timing`, split at `|` with the spaces around each field trimmed. It needs exactly three fields and a name.
  - The timing, in any case: first `hold N s` (N seconds, decimals allowed) or `N reps x S s` (N whole reps of S seconds each; `rep` and `×` also work). Then, separated by commas, any of: `alternating` (each movement one way counts as one rep), `each side` (done on one side, then the other), `right first` or `left first` (which side comes first; right if neither), and `K times` (done K times, as sets). With both `each side` and `K times`, each set does both sides.
  - A line that can't be read is skipped, with the console warning `Exercise: skipped a line of "<title>": <line>`. A routine with no exercises isn't listed.
- **Embedding:** the page never reads `exercise_routines/`. Each routine is a `<script type="text/plain" class="ex-routine">` element under the comment headed `Exercise routines` (see 2), holding the file's whole text as it is, from the line after the opening tag. Its `data-file` is the file's name without `.txt`, and is the routine's id. The `text/plain` type keeps the browser from running it, and the text must not contain `</script`. The drop-down lists the routines in the order of their elements.
- **Adding or changing a routine:** add or edit its file in `exercise_routines/`, then paste the file's whole text into its element (adding an element for a new one). After renaming a file, change its element's `data-file` to the new name.
- The elements come after the page script, so the script reads them on `DOMContentLoaded`, or at once if the page has already loaded.
- **The routines:** four, all listed from videos by หมอชวนฟิต [DeDoctor] (each file's header gives its video's link). Their elements come in this order, so the drop-down lists them in it, and the first is shown when no choice is saved:

| `data-file` | Title (drop-down) | Sections | Exercises | Steps | Exercise time | In all | Dings |
|---|---|---|---|---|---|---|---|
| `DD_neck_shoulder` | หมอชวนฟิต แก้ปวดคอบ่าไหล่ | วอร์มอัพ, ยืดกล้ามเนื้อ, เสริมความแข็งแรง | 13 | 19 | 332 s | 6:31 | 74 |
| `DD_neck_pain_no_drug_1` | หมอชวนฟิต ปวดคอไม่ง้อยา 1 | หายใจ, ขยับกล้ามเนื้อ, หายใจผ่อนคลาย | 11 | 11 | 406 s | 7:21 | 121 |
| `DD_neck_pain_no_drug_2` | หมอชวนฟิต ปวดคอไม่ง้อยา 2 | วอร์มอัพ, ยืดกล้ามเนื้อ, เสริมความแข็งแรง, คูลดาวน์ | 15 | 21 | 712 s | 12:57 | 169 |
| `DD_neck_pain_no_drug_3` | หมอชวนฟิต ปวดคอไม่ง้อยา 3 | วอร์มอัพ, คาร์ดิโอ, คูลดาวน์ | 23 | 32 | 1021.2 s | 18:39 | 533 |

  "In all" is the total time from Start at the first exercise (see Steps and timing), and "Dings" the number of dings that Start schedules for the whole routine (see Sounds).

#### Steps and timing

- A routine runs as steps: one per set and side of each exercise, in order (set by set, and in each set the first side, then the other). A hold's step lasts its hold time, and a reps step lasts the number of reps times the seconds per rep.
- Before each step comes a get-ready countdown: 5 seconds before the first step after Start, and 3 seconds before every other step, including one picked from the list while paused.
- The total time from a starting step is 5 seconds, plus every step from there, plus 3 seconds before each step after the first. For example, the neck and shoulder routine (`DD_neck_shoulder.txt`) has 13 exercises in 19 steps, with 332 seconds of exercise: 6:31 in all. The other routines' totals are in the table under Routines.

#### Running

- The routine is idle (not started), running, paused or done.
- **Start** (เริ่ม, when idle or done) runs the routine from the starting exercise to the end: the first exercise, or the one picked from the list. The 5-second countdown comes first.
- **Pause** (หยุดชั่วคราว, only while running) freezes the routine where it is: mid-countdown, mid-hold or mid-count. Dings that haven't played are cancelled, and one already ringing rings out.
- **Resume:** while paused, the Start button reads ทำต่อ and carries on from where the routine paused, at once, with no countdown and no ding.
- **Picking from the list** (while not running): a click on an exercise moves to the start of its first step's get-ready countdown. While paused, Resume then gives the 3-second countdown and starts that exercise from its beginning (a click on the current exercise starts it over). While idle or done, the exercise becomes the starting exercise, and Start gives the usual 5-second countdown.
- **Done:** when the last step ends. Start then runs the routine again from the first exercise.
- **Changing the routine** (the drop-down, disabled while running) stops the routine silently, cancelling its dings, and shows the new routine, idle, at its first exercise.
- **How it's timed:** Start and Resume plan the rest of the routine: every part still to come (a get-ready countdown or a step), with its start and end times in `performance.now()` milliseconds, counted from the current position. Pause stores the position: the step, the part (countdown or step), the seconds into it, and the countdown's length. While running, the display is redrawn on every animation frame, so a count changes in step with its ding. Animation frames stop in a background tab, so a 500 ms interval (`tick`) also runs: it wakes the audio context if needed, keeps the dings in step (see Sounds), announces each new get-ready countdown, and ends the routine. Both are safe to call at any time.

#### Display

- **Ring:** built like the Work Timer's (see 5.1, Ring), 200px square. Its arc shows the share of the current part left: full when a countdown or step begins, running back toward the top as it goes. It is drawn with `stroke-dasharray` the same way (C = 2π × 45), written only when it changes. The script sets one of these classes on `#ex-ring`:
  - `is-idle`: full and blue.
  - `is-ready`: amber, during a get-ready countdown.
  - `is-work`: blue, during a step.
  - `is-paused`, added to `is-ready` or `is-work` while paused: grey, with the track `--border`, the arc `--muted` and the count `--muted`.
  - `is-done`: full and blue.
- **Count and label:** the count in the middle and the label under it:

| State | Count | Label |
|---|---|---|
| Idle | The total time from the starting exercise, `m:ss` | เวลารวม |
| Get-ready countdown | The seconds left, rounded up (5 … 1, or 3 … 1) | เตรียมตัว |
| Hold | The seconds left, rounded up (15 … 1) | วินาที |
| Reps | The rep in progress, from 1 to the number of reps, moving on every S seconds | จาก N (of N) |
| Done | ✓ | ครบแล้ว |

  While paused, they show the paused moment.
- **Meta line** (`#ex-meta`): tags joined with " · ". First comes a tag for the state, if any: ท่าแรก (first exercise) or เริ่มที่ท่านี้ (start here) when idle, or ต่อไป (next) during a countdown. Then come the side (ข้างขวา or ข้างซ้าย) for an exercise done on each side, and the set (รอบที่ 1 จาก 2) for one done more than once. Last comes the hold (ค้างไว้ 15 วินาที) or the reps: 10 ครั้ง, then สลับไปมา if alternating, then ครั้งละ 2.4 วินาที. Seconds show with at most one decimal.
- **Name and description** (`#ex-name`, `#ex-desc`): the exercise of the step on show. During a countdown, that is the coming step. When done, the name is ครบทุกท่าแล้ว (all done) and the description is the routine's title.
- **Info line** (`#ex-info`): "กดเริ่ม แล้วทำตามได้เลย หรือแตะชื่อท่าในรายการเพื่อเริ่มจากท่านั้น" when idle, "หยุดชั่วคราว · แตะชื่อท่าในรายการเพื่อเลือกท่าที่จะทำต่อ" when paused, "กดเริ่มเพื่อทำอีกรอบ" when done, and empty while running. It keeps its height when empty.
- **Buttons:** Start is enabled unless the routine runs, and reads ทำต่อ while paused and เริ่ม otherwise. Pause is enabled only while it runs. When a button disables itself, focus moves to the other one.
- Texts and attributes are written only when they change.
- The page doesn't change the browser tab's title. The Work Timer's countdown still takes it while that timer runs (see 3.2).

#### Exercise list

- `nav#ex-list` holds `p.ex-list-head`, which gives the number of exercises and the total time from the first, for example "13 ท่า · เวลารวม 6:31 นาที". Then, for each section, `p.ex-group` with the section's name, and `ol.ex-items` with one `li` per exercise. Each `li` holds `button.ex-item` (`data-ex`, the exercise's index), which holds `span.ex-bar` (holding `span.ex-fill`) and `span.ex-item-name`.
- **Progress bar:** the `span.ex-bar` segments, one per exercise, make up a bar down the list's side, broken only between sections. Each segment fills from the top with the share of its exercise's step time done (`transform: scaleY(share)`, with 3 decimals, written when it changes). Countdowns don't count. The bar shows the position in the routine: exercises before the current one are full, even ones skipped by picking a later one, and later ones are empty. Idle, all are empty. Done, all are full.
- **Classes:** `is-done` on the exercises before the current one (all of them when done), and `is-current` on the exercise on show, which also gets `aria-current="step"`. Idle, the current one is the starting exercise.
- The buttons are disabled while the routine runs, and a click while it isn't running picks where to go on (see Running).

#### Sounds

The dings are generated with the Web Audio API, in the Work Timer's soft bell: the same four sine partials, attack and fade (see 5.1, Dings). There are three:

| Ding | Notes: start, pitch, level, fade time | Plays |
|---|---|---|
| Start | 0 s, A5 (880 Hz), 0.5, 1.2 s | When a step starts |
| End | 0 s, A5 (880 Hz), 0.45, 1 s; then 0.18 s, E5 (659.25 Hz), 0.45, 1.6 s | When a step ends |
| Rep | 0 s, A5 (880 Hz), 0.16, 0.45 s | When each rep after the first starts |

- So a single ding starts a step, a falling pair ends it, and the rep ding is the start ding, much quieter and shorter. The countdowns are silent, and Resume makes no sound.
- **Playback:** the page has its own `AudioContext`, created by the first Start (browsers allow audio only after a user gesture), with a master gain of 0.9. Each ding gets its own gain node (its bus) into the master, disconnected when all its oscillators have ended or when it's cancelled.
- **Scheduling:** Start and Resume schedule every ding of the rest of the routine ahead on the audio clock (a whole routine has from 74 to 533, see the table under Routines), so they play on time even in a background tab. The plan's `performance.now()` times become audio-clock times through a pair of matching readings (`ctx.currentTime` and `performance.now()`) taken when they're scheduled. If the context wasn't running yet, they are scheduled again as soon as it runs. Pause and a routine change cancel the dings that haven't started, and one already ringing rings out.
- **Staying in step:** on every tick, if the audio clock has drifted 0.15 seconds or more from that pair (it stalls while a computer sleeps, for example), the dings still ahead are scheduled again. Those whose time has passed are dropped.
- The audio context is suspended 5 seconds after the routine stops (Pause, or the end), once no ding is still ringing. If one is, it checks again every 5 seconds. The next Start or Resume wakes it.

#### Keeping the screen on

- While the routine runs, the page asks for a screen wake lock (Screen Wake Lock API), with no button, so the routine can be followed without touching the device. It lets go on Pause and at the end.
- Browsers drop the lock while the page is hidden, so it is asked for again when the page is shown while the routine runs.
- If the API is missing or the request is refused, nothing shows and the routine runs anyway. This is separate from the Work Timer's Keep screen on button.

#### Saved settings

- The chosen routine is saved to `localStorage` under `misc-utils:exercise` when the drop-down changes, as JSON like `{"routine":"DD_neck_shoulder"}`, the routine's id (its `data-file`). On load, it is used if it names a listed routine. Otherwise the first routine is shown, as happens to a choice saved before its routine's file was renamed.

#### Styles

Every rule is scoped under `#exercise`. The page's colors are custom properties on `#exercise`, with dark values in its own `prefers-color-scheme: dark` media query:

| Variable | Light | Dark | Used for |
|---|---|---|---|
| `--ex-work` | `#2f6fde` | `#5b8ef0` | The ring's arc during a step, idle and done, the Start button, the buttons' hover border and the progress bar's fill (the same blue as `--accent`) |
| `--ex-work-pale` | `#c1d4f5` | `#2e4266` | The ring's track with the blue arc (the Work Timer's work track) |
| `--ex-ready` | `#a86500` | `#e0a040` | The ring's arc during a get-ready countdown: amber |
| `--ex-ready-pale` | `#eee0cc` | `#4a3a1e` | Its track: in light, the amber mixed 20% into white; in dark, a dim amber |

- Contrast: the amber arc is 3.6:1 against its track in light and 4.8:1 in dark, and 4.6:1 and 7.1:1 against the card. The blue arc and track are the Work Timer's (at least 3.1:1). The progress bar's fill is 3.6:1 against its `--border` track in light and 3.9:1 in dark. The count and text use `--text` and `--muted`.
- `[hidden]` inside the page is forced to `display: none`. Buttons and the drop-down inherit the font and color, enabled buttons have a pointer cursor, and focus shows a 2px `--accent` outline with a 2px offset.
- `.ex-pick` is a wrapping flex row with an 8px × 12px gap and a 20px bottom margin. The label is `--muted`. The drop-down has 6px × 10px padding, a 1px `--border` border, 8px radius and a `--surface` background, at 60% opacity when disabled.
- `.ex-body` is a grid with a 24px gap. Wider than 720px, it has two columns, `minmax(0, 1fr) 240px`, with the list beside the routine. At 720px or narrower, the list goes below.
- `.ex-main` is a centered flex column with a 16px gap and is a container named `ex-main` (`inline-size`).
- `.ex-stage` is a flex row, vertically centered, with a 24px gap, full width. `.ex-text` takes the rest of the row (`flex: 1 1 0`, `min-width: 0`). When `.ex-main` is 520px wide or less (`@container ex-main`), the stage becomes a column with a 16px gap and centered text, the ring is 180px, the name 1.6rem and the description 1rem.
- `.ex-ring`, like the Work Timer's: a size container and a one-cell grid (`grid-template: minmax(0, 1fr) / minmax(0, 1fr)`), 200px square, not shrinking. Both children sit in the cell, centered. The SVG fills it (`overflow: visible`), and the track and arc have no fill and a stroke width of 6. `.ex-count`: `font-size: 30cqmin` (`20cqmin` when idle, for the `m:ss` total), weight 600, `font-variant-numeric: tabular-nums`, letter spacing −0.02em, line height 1.1. `.ex-ring-label`: `--muted`, `font-size: max(12px, 8cqmin)`, line height 1.4.
- `.ex-meta`: `--muted`, 0.95rem, 6px below. `.ex-name`: 2rem, line height 1.35, no margin. `.ex-desc`: 1.1rem, 8px above.
- `.ex-info`: `--muted`, 0.95rem, centered, no margin, at least 1.5em tall.
- `.ex-controls` is a flex row with an 8px gap, full width up to 360px. `.ex-btn` is styled like the Work Timer's `.wt-btn`: equal widths, 9px × 10px padding, pill-shaped, a 1px `--border` border, `--surface` background, weight 600, the border turning `--ex-work` on hover, and 45% opacity when disabled. `.ex-btn-primary` has an `--ex-work` background and border and `--accent-text` text.
- **List:** `.ex-list` is 0.85rem. `.ex-list-head` is `--muted` with no margin. `.ex-group` is `--muted`, weight 600, with a 14px top margin and 4px below. `.ex-items` has no margin, padding or list markers. `.ex-item` is a full-width flex row (items stretched, 10px gap) with no padding, border or background, and left-aligned text. `.ex-bar` is 4px wide with a `--border` background and `overflow: hidden`, and reaches 1px into the next row (`margin-bottom: -1px`), so no seam shows between segments. Its ends are rounded (2px) at the first and last row of each section. `.ex-fill` fills the segment (`position: absolute; inset: 0`) in `--ex-work`, scaled from the top (`transform-origin: top`). `.ex-item-name` has 3px padding above and below and a 20px line height, so every row is a whole 26px tall. Done exercises are `--muted` and the current one is weight 600. While the buttons are enabled, hovering one turns its name `--accent` and underlines it.
- `.ex-sr` is visually hidden, like the Work Timer's `.wt-sr`.

#### Accessibility

- The Thai parts of the page have `lang="th"`, so screen readers read them with a Thai voice. The `h1` stays English.
- `role="timer"` on the count keeps screen readers from reading out every change. The ring's SVG is hidden from them.
- `#ex-live` announces "เตรียมตัว: " and the coming exercise's name (and its side, if any) when each get-ready countdown starts, and "จบชุดท่าแล้ว" (the routine is done) at the end. The live region is emptied first and filled 100 ms later, so a repeated message is read again. The dings mark the start and end of each step.
- The list's exercises are buttons, so they work from the keyboard, and the one on show has `aria-current="step"`.

#### Known limits

- Phones usually suspend a page when the screen locks or the browser goes to the background, so the routine can only be followed with the page on screen. The page keeps the screen on while the routine runs, where the browser allows it. On an iPhone, silent mode may mute the dings.
- In a background tab, the dings stay on time, and the display catches up when the page is shown again. The routine's end registers when the slowed-down interval next runs.
- Delays in the audio output, such as Bluetooth headphones' (often 0.1 to 0.3 seconds), make each ding come a little after its count changes.
- In a browser without Web Audio, the routine runs silently.
- The dings scheduled ahead cost CPU time until they play, so a long routine costs more, most of all at its start. Start on ปวดคอไม่ง้อยา 3 schedules 533 dings: 2,260 oscillators, each with its own gain node. Measured in headless Firefox 161 on an Apple Silicon Mac, with the page in a background tab so that nothing was redrawn, Firefox used about 38% of a CPU core in the routine's first seconds, against about 15% for the neck and shoulder routine (74 dings) and about 1% with no routine running. The cost falls as the dings play out. Scheduling the dings a limited time ahead, topped up as the routine runs, would bound it.

### 5.3 Eye Exercise

Plays an eye exercise recording five times in a row, with no break between plays, then rings a big ding. Start, Pause and Reset control it. A ring like the Work Timer's shows which play is on ("2 of 5") and how much of it is left. The page shows no other text: no description or instructions. The recording is embedded in the page, so it still works offline as one file. The page's words are in English. The page's ids and classes start with `ee-`. Its styles sit under `/* ---------- Eye Exercise (#eye-exercise) ---------- */` in the page-specific styles, after the Exercise page's, and its script is the third page script.

#### Markup

Inside `<section class="page" id="eye-exercise" data-title="Eye Exercise">`, in order:

1. `<h1>Eye Exercise</h1>`.
2. `div.ee-main`, holding:
   - `div#ee-ring.ee-ring`, built like the Work Timer's ring: an inline `svg.ee-ring-svg` with `viewBox="0 0 100 100"` and `aria-hidden="true"`, holding `circle.ee-ring-track` (center 50, 50, radius 45) and `path#ee-ring-arc.ee-ring-arc` with `d="M50 5a45 45 0 1 1 0 90a45 45 0 1 1 0-90z"`; then `div.ee-ring-text` holding `div#ee-count.ee-count` (`role="timer"`) and `div#ee-ring-label.ee-ring-label`, which the script fills.
   - `div.ee-controls` with three buttons, all starting disabled: `#ee-start.ee-btn.ee-btn-primary` "Start", `#ee-pause.ee-btn` "Pause" and `#ee-reset.ee-btn` "Reset".
   - `p#ee-note.ee-note` (`role="status"`), empty unless the recording can't be played.
   - `p#ee-live.ee-sr` (`role="status"`), a visually hidden live region for announcements.

#### Recording

- **File:** `eye_exercise/eye_exercise.m4a`, AAC in an `.m4a` container, 48 kHz mono, 15.79 s, 136 KB. The voice starts about 1 s in and ends about 1.4 s before the end of the file. Those quiet ends stay, as part of each play.
- **Embedding:** the app never reads `eye_exercise/`. The recording is a `<script type="text/plain" class="ee-recording">` element at the very end of `<body>`, after the Work Timer's recordings (see 2). Its text is the file's bytes in base64, on one line (about 181 KB). The `text/plain` type keeps the browser from running it.
- **Decoding:** when the page is first shown (on load, or on the first `hashchange` that shows it), the script decodes the recording, once for the page's lifetime. `atob` turns the element's text into bytes, and `decodeAudioData` on a throwaway `OfflineAudioContext` (1 channel, 1 frame, 48 kHz, never rendered) turns those into an `AudioBuffer`. An offline context needs no user gesture, so the total time shows before Start. Decoding takes a fraction of a second (about 0.07 s in Chrome on a Mac). The buffer then plays in the page's own `AudioContext`: an `AudioBuffer` isn't tied to the context that made it, and if the audio device runs at another rate, the browser converts it as it plays. The routine's length comes from the decoded buffer.
- **Can't play:** if the browser has no `OfflineAudioContext` or `AudioContext`, the element is missing, or decoding fails, the ring turns grey and empty (`is-off`), Start stays disabled, and `#ee-note` says "This browser can’t play the recording."
- **Changing the recording:** replace `eye_exercise/eye_exercise.m4a`, then put the new file's base64 (for example from `base64 -i eye_exercise/eye_exercise.m4a`) in the element, as one line. The script reads the new length from the recording, so nothing else needs to change. AAC (`.m4a`) and MP3 are safe formats: every target browser decodes them.

#### Running

- **The routine:** the recording plays 5 times back to back, with no break between plays: each play starts the moment the one before it ends. The recording's own quiet ends stay, so about 2.4 seconds pass between the voice of one play and the next. With this recording the routine lasts about 79 seconds (5 × 15.8), shown as 1:19. Chrome decodes the file as 15.83 s, as it keeps the AAC encoder's 44 ms of priming at its start. The 5 plays are a constant in the script (`PLAYS`), not a setting.
- When the last play ends (at the end of the file), the big ding rings and the routine is done.
- The routine is idle (not started), running, paused or done.
- **Start** (when idle or done) runs the routine from the first play, at once (50 ms after the click, `LEAD`).
- **Pause** (only while running) stops the routine where it is, even partway through a play. A play in progress fades out over about 30 ms, so it doesn't click, and the plays and the big ding still to come are cancelled.
- **Resume:** while paused, the Start button reads Resume and carries on from where the routine paused, at once. The play picks up at the same point in the recording, fading in over 15 ms (`FADE`), and the plays after it follow back to back.
- **Reset** (while running or paused) stops the routine silently, as Pause does, and puts it back to the start, idle.
- **Done:** when the last play ends. Start then runs the routine again from the first play.
- Nothing is saved: the routine is idle on every load.
- **How it's timed:** the routine's time is kept on the audio clock (`ctx.currentTime`), so the display always matches what plays. If the audio starts a moment late, or its clock stalls (while a computer sleeps, for example), the display waits with it. Start and Resume schedule the rest of the routine ahead on the audio clock: each play still to come through its own `AudioBufferSourceNode` (`start(time)`, or `start(time, offset)` for the play resumed partway), and the big ding at the end. So everything plays on time even in a background tab. While the routine runs, it is `ctx.currentTime − origin` seconds in, where Start and Resume set `origin` (the moment the routine started, or would have started). It never shows a moment before the one it started or resumed from (`at`). While it doesn't run, `at` holds where it stands: 0 when idle, or the paused moment. Play *n* (counting from 0) is scheduled at `origin + n × L`, where L is the recording's length, so each play starts exactly as the one before it ends, and the big ding at `origin + 5 × L`.
- **Redrawing:** while running, the display is redrawn on every animation frame, so the arc moves smoothly. Animation frames stop in a background tab, so a 500 ms interval (`tick`) also runs: it wakes the audio context if it was suspended, and ends the routine on time. Both are safe to call at any time.

#### Display

- **Ring:** built like the Work Timer's (see 5.1, Ring), `min(300px, 100cqi)` square. Its arc shows the share left of the play in progress: full when the play begins and running back toward the top as it goes, then full again as the next play begins. It is drawn with `stroke-dasharray` the same way (C = 2π × 45), written only when it changes. The script sets one of these classes on `#ee-ring`:
  - `is-idle`: full and blue.
  - `is-play`: blue, while the recording plays.
  - `is-paused`, added to `is-play` while paused: grey, with the track `--border`, the arc `--muted` and the count `--muted`.
  - `is-done`: full and blue.
  - `is-off`: grey and empty, when the recording can't be played.
- **Count and label:** the count in the middle and the label under it:

| State | Count | Label |
|---|---|---|
| Idle | The routine's total time, `m:ss` (1:19) | Total time |
| A play | Its number, 1 to 5 | of 5 |
| Done | ✓ | Done |

  While paused, they show the paused moment. Until the recording is decoded, both are empty.
- The page shows no other text while it works: no description, instructions or info line. The only other text is `#ee-note`, when the recording can't be played.
- **Buttons:** Start is enabled once the recording is decoded, unless the routine runs. It reads Resume while paused and Start otherwise. Pause is enabled only while the routine runs, and Reset while it runs or is paused.
- Texts and attributes are written only when they change.
- The page doesn't change the browser tab's title. The Work Timer's countdown still takes it while that timer runs (see 3.2).

#### Sounds

- **The recording** plays at its own level, through a master gain of 0.9. It peaks at about −11 dBFS, and its loudest 400 ms average about −26 dBFS, so it is a quiet recording, 7 to 12 dB quieter than the Work Timer's.
- **The big ding** is generated with the Web Audio API in the Work Timer's soft bell: the same four sine partials, attack and fade (see 5.1, Dings). It is two notes struck together:

| Ding | Notes: start, pitch, level, fade time | Plays |
|---|---|---|
| Big ding | 0 s, A4 (440 Hz), 0.25, 5 s; and 0 s, A5 (880 Hz), 0.125, 3 s | When the last play ends |

  - So it is lower and longer than the other pages' dings, and rings for about 5 seconds. Its level is set from the recording rather than from the other dings. It peaks at about −8 dBFS, and its loudest 400 ms are about 10 dB louder than the recording's, so it stands out clearly from the voice without being harsh. At the other dings' levels (0.5 for its main note) it would be about 16 dB louder than the voice.
- **Playback:** the page has its own `AudioContext`, created by the first Start (browsers allow audio only after a user gesture), with a master gain of 0.9. Each play and the big ding get their own gain node (bus) into the master. A play resumed partway fades in on its bus. A play stopped by Pause or Reset fades out on it, with `setTargetAtTime` to 0 and a 5 ms time constant, and its source stops 30 ms later. Sounds still to come are stopped before they start. Each bus is disconnected when all of its sources have ended or the sound is cancelled. The script keeps the set of sounds scheduled or playing.
- The audio context is suspended 5 seconds after the routine stops (Pause, Reset or the end), once no sound is still playing. While the big ding rings out, it checks again every 5 seconds. The next Start or Resume wakes it. Since the routine's time is on the audio clock, nothing is lost while the audio wakes.

#### Keeping the screen on

- As on the Exercise page (5.2), while the routine runs, the page asks for a screen wake lock, with no button. It asks again when the page is shown again while the routine runs, and lets go on Pause, on Reset and at the end. If the API is missing or the request is refused, nothing shows and the routine runs anyway.

#### Styles

Every rule is scoped under `#eye-exercise`. The page's colors are custom properties on `#eye-exercise`, with dark values in its own `prefers-color-scheme: dark` media query. They are the Work Timer's work shades:

| Variable | Light | Dark | Used for |
|---|---|---|---|
| `--ee-play` | `#2f6fde` | `#5b8ef0` | The ring's arc while a play is on, idle and done, the Start button and the buttons' hover border (the same blue as `--accent`) |
| `--ee-play-pale` | `#c1d4f5` | `#2e4266` | The ring's track under the blue arc |

- Contrast, light and dark: the blue arc is 3.15:1 against its track in both, and 4.72:1 and 5.05:1 against the card. The grey arc (paused) is 4.13:1 and 4.97:1 against its track. The Start button's `--accent-text` label is 4.72:1 and 5.92:1 on the blue.
- `[hidden]` inside the page is forced to `display: none`. Buttons inherit the font and color, enabled buttons have a pointer cursor, and focus shows a 2px `--accent` outline with a 2px offset.
- `.ee-main` is a centered flex column with a 16px gap and `container-type: inline-size`.
- `.ee-ring`, like the Work Timer's: a size container and a one-cell grid (`grid-template: minmax(0, 1fr) / minmax(0, 1fr)`), `min(300px, 100cqi)` on each side (`cqi` here is the width of `.ee-main`). Both children sit in the cell, centered. The SVG fills it (`overflow: visible`), and the track and arc have no fill and a stroke width of 6.
- `.ee-ring-text` is a centered flex column. `.ee-count`: `font-size: 30cqmin` (`22cqmin` when idle, for the `m:ss` total), weight 600, `font-variant-numeric: tabular-nums`, letter spacing −0.02em, line height 1.1. `.ee-ring-label`: `--muted`, `font-size: max(12px, 7cqmin)`, line height 1.3. At the full 300px, the count is 90px (66px when idle) and the label 21px.
- `.ee-controls` is a flex row with an 8px gap, full width up to 360px, and an 8px top margin. `.ee-btn` is styled like the Work Timer's `.wt-btn`: equal widths (`flex: 1 1 0`), 9px × 10px padding, pill-shaped, a 1px `--border` border, `--surface` background, weight 600, the border turning `--ee-play` on hover, and 45% opacity when disabled. `.ee-btn-primary` has an `--ee-play` background and border and `--accent-text` text.
- `.ee-note`: `--muted`, 0.85rem, centered, no margin, and not shown when empty.
- `.ee-sr` is visually hidden, like the Work Timer's `.wt-sr`.

#### Accessibility

- `role="timer"` on the count keeps screen readers from reading out every change. The ring's SVG is hidden from them.
- `#ee-live` announces "Eye exercise done" at the end. It is emptied first and filled 100 ms later, so a repeated message is read again. Nothing is announced while the routine runs, so a screen reader doesn't talk over the recording.
- When Start, Pause or Reset disables the button that had focus, focus moves on: from Start to Pause, from Pause to Start, and from Reset to Start. When the routine ends by itself, focus on Pause or Reset moves to Start.

#### Known limits

- Phones usually suspend a page when the screen locks or the browser goes to the background, and the recording stops with it. Since the routine's time is on the audio clock, the routine doesn't run on in the meantime. If the audio doesn't come back by itself on return, Pause and then Resume start it again. The page keeps the screen on while the routine runs, where the browser allows it. On an iPhone, silent mode may mute the recording and the ding.
- In a background tab on a computer, the recording and the ding stay on time, and the display catches up when the page is shown again. The routine's end registers when the slowed-down interval next runs.
- Delays in the audio output, such as Bluetooth headphones' (often 0.1 to 0.3 seconds), make the display run a little ahead of what's heard.
- The recording is AAC, which current Chrome, Edge, Safari and Firefox decode (Firefox through the operating system's decoders, so on Linux it needs the system's FFmpeg). In a browser that can't decode it, or has no Web Audio, the page can't play it (see Recording).

### 5.4 Breathing

Breathing exercises, one per tab: box breathing and 4-7-8 breathing. A session repeats the exercise's cycle (inhale, hold, exhale and so on) for a chosen length, rounded up to whole cycles. A ring shows one cycle, a stretch for each part in its kind's color (blue to inhale, yellow to hold, green to exhale), and the part in progress fills as it goes. A ding marks every second: a loud one at the start of each part and a quiet one at each second after, each kind of part with its own note. A 3-second get-ready countdown comes before the first inhale, and again on Resume, which starts the interrupted cycle again. A closing chime ends the session. Start, Pause and Reset control it. The sounds are generated, so nothing is embedded. The page's words are in English. The page's ids and classes start with `br-`. Its styles sit under `/* ---------- Breathing (#breathing) ---------- */` in the page-specific styles, after the Eye Exercise's, and its script is the fourth page script.

#### Markup

Inside `<section class="page" id="breathing" data-title="Breathing">`, in order:

1. `<h1>Breathing</h1>`.
2. `div.br-main`, holding:
   - `div#br-tabs.br-tabs` with `role="tablist"` and `aria-label="Exercises"`, empty: the script adds a tab per exercise (see Exercises).
   - `div#br-panel.br-panel` with `role="tabpanel"`, holding:
     - `div.br-length`: `<label for="br-length">Session length</label>`, an empty `<select id="br-length">`, which the script fills (see Session length), and `span#br-other.br-other`, starting `hidden`, holding `input#br-other-min` followed by " min". The input is `<input type="text" inputmode="numeric" autocomplete="off" aria-label="Session length in minutes">`, typed into by hand like the Work Timer's settings, so it has no up/down arrows.
     - `div#br-ring.br-ring`: an inline `svg.br-ring-svg` with `viewBox="0 0 100 100"` and `aria-hidden="true"`, holding the empty groups `g#br-track.br-track` and then `g#br-fill.br-fill`, in which the script draws the parts (see Display); then `div.br-ring-text`, holding `div#br-count.br-count` (`role="timer"`) and `div#br-ring-label.br-ring-label`.
     - `ol#br-key.br-key` with `aria-label="One cycle"`, empty: the color key, which the script fills.
     - `div.br-controls` with three buttons: `#br-start.br-btn.br-btn-primary` "Start", `#br-pause.br-btn` "Pause" and `#br-reset.br-btn` "Reset", the last two starting disabled.
     - `p#br-live.br-sr` (`role="status"`), a visually hidden live region for announcements.

The script fills in every text. It needs nothing that comes after it in the file, so it runs at once.

#### Exercises

- The exercises are a list in the script (`EXERCISES`), one tab each, in its order. Each has an `id`, a `name` (its tab's label) and `parts`, its cycle: `[kind, seconds]` pairs in order. The kind is `inhale`, `hold` or `exhale`, and the seconds are a whole number, at least 1, since a ding marks every second.

| id | Tab | Cycle | Cycle length |
|---|---|---|---|
| `box` | Box breathing | inhale 5 s, hold 5 s, exhale 5 s, hold 5 s | 20 s |
| `4-7-8` | 4-7-8 breathing | inhale 4 s, hold 7 s, exhale 8 s | 19 s |

- Each kind has its own label (Inhale, Hold, Exhale), color (see Styles) and note (see Sounds). Box breathing's two holds are the same kind.
- **Tabs:** each is a `button#br-tab-<id>` with `role="tab"`, `aria-controls="br-panel"` and the exercise's name, built like the Work Timer's Work/Rest switch. The tab on show has `aria-selected="true"` and `tabindex="0"`, the others `false` and `-1`, and the panel's `aria-labelledby` names it. On a tab, Left and Right arrows go to the previous and next tab, wrapping around, and Home and End to the first and last, showing that exercise at once.
- **Adding an exercise:** add a line to `EXERCISES`. Its tab, ring and color key come with it, and it starts with the default session length.

#### Session length

- Each exercise has its own session length, in whole minutes: one from the drop-down, or one typed in under Other. The default is 5.
- The script fills the drop-down with the lengths in `PRESETS`, 3, 5, 10, 15, 20, 25, 30 and 60 minutes (options "3 min" to "60 min", with values "3" to "60"), then "Other…" (value `other`).
- **Other:** choosing it shows the box beside the drop-down, holding the current minutes, focused and selected, ready to type over. The length doesn't change until an entry is committed. Choosing a length from the list hides the box again and applies that length. A length typed under Other stays under Other, even one that is also in the list.
- The box is checked on `change`, which fires when an entry is committed (Enter, or leaving the box), like the Work Timer's settings: an empty box (spaces only, too) or anything `Number()` can't read, such as "abc", puts the previous value back. A number is rounded to the nearest whole number and clamped to 1 to 60 (2.6 becomes 3, 0 becomes 1, 75 becomes 60). The box then shows the value in use.
- **Whole cycles:** a session always ends at the end of a cycle. It runs as many whole cycles as it takes to reach at least the chosen length: total = ⌈minutes × 60 ÷ cycle length⌉ × cycle length. Box breathing's 20-second cycle fits any whole number of minutes. 4-7-8's 19 seconds rarely does: 5 minutes is 16 cycles, 5:04, 3 minutes is 3:10 and 60 minutes 60:10.
- A new length applies at once. A session that is paused or done ends, as on Reset, so the new length shows. While a session runs, the drop-down and the box are disabled.

#### Running

- The session is idle (not started), running (the get-ready countdown, then the breathing), paused or done.
- **Start** (when idle or done) begins a 3-second get-ready countdown (`READY`) 50 ms after the click (`LEAD`), and then the session, from the first cycle's inhale.
- **Pause** (only while running) stops the session. The dings that haven't started are cancelled, and one already ringing rings out. Since Resume starts the interrupted cycle again, the session goes back to the start of the cycle in progress. Paused during a get-ready countdown, it stays where it was.
- **Resume:** while paused, the Start button reads Resume. It gives the 3-second get-ready countdown, then starts the interrupted cycle again, from its inhale. So the time left goes back to what it was when that cycle began.
- **Reset** (while running or paused) stops the session silently and puts it back to the start, idle. If the session has just reached its end, the end chime rings out.
- **Done:** when the last cycle ends, with the end chime. Start then begins a new session.
- **Tabs:** disabled while a session runs. While one is paused or done, showing another exercise ends it, as on Reset, and the other exercise shows idle.
- **How it's timed:** like the Eye Exercise (5.3), the session's time is kept on the audio clock (`ctx.currentTime`), so the ring and the count always match the dings, even if the audio starts late or its clock stalls (while a computer sleeps, for example). While running, the session is `ctx.currentTime − origin` seconds in. `from` is where it starts: 0 on Start, or on Resume the start of the cycle that was interrupted. Start and Resume set `origin` to the current time + `LEAD` + `READY` − `from`, so the get-ready countdown fills the 3 seconds before `from`. Without Web Audio, the session runs silently, on `performance.now()`.
- **Ticking:** while a session runs, the display is redrawn on every animation frame, so the arc moves smoothly. Animation frames stop in a background tab, so a 500 ms interval (`tick`) also runs, along with a chain of silent one-second `ConstantSourceNode`s (offset 0, connected to the output), as on the Work Timer (5.1, Keeping time in a background tab): their `ended` events aren't slowed down in background tabs, and each calls `tick` and starts the next. `tick` wakes the audio context if it was suspended, tops up the scheduled sounds (see Sounds) and ends the session on time. Both are safe to call at any time.

#### Display

- **Ring:** `min(300px, 100cqi)` square, a circle of radius 45 on a 100 × 100 viewBox with a stroke width of 6, like the Work Timer's (5.1, Ring), but drawn in parts. For each part of the cycle, the script adds a `path` to `g.br-track` (the part's track, in its pale shade) and one to `g.br-fill` (its arc, in the solid color), each with the class `is-<kind>`, `is-inhale` for example. Each path is the whole ring (`d="M50 5a45 45 0 1 1 0 90a45 45 0 1 1 0-90z"`, from the top and clockwise) drawn as one dash: `stroke-dasharray` is `<length> <C>` and `stroke-dashoffset` is `−<start>`, with 2 decimals, where C = 2π × 45 ≈ 282.74 (the circumference). A part's stretch starts at the share of the cycle before it × C, plus half a gap, and its length is its own share × C, less a gap (`GAP`, 1.2 units, about 1.5°). So a small gap separates the parts, and box breathing shows as four equal quarters.
- **Filling:** the arc of the part in progress is filled by the share of it done, so it grows clockwise, smoothly. The arcs of the cycle's earlier parts are full, and the later ones empty (a dash of length 0). Each cycle starts empty. Arcs are written only when they change.
- The script sets one of these classes on `#br-ring`:
  - `is-idle`: the tracks only.
  - `is-ready`: the tracks only, during a get-ready countdown.
  - `is-breathing`: while a part runs.
  - `is-paused`: grey: every track is `--border`, and the arcs are empty.
  - `is-done`: every arc full.
- **Count and label:** the count in the middle and the label under it:

| State | Count | Label |
|---|---|---|
| Idle | The session's length, `m:ss` (5:00, or 5:04 for 5 minutes of 4-7-8) | Total time |
| Get-ready countdown | The seconds left, rounded up: 3, 2, 1 | Get ready |
| A part | The session's time left, `m:ss`, rounded up | Inhale, Hold or Exhale |
| Paused | The time left from the start of the cycle that Resume starts again, `m:ss` | Paused |
| Done | ✓ | Done |

  Minutes can go past 59 (60:10). The count changes with each ding.
- **Color key** (`#br-key`): one `li` per part of the cycle, in order, with the class `is-<kind>` and text such as "Inhale 4 s", each after a dot in its kind's solid color. It spells out the cycle: "Inhale 5 s, Hold 5 s, Exhale 5 s, Hold 5 s" for box breathing.
- **Buttons:** Start is enabled unless a session runs, and reads Resume while paused and Start otherwise. Pause is enabled only while a session runs, and Reset while it runs or is paused.
- Texts and attributes are written only when they change.
- The page doesn't change the browser tab's title. The Work Timer's countdown still takes it while that timer runs (see 3.2).

#### Sounds

- Every sound is generated with the Web Audio API, in a mellow bell, softer than the other pages' (5.1, Dings): three sine partials, with no inharmonic fourth one.

| Partial | Frequency | Peak (× note level) | Fade (× note fade time) |
|---|---|---|---|
| 1 | f | 1 | 1 |
| 2 | 2f | 0.3 | 0.55 |
| 3 | 3f | 0.1 | 0.35 |

  Each partial's gain starts at 0, rises linearly to its peak in 10 ms (the other pages' bell takes 5 ms), then falls exponentially to 0.0001 by the end of its fade. Its oscillator stops 50 ms after that.
- **Dings:** one at every whole second of the session: a loud ding at the start of each part, and a quiet one, shorter, at each second after it. Each kind of part has its own note, all three from the chord of C major, high to inhale, middle to hold and low to exhale, so the parts can be told apart with eyes closed:

| Kind | Note | Loud: level, fade time | Quiet: level, fade time |
|---|---|---|---|
| Inhale | E5 (659.25 Hz) | 0.36, 2.4 s | 0.15, 0.7 s |
| Hold | G4 (392 Hz) | 0.44, 2.4 s | 0.18, 0.7 s |
| Exhale | C4 (261.63 Hz) | 0.53, 2.4 s | 0.22, 0.7 s |

  - So box breathing goes: a loud inhale ding, 4 quiet inhale dings, a loud hold ding, 4 quiet hold dings, a loud exhale ding, 4 quiet exhale dings, a loud hold ding and 4 quiet hold dings, then the next cycle. 4-7-8 goes: a loud inhale ding and 3 quiet, a loud hold ding and 6 quiet, a loud exhale ding and 7 quiet.
  - The lower notes are set higher, so the three sound about as loud: measured on their loudest 400 ms, averaging K weighting (ITU-R BS.1770) and A weighting, the three loud dings are within 0.1 dB of each other, and the three quiet ones within 0.2 dB, about 12 dB below them. The loud dings are about 1 dB softer than the Exercise page's start ding. They peak at about −8.8, −7.0 and −5.4 dBFS, after the master gain.
- **End chime:** when the last cycle ends, where the next inhale would have started, the chord of C major rolls up from C4: C4 (261.63 Hz) at 0 s, level 0.28, fade 5 s; G4 (392 Hz) at 0.09 s, 0.25, 4.5 s; C5 (523.25 Hz) at 0.18 s, 0.23, 4 s; and E5 (659.25 Hz) at 0.27 s, 0.2, 3.5 s. It rings for about 5 seconds, about 2.5 dB louder than a loud ding, and peaks at about −4.9 dBFS.
- The get-ready countdowns are silent, and so is Resume.
- **Playback:** the page has its own `AudioContext`, created by the first Start (browsers allow audio only after a user gesture), with a master gain of 0.9. Each sound gets its own gain node (its bus) into the master, disconnected when all its oscillators have ended or when it's cancelled. The script keeps the set of sounds scheduled or playing.
- **Scheduling:** the sounds are scheduled ahead on the audio clock, so they play on time even in a background tab, but only the next 6 seconds of them (`AHEAD`): Start and Resume schedule the first ones, and every tick schedules more. So however long the session, no more than about 7 dings wait at once, each 3 oscillators with their gain nodes. A sound whose time has passed by more than 0.1 s (if the ticks were held up) is skipped rather than played late. Pause and Reset cancel the ones that haven't started.
- The audio context is suspended 5 seconds after the session stops (Pause, Reset or the end), once no sound is still ringing. If one is, it checks again every 5 seconds. The next Start or Resume wakes it. Since the session's time is on the audio clock, nothing is lost while the audio wakes.

#### Keeping the screen on

- As on the Exercise page (5.2), while a session runs, the page asks for a screen wake lock, with no button. It asks again when the page is shown again while a session runs, and lets go on Pause, on Reset and at the end. If the API is missing or the request is refused, nothing shows and the session runs anyway.

#### Saved settings

- The exercise on show and each exercise's session length are saved to `localStorage` under `misc-utils:breathing`, on a tab switch and on a length change, as JSON like `{"exercise":"box","sessions":{"box":{"min":5,"other":false},"4-7-8":{"min":7,"other":true}}}`: the id of the exercise on show, and for each exercise its minutes (`min`) and whether they are under Other (`other`).
- On load, `exercise` is used if it names an exercise, otherwise the first is shown. Each `min` is used if it is a finite number, rounded and clamped to 1 to 60, otherwise it is 5. `other` is true if it was saved as `true`, or if `min` isn't one of the drop-down's lengths, since such a length can only show under Other.

#### Styles

Every rule is scoped under `#breathing`. The page's colors are custom properties on `#breathing`, with dark values in its own `prefers-color-scheme: dark` media query:

| Variable | Light | Dark | Used for |
|---|---|---|---|
| `--br-inhale` | `#2f6fde` | `#5b8ef0` | Inhale arcs and key dots (the Work Timer's work blue, the same as `--accent`) |
| `--br-inhale-pale` | `#c1d4f5` | `#2e4266` | Inhale tracks (the Work Timer's work track) |
| `--br-hold` | `#c59711` | `#dab33a` | Hold arcs and key dots: a golden yellow |
| `--br-hold-pale` | `#eee0b8` | `#4d4122` | Hold tracks |
| `--br-exhale` | `#318454` | `#53ae77` | Exhale arcs and key dots (the Work Timer's rest green) |
| `--br-exhale-pale` | `#c1dacc` | `#2f4a38` | Exhale tracks (the Work Timer's rest track) |

- The yellow is OKLCH lightness 0.70, chroma 0.14 and hue 86 in light, and 0.78, 0.14 and 90 in dark. Its pale shades are made like the Work Timer's (5.1, Timer colors): in light, the yellow mixed 30% into white; in dark, a dim yellow at OKLCH lightness 0.38 (chroma 0.05) in its own hue.
- Contrast: the blue and green arcs are at least 3.1:1 against their tracks, as on the Work Timer. The yellow is 8.1:1 against the card and 5.0:1 against its track in dark, but only 2.7:1 and 2.1:1 in light: a yellow dark enough for 3:1 on white looks brown. The part's name under the time says which part it is anyway. The count and texts use `--text` and `--muted`.
- `[hidden]` inside the page is forced to `display: none`. Buttons, the drop-down and the box inherit the font and color, enabled buttons have a pointer cursor, and focus shows a 2px `--accent` outline with a 2px offset.
- `.br-main` is a centered flex column with a 16px gap and `container-type: inline-size`. `.br-panel` is the same column (16px gap), full width.
- **Tabs:** `.br-tabs` is styled like the Work Timer's `.wt-tabs`: a flex row with a 2px gap and 3px padding, a 1px `--border` border and a `--bg` background, here centered, wrapping if the tabs don't fit, no wider than the page, with a 22px radius (a pill on one line, a rounded box on more). Each tab: at least 92px wide, 6px × 18px padding, pill-shaped, a transparent 1px border, no background, `--muted` text on one line, turning `--text` on hover. The selected tab has a `--border` border, a `--surface` background, `--text` and weight 600. While a session runs, the other tabs are at 45% opacity. When `.br-main` is 340px wide or less (`@container`), tabs have 6px × 12px padding and no minimum width, so the two fit on one line on a 375px-wide phone.
- **Session length:** `.br-length` is a centered, wrapping flex row with an 8px × 10px gap. The label is `--muted`. The drop-down has 6px × 10px padding, and `.br-other` is an inline flex row with a 6px gap in `--muted`, whose box is 3.5em wide with 5px × 8px padding, `--text` and right-aligned text. The drop-down and the box have a 1px `--border` border, 8px radius and a `--surface` background, at 60% opacity when disabled.
- **Ring:** `.br-ring`, like the Eye Exercise's, is a size container and a one-cell grid (`grid-template: minmax(0, 1fr) / minmax(0, 1fr)`), `min(300px, 100cqi)` on each side (`cqi` here is the width of `.br-main`). Both children sit in the cell, centered. The SVG fills it (`overflow: visible`), and its paths have no fill, a stroke width of 6 and flat (butt) ends. `.br-ring-text` is a centered flex column. `.br-count`: `font-size: 22cqmin` (`30cqmin` during a get-ready countdown and when done), weight 600, `font-variant-numeric: tabular-nums`, letter spacing −0.02em, line height 1.1, `--muted` while paused. `.br-ring-label`: `--muted`, `font-size: max(13px, 8cqmin)`, line height 1.3, and `--text` at weight 600 while a part runs. At the full 300px, the count is 66px (90px for the countdown and ✓) and the label 24px.
- **Color key:** `.br-key` is a centered, wrapping flex row with a 4px × 16px gap, no margin, padding or list markers, in `--muted` at 0.95rem. Each item is an inline flex row with a 6px gap, kept on one line, whose `::before` is a 10px round dot in `--br-dot`, which `.is-inhale`, `.is-hold` and `.is-exhale` set to their kind's solid color.
- `.br-controls` is a flex row with an 8px gap, full width up to 360px, and an 8px top margin. `.br-btn` is styled like the Eye Exercise's `.ee-btn`: equal widths, 9px × 10px padding, pill-shaped, a 1px `--border` border, `--surface` background, weight 600, the border turning `--accent` on hover, and 45% opacity when disabled. `.br-btn-primary` has an `--accent` background and border and `--accent-text` text.
- `.br-sr` is visually hidden, like the Work Timer's `.wt-sr`.

#### Accessibility

- The tabs follow the usual tabs pattern: `role="tablist"`, `tab` and `tabpanel`, `aria-selected`, only the selected tab in the tab order, and the arrow keys, Home and End (see Exercises).
- `role="timer"` on the count keeps screen readers from reading out every change. The ring's SVG is hidden from them. The color key is a list labelled "One cycle", so the cycle is read out as text.
- `#br-live` announces "Breathing session done" at the end. It is emptied first and filled 100 ms later, so a repeated message is read again. Nothing is announced during a session: the dings mark the parts.
- When Start, Pause or Reset disables the button that had focus, focus moves on: from Start to Pause, from Pause to Start, and from Reset to Start. When the session ends by itself, focus on Pause or Reset moves to Start.

#### Known limits

- Phones usually suspend a page when the screen locks or the browser goes to the background. Since the session's time is on the audio clock, the session waits meanwhile rather than running on. The page keeps the screen on while a session runs, where the browser allows it. On an iPhone, silent mode may mute the dings.
- In a background tab on a computer, the dings stay on time, and the display catches up when the page is shown again. Tested in headless Firefox 161, with the page in a background tab: the dings stayed exactly one second apart, each scheduled about 6 seconds ahead, and Firefox used about 9% of a CPU core, against about 6% with no session running.
- Delays in the audio output, such as Bluetooth headphones' (often 0.1 to 0.3 seconds), make each ding come a little after its count changes.
- In a browser without Web Audio, the session runs silently.
- In light, the yellow is weaker against the white card than the other colors (see Styles).
