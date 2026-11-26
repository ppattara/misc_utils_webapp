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
| Work Timer | Work and rest countdowns, with random short breaks during work |
| Exercise | Exercise routines to follow along with: holds are timed and reps counted, with dings |
| Eye Exercise | An eye exercise recording, played five times in a row, then a big ding |

## Work Timer

A countdown for focused work and another for breaks. While you work, short breaks come at random times: a sound tells you to stop for a moment (stand up, stretch, look away), and another tells you to get back to work. During each break the timer turns bright green, so you notice it even with the sound off.

- **Work and Rest:** choose a timer with the Work and Rest tabs. Work starts at 25 minutes and Rest at 5, and you can change both in the settings.
- **Colors:** each timer has its own color, blue for Work and green for Rest, so you can tell at a glance which one you're looking at. The card behind the timer turns a very soft shade of that color, soft blue or soft green, as you switch timers, and so does the floating window. The ring, the Start button, the time while it runs and the dot on a running timer's tab take the color itself. During a short break, the card (and the floating window) fills with a brighter green of its own, quite unlike Rest's soft green.
- **Ring:** the time sits in the middle of a ring, with the timer's name (Work or Rest) under it. The ring's solid strip shows how much of the session is left, over a lighter track in the same color. It's full at the start and runs down, back toward the top, as time passes. The whole ring turns grey while the timer is paused. During a short break, the ring counts the break down instead, and its name reads Break.
- **Start, Pause, Reset:** Start begins a session with its start sound. Pause stops the clock, and the Start button then reads Resume. Reset puts the timer back to its full time.
- **One at a time:** starting one timer resets the other. Switching tabs doesn't stop anything, and a small dot on a tab, in that timer's color, shows which timer is running.
- **When time is up:** the session's end sound plays and the page switches to the other timer, ready for you to start it.
- **Random breaks:** while the work timer runs, a short break comes after a random 2 to 5 minutes of work, again and again.
  - A sound starts each break, and another ends it 10 seconds later. In between, the timer turns bright green, the ring counts the break down, and the browser tab shows a green dot and the break's time left ("00:08 · Break").
  - The work clock keeps running through breaks, so a 25-minute session still ends after 25 minutes.
  - Each gap is picked fresh, and the time to the next break is never shown.
  - A break only comes if it can finish at least 15 seconds before the session ends.
  - Pausing during a break ends that break. Pausing at other times keeps the time to the next break for when you resume.
  - Turn breaks off in the settings for a plain work timer.
- **Sounds:** each start and end has its own recorded sound: the start and end of a work session, of a rest session and of each break. If you'd rather hear simple dings, choose Dings in the settings: a short rising chime when a session starts, a falling one when it ends, and a single ding at the start and end of each break. The recordings are built into the page, so it still works as a single file, even offline. A browser that can't play a recording plays its ding instead.
- **Time in the browser tab:** while a timer runs, the browser tab shows the time left (for example "18:42 · Work"), so you can check it from another tab. During a short break it shows the break's time left, and the tab's icon becomes a green dot. Apart from that dot, the app's tab has no icon.
- **Floating window:** on a computer, the window button next to the gear moves the timer into a small window that stays on top of your other windows, so you can keep it in a corner of the screen while you work. Drag it where you like and resize it. The ring comes along and grows or shrinks with the window. In a smaller window, the Work/Rest tabs and the line under the ring drop out, and the name inside the ring still tells you which timer it is (or Break). In a wide, short window (like the one it opens in), the buttons sit beside the ring. In a very short strip, a small ring sits next to the time, with the buttons on the same line, and the colors tell you what's on: a soft blue or soft green window for Work or Rest, bright green during a break. Only the buttons you can use right now are shown. It's still the same timer, so the sounds and the time in the browser tab carry on, and you can change settings on the page meanwhile. To put the timer back, press the window button again, press "Bring it back" on the page, or close the floating window. It works in Chrome, Edge and Firefox on a computer. Safari and phones don't support it, so the button doesn't appear there.
- **Settings:** the gear button opens the settings sidebar. On a phone it opens over the timer. There you can:
  - set the work time (10 to 120 minutes, 25 to start with) and the rest time (3 to 30 minutes, 5 to start with),
  - turn the random breaks on or off,
  - set how long each break lasts (5 to 15 seconds, 10 to start with),
  - set the shortest and longest gap between breaks, the work time from one break to the next (shortest 1 to 5 minutes, longest 2 to 10 minutes, 2 and 5 to start with),
  - choose the recorded sounds (the default) or dings,
  - play each sound to hear it: the start and end of work, of rest and of a break.

  Type the numbers in by hand, as whole numbers. The boxes have no up/down arrows. A new number applies when you press Enter or leave the box. A number with a decimal is rounded to the nearest whole number, one outside its range becomes the nearest limit (200 minutes of work becomes 120), and anything that isn't a number brings back the previous value. The shortest gap always stays below the longest: if you type one that would reach or pass the other, the other moves to 1 minute away. For example, with gaps of 2 and 5 minutes, typing 5 as the shortest makes the longest 6. A new work or rest time applies right away if that timer hasn't started; otherwise it applies from its next session.
