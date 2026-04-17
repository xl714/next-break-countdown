# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page French countdown app that displays time remaining until a user's sabbatical date and/or their next vacation period, alongside a rotating photo slideshow backdrop. No build system — the entire app lives in `index.html` (HTML + CSS + vanilla JS), with flatpickr and Montserrat pulled from CDNs. Background images are served as static files from the repo root.

## Running / developing

- No package manager, no tests, no lint. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). Live-reload is not configured; reload the page after edits.
- Settings persist in `localStorage` under the key `countdown-settings-v1`. To reset during development, clear that key in DevTools or click the "Reinitialiser" button in the Settings modal (which restores `DEFAULT_SETTINGS` from the source).

## Architecture notes worth knowing before editing

The countdown logic is more nuanced than it looks — understand these before changing `tick()` or the helpers around it:

- **Two display modes, driven by `settings.hasSabbatical`.** When true: the main timer counts calendar time to `sabbaticalDate` (end-of-day), plus a "work time remaining" line computed in business-day milliseconds. When false: the main timer counts down to the last working moment before the *next* vacation block, and the vacation line becomes the featured stat.
- **"Business day" = not weekend, not French public holiday, not in a personal vacation period.** French holidays are computed per-year (including Easter-derived ones via `getEasterSunday`) and cached in `state.holidayCache`. `computeWorkRemainingUntil` sums `FULL_WORKDAY_MS` per remaining business day, with the current day pro-rated up to `WORK_END_HOUR` (18:00 local).
- **Vacation period merging is deliberate.** `mergeContinuousVacationPeriods` fuses adjacent user-entered periods when only weekends/holidays separate them, so a Fri-ending block and the following Mon-starting block are treated as one vacation for "next vacation" display. But the `paidLeaveDays` count on the merged block only counts days actually inside user-entered periods (via `personalLeavesRaw`) — do not conflate the two leave sets.
- **Date key format is `YYYYMMDD` strings** (no separators) for vacation periods and holiday sets, but the `<input type="date">` and flatpickr use `YYYY-MM-DD`. `parseKey` / `formatYYYYMMDD` / `parseDateInput` are the conversion points — keep them straight when touching date code.
- **Settings flow: live vs. draft.** `state.settings` is the saved/live config driving `tick()`. Opening the modal copies it into `state.draftSettings`, which all the modal handlers mutate. Only `saveDraftSettings` writes back via `saveSettings()` (which re-normalizes and persists). Cancelling (close / Escape) discards the draft.
- **`normalizeSettings` is the single validation gate** — it runs on both load and save, filtering malformed periods/photos and falling back to `DEFAULT_SETTINGS` fields. New settings fields should be added there or they will silently drop.

## Language / copy

UI copy is in French and intentionally written without accents (e.g. "Annee sabbatique", "Reinitialiser"). Match that style when adding new strings.
