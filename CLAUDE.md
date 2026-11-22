# misc_utils_webapp

A web app that collects miscellaneous utilities for everyday use. It is a static HTML file, with no backend, database or user login.

## Project files
- `index.html`: the app itself. It ends with the Work Timer's recorded sounds, embedded as base64 (see Working on the app).
- `CLAUDE.md`: ground rules and working notes for Claude (this file).
- `SPEC.md`: the full description of the app, detailed enough to rebuild it (or something very close) from that file alone.
- `README.md`: the project summary, which doubles as the user manual.
- `timer_sound/`: the owner's recordings for the Work Timer's sounds, the source of the copies embedded in `index.html`. The app never reads this folder. Leave the files as they are unless the owner asks for a change. To change a sound, replace its file and embed it again (`SPEC.md` 5.1, Sounds).

## Ground rules (always follow)

### 1. No local memory: everything lives in the repo
This project is worked on from multiple machines through a GitHub repo, so anything a future session needs to know must be in a file in this repo.
- Never use Claude Code's per-machine memory (`~/.claude/projects/<project>/memory/`), `~/.claude/CLAUDE.md`, or `CLAUDE.local.md` for this project.
- Record decisions, requirements, conventions, progress and open TODOs in this file, or in a repo file linked from here.

### 2. No git commands
The owner is the only one who interacts with git.
- Never run `git` in any form, including read-only commands like `git status`, `git diff` or `git log`. The same goes for `gh`.
- Watch for tools that run git under the hood: scaffolders that call `git init` (use their option to skip it), `npm version`, git-hook installers like husky, and Claude Code features built on git (worktrees, branch/diff-based reviews).
- When a change is ready, say so. Committing and pushing are up to the owner.

### 3. Keep the app and its docs in sync
Every change to the webapp (a new feature, changed behavior, a removal, a UI change) must update the docs in the same piece of work:
- `SPEC.md`, so it still describes the current app in enough detail to rebuild it from that file alone.
- `README.md`, so the summary and user manual match what the app does now.
- `CLAUDE.md`, if the change affects conventions, decisions or the status notes below.

A change isn't finished until the code and all three files agree. When reporting a change, list which of these files were updated. If the code and docs are found to disagree (for example after a manual edit), point it out and ask the owner which is correct before changing either.

## Working on the app
- Read `SPEC.md` before changing the app. It covers the structure, conventions and constraints (one self-contained file, no external dependencies), including how to add a page.
- To check a change, open `index.html` in a browser. There is no server or build step.
- `index.html` ends with about 220 KB of base64: the embedded recordings, four very long lines under the comment headed `Embedded data`. Read only the code above them, for example with a line limit that stops at that comment (`grep -n 'Embedded data' index.html` gives its line), and never print or hand-edit them. `SPEC.md` 5.1 (Sounds) says how to replace a recording.

