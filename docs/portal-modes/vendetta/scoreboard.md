# Scoreboard

Vendetta uses `mod.SetScoreboardType(CustomFFA)` with a five-column layout. Players are grouped by squad via the leading `Squad` column and `mod.SetScoreboardSorting`.

## The 6-argument cap

!!! danger "SetScoreboardPlayerValues hard caps at 6 arguments"
    `mod.SetScoreboardPlayerValues(player, ...)` accepts the player handle plus a maximum of **5** data values. Passing 6 data values throws **at runtime, not at parse time**. There is no compile-time check.

    Vendetta uses exactly 5 data columns, hitting the cap. Adding a sixth means dropping or merging one of the existing columns. See [Portal Scripting Gotchas](../../portal-scripting/gotchas.md#setscoreboardplayervalues-hard-caps-at-6-arguments).

## Current column layout

| # | Column | Source | Notes |
|---|---|---|---|
| 1 | **SQUAD** | Player's squad ID | Used for vertical grouping; squadmates cluster together. |
| 2 | **SCORE** | Per-player accumulator | `kills × 1 + hvtKills × 5 + hvtTimeTicks × 1` |
| 3 | **HVT KILLS** | Times this player killed an HVT | Tiebreaker rank 2 |
| 4 | **KILLS** | Total kills (includes HVT kills) | Tiebreaker rank 3 |
| 5 | **HVT TIME** | Seconds this player personally held HVT status | Tiebreaker rank 4 |

The five string keys live in `vendetta_v0_X_Y_strings.json`:

```json
"vendetta_scoreboard_squad_label":   "Squad",
"vendetta_scoreboard_score_label":   "Score",
"vendetta_scoreboard_hvtkills_label":"HVT Kills",
"vendetta_scoreboard_kills_label":   "Kills",
"vendetta_scoreboard_hvttime_label": "HVT Time"
```

## What BF6 prepends to every row

The native scoreboard prefixes each row with elements you don't control: rank icon, player name, and the engine's built-in player score (kills × 100 etc.). Reading a row left-to-right in-game looks like:

```
[rank] [name] [BF6 score]   [SQUAD] [SCORE] [HVT KILLS] [KILLS] [HVT TIME]
                            ←──── our 5 columns ────→
```

The BF6 score column is *not* what we display in the SCORE column — that's our personal-score accumulator with the Vendetta scoring formula. Both numbers will diverge. If a player asks why their "score" looks different from their "score," the answer is that one is BF6's built-in metric and the other is Vendetta's.

## Update cadence

The scoreboard is refreshed once per second by `MatchTickLoop` via `RefreshScoreboard()`, which iterates connected players and calls `SetScoreboardPlayerValues` for each. One additional refresh fires at the very end of `OnGameModeStarted` so the initial all-zeros render is immediate rather than waiting up to a second.

Per-event refreshes (on every kill) were considered and rejected — too many call sites, brittle to add new event types. Cadence-based is simpler and the 1-second granularity is imperceptible in practice.

## Squad grouping

`mod.SetScoreboardSorting(true, false)` is the call. The first arg sorts ascending by the SQUAD column (clustering squadmates); the second arg controls secondary sort which is currently disabled. If squadmates appear scattered, flip the second arg to `true` and re-test.

## HVT Time column scaling

HVT Time is shown as raw seconds. A player who held HVT for 90 seconds shows `90`. If this looks cramped or sprawling at scale (e.g. a player who held it for 600 seconds), Phase 5 includes a custom HUD widget that can format as MM:SS — but the scoreboard column itself is constrained to a single number.

## Adding a sixth column

You can't, without dropping one of the existing five. Options if you need to convey new data:

1. **Combine** — fold two values into a single string column. E.g. `"15K / 3HVT"` in one slot.
2. **Cycle** — alternate column meaning every N seconds. UX cost, but technically possible.
3. **Move it off the scoreboard** — Phase 5 HUD widgets are unconstrained by the 6-arg cap.

## Initial-render fix

A subtle bug in early Phase 3 builds: the scoreboard wouldn't render until the first `MatchTickLoop` pass, leaving players to see a blank board for up to a second after deploying. The fix is one extra `RefreshScoreboard()` call at the end of `OnGameModeStarted`:

```typescript
gameStarted = true;

// Render an initial all-zeros scoreboard so players see headers and rows
// immediately, instead of waiting up to 1 sec for the first MatchTickLoop pass.
RefreshScoreboard();

// Kick off async loops
HvtAliveTickLoop();
MatchTickLoop();
HvtIconUpdateLoop();
```