- **Saved settings:** your settings are remembered by this browser on this device. Clearing the browser's site data brings back the defaults.
- **Keep screen on:** stops the screen from turning off while the page is open, which is handy on a phone. Tap it again to turn it off. It appears only in browsers that support it, and it starts off each time you open the page.
- **In the background:** on a computer, the timer keeps running and the sounds play on time while the page sits in a background tab, or while you use another utility in the app.
- **On a phone:** phones usually pause web pages when the screen locks or you switch apps, so no sounds play then. The time is still right when you come back. Use Keep screen on to avoid this. On an iPhone, silent mode may mute the sounds.

## Exercise

An exercise routine to follow along with. Pick a routine, press เริ่ม (Start), and follow it from beginning to end without touching anything. The page counts down each hold, counts your reps at the right pace, and dings to tell you when to start and stop. The page's words are in Thai, like the exercises.

- **Routines:** pick one from the ชุดท่า drop-down. For now there is one: หมอชวนฟิต แก้ปวดคอบ่าไหล่, 13 neck and shoulder exercises taking about 6½ minutes, from a video by หมอชวนฟิต [DeDoctor]. Its link is in `exercise_routines/DD_neck_shoulder.txt`. The page remembers your choice.
- **Start:** first comes a 5-second countdown (เตรียมตัว) to get ready. Then the routine runs by itself to the end. Before each later exercise, and before switching sides, a 3-second countdown shows what comes next (ต่อไป) and gives you time to get into position.
- **Holds:** the ring counts the hold down in seconds. For exercises done on each side, the page shows ข้างขวา or ข้างซ้าย, and for one done twice, รอบที่ 1 จาก 2.
- **Reps:** the ring counts your reps (for example 2 จาก 10) at the routine's pace. In alternating moves (สลับไปมา), each movement one way counts as one rep, so 10 reps means 5 each way.
- **Dings:** one ding when an exercise starts, a falling two-note ding when it ends, and a softer, shorter ding at each new rep. The countdowns are silent.
- **On screen:** the exercise's name in large type, with its description beside the ring. Above the name are the side, the set, and the hold time or reps. The ring's color shows what's happening: amber during a countdown, blue during an exercise, grey while paused. Before you start, the ring shows the total time.
- **The list:** on the right (below, on a phone) are all of the routine's exercises in small type, grouped into warm-up, stretches and strengthening, with the total time at the top. The bar beside them fills as you go. Finished exercises turn grey, and the current one is bold.
- **Pause and resume:** หยุดชั่วคราว (Pause) stops everything where it is. ทำต่อ (Resume) carries on from that moment, straight away. Or, while paused, click an exercise in the list to continue from there instead. Resume then gives you the 3-second countdown and starts that exercise from the beginning.
- **Start partway:** before starting, click an exercise in the list to start from it. The ring shows the time from there to the end.
- **When it's done:** the page shows ครบทุกท่าแล้ว. Press เริ่ม to do it again.
- **Screen stays on:** while the routine runs, the page stops the screen from turning off, in browsers that allow it, and lets it go when you pause or finish.
- **In the background:** on a computer, the dings stay on time while you use another tab or utility, and the screen catches up when you come back. Phones usually pause web pages when the screen locks or you switch apps, so keep the page on screen. On an iPhone, silent mode may mute the dings.

## Eye Exercise

An eye exercise recording, played five times in a row with no break in between, and a big ding when it's all done. It takes about 1 minute 20 seconds.

- **Start, Pause, Reset:** Start plays it from the beginning. Pause stops it right where it is, even mid-sentence, and the Start button then reads Resume, which carries on from that point. Reset stops it and goes back to the beginning.
- **The ring:** the middle shows which play it is (for example 2 of 5), and the ring's blue strip runs down as that play goes, then fills up again as the next one starts. It turns grey while paused. Before you start, it shows the total time, and at the end a ✓.
- **No breaks:** each play starts the moment the one before it ends. The recording has a second or so of quiet at each end, so about 2½ seconds pass between one play's voice and the next.
- **Sounds:** the recording plays at its own volume. When the last play ends, a big, low ding rings out for a few seconds, clearly louder than the voice. The recording is built into the page, so it works offline too.
- **Screen stays on:** while it plays, the page stops the screen from turning off, in browsers that allow it, and lets it go when you pause, reset or finish.
- **In the background:** on a computer, it keeps playing on time while you use another tab or utility. Phones usually pause web pages when the screen locks or you switch apps, so keep the page on screen. If the sound doesn't come back when you return, press Pause and then Resume. On an iPhone, silent mode may mute it.
- If your browser can't play the recording, the page says so and the Start button stays off.

## More

- `exercise_routines/`: the exercise routines, one text file each. The app has its own copies built in, so after changing a file, it needs to be copied into `index.html` again (see `SPEC.md` 5.2).
- `eye_exercise/`: the Eye Exercise recording. The app has its own copy built in, so after replacing it, it needs to be copied into `index.html` again (see `SPEC.md` 5.3).
- `SPEC.md`: the full description of how the app works, detailed enough to rebuild it.
- `CLAUDE.md`: working rules for developing this project with Claude Code.