## Decisions
- **Saved settings** (owner, 2026-11-22): pages may remember their settings in the browser's `localStorage` only, never in files, a database or an account. If the browser's data is cleared, settings simply return to their defaults. Keep this simple. The convention is in `SPEC.md` section 3.4.
- **Browser tab title** (owner, 2026-11-22): the title is normally the page name, but while a Work Timer countdown runs it shows the time left instead (`SPEC.md` 3.2 and 5.1).
- **Floating window** (owner, 2026-11-22): ordinary browser windows can't be made narrower than about 400–500 px, and a page can't change that. So that the Work Timer can sit small in a corner of the screen, it can pop out into an always-on-top floating window using the Document Picture-in-Picture API (desktop Chrome, Edge and Firefox 151+; not Safari or phones). The owner tried a prototype in Firefox 160 before it was built in. Details in `SPEC.md` 5.1.
- **Ring around the time** (owner, 2026-11-22): from a reference picture the owner shared, the Work Timer shows its time in the middle of a ring whose arc runs down as the session goes, on the page and in the floating window. At first the arc used the app's blue accent rather than the picture's green; it now takes the timer's color (see Timer colors). The text under the time is the timer's name ("Work" or "Rest"), not the picture's "Hours worked". The owner left the design for floating windows too small for the ring to Claude. Chosen: in a wide, short window the buttons move beside the ring, and at 100px tall or less a small ring sits beside the time (`SPEC.md` 5.1, Styles).
- **Timer colors** (owner, 2026-11-22): so the two timers look distinct, Work is blue and Rest is green, in soft tones rather than pure `#0000ff` and `#00ff00`. The ring's arc is the solid color, over a much paler track of the same color (later reworked into three shades, see the next entry). Details chosen by Claude: Work keeps the app's accent blue, so it matches the page tabs; Rest is a calm green of about the same lightness, dark enough for the Start button's white label; the Start button, the running time and the running dot take the timer's color too, while the rest of the page keeps the accent; a paused ring still turns grey, track included (`SPEC.md` 5.1, Timer colors).
- **Three shades per timer color** (owner, 2026-11-22): keeping the same blue and green, the owner asked for the ring's unfilled part (the track) to be a lighter shade of the color, and the card behind the timer an even lighter, very soft shade of it. Details chosen by Claude:
  - The card follows the timer on show, so it changes on a tab switch and when a session ends. The floating window's background follows it too.
  - The track went from an 18% to a 30% mix into white, so that it still stands out on the tinted card.
  - The two cards share one lightness and chroma, so the green doesn't look greyer than the blue.
  - Dark-theme shades keep each color's own hue, because mixing green into the bluish dark surface turns it teal. The dark tracks are dimmed until the arc is at least about 3:1 against them.
  - The card keeps its tint while paused, and only the ring turns grey.
  - The buttons, the Work/Rest switch and the settings sidebar keep their own backgrounds.

  Values and contrast are in `SPEC.md` 5.1, Timer colors.
- **Random breaks** (owner, 2026-11-22): the random reminder's single ding became a short break: a ding, then a break of 10 seconds by default (settable), then the same ding. (Breaks now have recorded sounds by default, see Recorded sounds.) It's a moment to stop working, maybe stand up and stretch. So that someone with the sound off still notices, the timer turns green during a break, in a green of its own, distinct from Rest's. The work clock keeps running through breaks, and a session still ends at its set time. During a break the browser tab's icon becomes a green dot. Details chosen by Claude, in a design the owner approved:
  - A gap is the work time between breaks, from the end of one to the start of the next, so breaks can't overlap. A break starts only if it can end at least 15 s before the session does.
  - During a break, the ring counts the break down, the name under the time reads "Break", the info line "Take a short break" and the tab title `00:08 · Break`. If the Rest tab is on show when a break starts, the page switches to Work.
  - Pausing during a break ends it, and Resume draws a fresh gap. Reset, starting Rest and changing a break setting end it silently.
  - Screen readers hear "Short break" and "Back to work".
  - The break length went from 5 to 300 seconds, in whole seconds (now 5 to 15, see Typed-in settings). The saved-settings key `remindOn` keeps its old name, so saved settings carry over.
  - The break's green is brighter and a little yellower than Rest's. In light mode the card fills with a bright green and the ring and time turn deep green. In dark mode the card is a deep green and the ring and time a very light green.
  - Apart from the green dot, the tab's icon is blank (transparent), because browsers don't reliably bring back their default icon when a page removes its own.

  Values and contrast are in `SPEC.md` 5.1 (Random breaks, Timer colors, Tab icon).
