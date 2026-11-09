# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ポモドーロタイマー

- 応答は日本語で。変更点は最後に3行でまとめる。
- index.html の1ファイル構成を保つ。
- 外部のライブラリやインターネット接続は使わない。
- ToDo の保存形式(localStorage の pomodoro-todos)は変えない。

## Overview

A Pomodoro timer with a three-item ToDo list. The whole app is `index.html`: inline CSS, markup, and one inline script in an IIFE. There are no dependencies and no external requests. UI text and code comments are in Japanese.

## Commands

There is no build, lint, or test tooling. To run the app, open `index.html` in a browser.

## Testing changes

State lives inside the IIFE, so timer lengths can't be changed from the console. To exercise a full cycle, copy `index.html` outside the repo, shorten `DURATIONS` in the copy, and drive the copy in a browser. Don't edit the constants in place.

Headless Chrome won't shrink its window below 500px. To check the 640px-and-under layout at phone widths, load the page in a narrower iframe.

## Timer architecture

- **Time comes from the wall clock.** While running, `endAt` holds the finish timestamp and `tick()` computes `remaining = endAt - Date.now()`. Never count down by decrementing, or the timer drifts when ticks are throttled. `endAt === null` means stopped or paused.
- **Ticks come from an inline Worker** built from a Blob, because background tabs throttle page timers. `startTicker()` and `stopTicker()` fall back to `setInterval` if the Worker is unavailable or errors.
- **`DURATIONS` defines the modes**, and `MODES` is derived from its keys. Adding a mode needs a `DURATIONS` entry, a `LABELS` entry, a `<button id="mode-<key>">`, a `body[data-mode="<key>"]` accent rule, and a place in the transition logic in `tick()`. `.modes` also has a hard-coded column count.
- **Long breaks:** `completedWork` counts only work sessions that ran to zero, so reset and manual mode changes leave it alone. It returns to 0 when a long break is entered automatically and when any long break finishes. It is deliberately not persisted, matching the timer's other state.
- **Mode changes go through `setMode()`**, which stops the ticker and resets `remaining`. `render()` writes all timer DOM from state, including the progress ring, `aria-pressed`, and the tab title.
- **Audio must be unlocked in a user gesture.** `prepareAudio()` runs from the start button's click handler and plays a silent buffer for iOS. `chime()` can only sound if that has already happened.

## ToDo list

Rows are built in JS (`TODO_COUNT`) and saved to `localStorage` under `pomodoro-todos` as `{ items: [{ text, done }] }`. Items intentionally carry over between days; the date in the header is display-only and is not used to clear anything. Storage access is wrapped in `try/catch` so the page still works without it.

## Styling conventions

- Colors are CSS variables on `:root`, redefined in the `prefers-color-scheme: dark` block. New colors need both values.
- `--accent` follows the current mode through `body[data-mode]`.
- Touch targets are at least 48px tall, for use on phones.
