# NOTES.md

## Summary of Changes

Fixed seven bugs across frontend, backend, and SQL:

1. **SQL AND/OR precedence** (`TaskRepository.java`, `search_tasks.sql`, Oracle package) — missing parentheses caused the archived and status filters to apply to only one branch of the OR, leaking archived tasks and ignoring status on title matches.
2. **Artificial sleep** (`TaskController.java`) — `Thread.sleep` of up to 1000ms injected per request, worst on initial page load (empty query = full 1s delay).
3. **Loading stuck on error** (`useTasks.js`) — `setLoading(false)` missing from `.catch`, causing infinite spinner on fetch failure.
4. **Stale error state** (`useTasks.js`) — `setError(null)` never called at the start of a new fetch, so old errors blocked valid results from a successful retry.
5. **Page not reset on filter change** (`App.jsx`) — changing search or status while on a high page returned empty results because page was never reset to 1.
6. **Race condition on fast typing** (`useTasks.js`, `api.js`) — no `AbortController`, so slow out-of-order responses could overwrite current UI with stale data.
7. **Invalid status causes 500** (`TaskController.java`) — unrecognised status values threw uncaught `IllegalArgumentException`; now returns 400 with a clear message.

## What I Chose Not to Change

Backend fetches all rows then paginates in Java. Fixing this with DB-level `LIMIT`/`OFFSET` is the right call but touches the query layer and needs tests I couldn't add in the timebox.

## Biggest Remaining Risk

In-memory pagination. Fine at current volume, but as row count grows it will cause memory pressure and slow responses. Needs DB-level pagination before scaling.

## Tools / AI Used

Used Antigravity (Google AI assistant) to scan and understand project structure and find relevent files for bugs. I reviewed every finding against the actual source, All fixes were reviewed and verified by me before being applied.
