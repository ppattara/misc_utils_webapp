# misc_utils_webapp

A small personal web app that collects everyday utilities in one place, one per tab. It is a single HTML file: nothing to install, no account, and no internet connection needed.

## Using it

- **Open it:** double-click `index.html`, or open it in any modern browser. It can also be hosted on any static site host, such as GitHub Pages.
- **Switch utilities:** click a tab at the top. The browser tab shows the name of the page you're on, except while a Work Timer countdown runs, when it shows the time left.
- **Bookmark a utility:** each tab has its own link (for example `index.html#/work-timer`), so a bookmark can jump straight to it. The browser's back and forward buttons move between tabs you have visited.
- **Light or dark:** the app follows your device's appearance setting.
- **Saved settings:** a utility may remember its settings in your browser, on that device only. Clearing the browser's site data brings back the defaults.

## Utilities

| Tab | What it does |
|---|---|
| Work Timer | Work and rest countdowns, with random reminder dings during work |
| Page 2 | Not built yet |
| Page 3 | Not built yet |

## Work Timer

A countdown for focused work and another for breaks. While you work, a soft ding sounds now and then at random times, as a reminder to stay on task.

- **Work and Rest:** choose a timer with the Work and Rest tabs. Work starts at 25 minutes and Rest at 5, and you can change both in the settings.
- **Colors:** each timer has its own color, blue for Work and green for Rest, so you can tell at a glance which one you're looking at. The card behind the timer turns a very soft shade of that color, soft blue or soft green, as you switch timers, and so does the floating window. The ring, the Start button, the time while it runs and the dot on a running timer's tab take the color itself.
- **Ring:** the time sits in the middle of a ring, with the timer's name (Work or Rest) under it. The ring's solid strip shows how much of the session is left, over a lighter track in the same color. It's full at the start and runs down, back toward the top, as time passes. The whole ring turns grey while the timer is paused.
- **Start, Pause, Reset:** Start begins a session with a short rising chime. Pause stops the clock, and the Start button then reads Resume. Reset puts the timer back to its full time.
- **One at a time:** starting one timer resets the other. Switching tabs doesn't stop anything, and a small dot on a tab, in that timer's color, shows which timer is running.
- **When time is up:** a falling chime plays and the page switches to the other timer, ready for you to start it.
- **Random reminder:** while the work timer runs, a single ding plays after a random gap of 2 to 5 minutes, again and again. Each gap is picked fresh, and the time to the next ding is never shown. Pausing pauses the reminders too, and no ding plays in the last 15 seconds of a session. Turn the reminder off in the settings for a plain work timer.
- **Time in the browser tab:** while a timer runs, the browser tab shows the time left (for example "18:42 · Work"), so you can check it from another tab.
- **Floating window:** on a computer, the window button next to the gear moves the timer into a small window that stays on top of your other windows, so you can keep it in a corner of the screen while you work. Drag it where you like and resize it. The ring comes along and grows or shrinks with the window. In a smaller window, the Work/Rest tabs and the reminder line drop out, and the name inside the ring still tells you which timer it is. In a wide, short window (like the one it opens in), the buttons sit beside the ring. In a very short strip, a small ring sits next to the time, with the buttons on the same line, and the colors (a soft blue or soft green window) tell you which timer it is. Only the buttons you can use right now are shown. It's still the same timer, so the sounds and the time in the browser tab carry on, and you can change settings on the page meanwhile. To put the timer back, press the window button again, press "Bring it back" on the page, or close the floating window. It works in Chrome, Edge and Firefox on a computer. Safari and phones don't support it, so the button doesn't appear there.
- **Settings:** the gear button opens the settings sidebar. On a phone it opens over the timer. There you can:
  - set the work and rest times,
  - turn the random reminder on or off,
  - set the shortest and longest gap between reminders,
  - play each sound to hear it.

  Times go in half-minute steps: 0.5 to 999 minutes for work and rest, and 0.5 to 60 for the gaps. A new work or rest time applies right away if that timer hasn't started; otherwise it applies from its next session.
- **Saved settings:** your settings are remembered by this browser on this device. Clearing the browser's site data brings back the defaults.
- **Keep screen on:** stops the screen from turning off while the page is open, which is handy on a phone. Tap it again to turn it off. It appears only in browsers that support it, and it starts off each time you open the page.
- **In the background:** on a computer, the timer keeps running and the dings play on time while the page sits in a background tab, or while you use another utility in the app.
- **On a phone:** phones usually pause web pages when the screen locks or you switch apps, so no dings play then. The time is still right when you come back. Use Keep screen on to avoid this. On an iPhone, silent mode may mute the sounds.

## More

- `SPEC.md`: the full description of how the app works, detailed enough to rebuild it.
- `CLAUDE.md`: working rules for developing this project with Claude Code.
