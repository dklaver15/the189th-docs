# Scoreboard

Vendetta uses a custom per-player scoreboard via `SetScoreboardPlayerValues`. The default Battlefield scoreboard isn't expressive enough for HVT-mode stats (kill-as-HVT vs kill-of-HVT is a different concept than a standard kill).

## The 6-argument cap

!!! danger "SetScoreboardPlayerValues hard caps at 6 columns"
    The Portal API `SetScoreboardPlayerValues` accepts at most 6 values. Passing 7 throws at runtime, not at parse time, so this isn't caught by TypeScript types. **You will not see the error in the editor — you will see it on PS5 when the mode goes live.**

    Plan column layouts under this constraint from the start. If you need more data, fold related stats into a single column (e.g. "K/HVT-K" as a combined string) rather than splitting.

## Current column layout

| # | Column | Source |
|---|---|---|
| 1 | Score | per-player score accumulator |
| 2 | Kills | standard kill count |
| 3 | HVT Kills | kills where victim was the HVT |
| 4 | _TODO_ | _TODO_ |
| 5 | _TODO_ | _TODO_ |
| 6 | _TODO_ | _TODO_ |

## Update cadence

The scoreboard is updated on:

- Every kill event (immediate refresh for the killer and victim)
- Every round transition (full refresh for all players)
- Periodic tick (every N seconds, to reflect timer-driven values like "rounds played")

Updating the scoreboard every frame is wasteful. The current implementation only pushes when something actually changed.

## Player-name display

Player names come from the standard Portal player object — no custom rendering is needed for the name column itself. Only the **values** for the configured columns are passed via `SetScoreboardPlayerValues`.

## Adding a new column

1. Pick which existing column to drop or merge — there is no spare slot.
2. Add the new accumulator to the per-player state object.
3. Update kill/score event handlers to maintain it.
4. Add it to the `SetScoreboardPlayerValues` call.
5. Update the column header (Portal editor, not script).
6. Test on PS5, since the runtime hard-fails over 6 args and that surfaces only at execution.