- **Recorded sounds** (owner, 2026-11-22): the owner's six recordings in `timer_sound/` play by default at six moments: `work_begin` and `work_end` at the start and end of a work session, `rest_begin` and `rest_end` for rest, and `break_begin` and `break_end` at the start and end of a random break. The old dings stay as a setting. The recordings are embedded in `index.html` so that it stays self-contained, even though it grows (from about 60 KB to about 280 KB). Details chosen by Claude:
  - The setting is a pair of radio buttons in the Sounds group, "Recorded sounds" (the default) and "Dings", saved as `sounds`, so settings saved before it get the recordings. Under them, a Start and an End play button each for Work, Rest and Break preview the six moments in the chosen style.
  - The recordings play through Web Audio, scheduled ahead on the audio clock like the dings, so they stay on time in background tabs. They're decoded on the first click that needs sound.
  - `work_begin.m4a` and `rest_end.m4a` are the same file, and so are `work_end.m4a` and `rest_begin.m4a`, so each pair is embedded once: four recordings, not six.
  - The base64 goes at the end of the file, so the code above stays readable.
  - A recording that a browser can't decode plays its ding instead.
  - A new sound choice applies at once, even to the sounds already scheduled.
  - The audio device idles only once the last sound has finished, so a longer recording isn't cut off.
  - The recordings play at their own level, unchanged.

  Details in `SPEC.md` 5.1 (Sounds).
- **Typed-in settings** (owner, 2026-11-22): the Work Timer's five number settings are typed in by hand, with no up/down arrows, and take whole numbers only. Ranges, with defaults: work 10–120 minutes (25), rest 3–30 minutes (5), break length 5–15 seconds (10), shortest gap 1–5 minutes (2) and longest gap 2–10 minutes (5). The shortest and longest gap can't be equal: when a typed one would meet the other, the other moves 1 minute away. Details chosen by Claude:
  - The fields are text inputs with `inputmode="numeric"`, not number inputs, so they have no arrows, and the mouse wheel and arrow keys don't change them either. Phones show a number keypad.
  - An entry is checked when it's committed (Enter or leaving the field). A decimal is rounded to the nearest whole number, a number out of range becomes the nearest limit, and anything else (an empty field too) puts the previous value back. An entry that ends up at the value already set changes nothing, so retyping a gap doesn't redraw the planned breaks.
  - A gap typed past the other one, not just onto it, also moves the other to 1 minute from the typed value.
  - Saved settings from before are rounded and clamped into the new ranges. If the gaps then aren't at least a minute apart, both go back to 2 and 5.
  - The fields went from 4.5em to 3.5em wide, since they no longer have arrows.

  Details in `SPEC.md` 5.1 (Settings sidebar).

## Status
- v1: blank shell with three placeholder pages (Page 1, Page 2, Page 3) and the shared tab navigation.
- v2 (2026-11-22): the first page is now the Work Timer (`#/work-timer`, replacing Page 1). It has work and rest countdowns, random reminder dings during work, a settings sidebar with saved settings and sound previews, the countdown in the browser tab title, and a keep-screen-on button. Pages 2 and 3 are still placeholders.
- v3 (2026-11-22): the Work Timer can pop out into a floating window (the button next to the gear). The window adapts to its size, down to a one-line strip with the time and a single button.
- v4 (2026-11-22): the Work Timer's time sits in the middle of a ring that runs down as the session goes, with the timer's name under it, on the page and in the floating window. The floating window's layouts were reworked around the ring.
- v5 (2026-11-22): the Work and Rest timers have their own colors, blue and green. The ring is a solid arc over a much paler track of the same color, and the Start button, the running time and the running dot take the timer's color.
- v6 (2026-11-22): each timer color comes in three shades. The ring's track is a lighter shade of the timer's color, and the card behind the timer (and the floating window) is a very soft shade of it that follows the timer on show.
- v7 (2026-11-22): the random reminder became random short breaks: a ding, a break (10 s by default, settable), then the same ding. During a break the timer turns a bright green of its own and counts the break down, and the browser tab shows a green dot. The work clock keeps running through breaks.
- v8 (2026-11-22): the Work Timer plays the owner's recordings at the start and end of work, of rest and of each break, embedded in `index.html` as base64. The old dings are a setting, and the sidebar previews all six sounds.
- v9 (2026-11-22): the Work Timer's settings are typed in by hand as whole numbers, with no up/down arrows, in narrower ranges (work 10–120 min, rest 3–30 min, break length 5–15 s, gaps 1–5 and 2–10 min), and the shortest gap always stays below the longest.
- Next: the owner will describe the utilities for Pages 2 and 3.
