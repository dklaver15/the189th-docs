# Changelog

Phase-by-phase version history. Each phase is a self-contained scope with its own version bump.

## v0.4.0 — Phase 4 (current)

**HUD: splash, clan info box, MM:SS round timer.**

### Added
- **Match-start splash panel** — center of screen on match start: "189TH CLAN VENDETTA / By xAP3XRONINx / Vendetta v0.4.0 started / discord.gg/189th". Auto-dismisses after the splash window.
- **Persistent clan info box** — top-left corner, amber accent, visible for the full match: "189th Clan / Vendetta / discord.gg/189th".
- **MM:SS round timer** — top-center, below the BF6 compass. Counts down from 15:00. Goes red and pulses brackets in the final 60 seconds.
- New string keys for HUD elements (`vendetta_splash_title`, `vendetta_clan_box_title`, `vendetta_timer_separator`, etc.).

## v0.3.0 — Phase 3

**Per-player stat tracking and custom 5-column scoreboard.**

### Added
- `mod.SetScoreboardType(CustomFFA)` with five data columns: SQUAD / SCORE / HVT KILLS / KILLS / HVT TIME.
- Per-player stat accumulators: `score`, `hvtKills`, `kills`, `hvtTime`.
- `RefreshScoreboard()` cadence-driven from `MatchTickLoop` (1 Hz).
- Initial-render `RefreshScoreboard()` call at the end of `OnGameModeStarted` so the board is populated immediately rather than after the first tick.
- Squad-grouped sorting via `mod.SetScoreboardSorting`.

### Fixed (code review between Phase 2 and 3)
- Stale header docstring referencing only Phase 1 + 2.
- Incorrect `OnPlayerDeployed` refresh pattern (changed to first-deploy-only with a 0.1s delay matching the CTF v4.5.11 pattern).
- Inconsistent `worldIconManager` null guards across HVT lifecycle handlers.
- Removed call to non-existent `mod.EnableDefaultGameModeWinCondition` API.

## v0.2.0 — Phase 2

**HVT WorldIcon pings (2A) and +25 HP buff (2B).**

### Added — Phase 2A
- `WorldIconManager` class (lifted from CTF v4.5.11).
- HVT WorldIcon: Alert image, parented to HVT player at head height, visible to all players.
- `HvtIconUpdateLoop` async iterator refreshing icon position at ~4 Hz.
- Icon lifecycle hooks tied to slot promotion / loss events.

### Added — Phase 2B
- HVT max health buffed from 100 to 125 via `mod.SetPlayerMaxHealth` on promotion.
- Heal-up to new max immediately on promotion (otherwise player walks around at 80% effective HP).
- Max health reverts to 100 on demotion / death.

### Removed
- Proximity score multiplier (was in the original Phase 2 design; cut after playtests showed it incentivized HVT camping rather than aggressive play).

## v0.1.0 — Phase 1

**State machine, scoring loop, end-by-time with tiebreakers.**

### Added
- Match state machine: `WaitingToStart` → `Live` → `FinalMinute` → `Ended`.
- HVT slot system: per-squad slots with Bootstrap and Locked phases.
- Slot transition logic for cross-squad kills, HVT-on-HVT kills, and non-kill HVT deaths.
- Scoring per squad: `+1` per kill, `+5` per HVT-kill, `+1` per 5s alive tick (per live HVT), `+1` recovery bonus on direct chain recovery.
- Final-minute trigger at t=60s remaining with global 2× multiplier.
- End-by-time at t=0 with four-stage tiebreaker (score → HVT-kills → total kills → HVT time held).
- Match dispatch via `DummySpatialProp` at `ObjId 40010`.
- Heavy debug logging via `console.log` (PC-visible) and `mod.DisplayHighlightedWorldLogMessage` (PS5-visible toasts).
- On-screen breadcrumbs via `mod.SendErrorReport` for the PS5 debug overlay.
- Full string-key coverage for promotions, slot losses, and debug events.

### Design phase decisions (locked before Phase 1)
- Two-team initial concept revised to per-squad slot system.
- HVT slot count formula: `max(1, squadCountAtMatchStart - 2)`.
- 15-minute hard timer, no score limit.
- Sudden death overtime scope reduced to two-way ties at first place only.
- No vehicles (Portal Builder restriction, not script).

---

## Deferred phases

### Phase 5 _(planned)_

- HVT identity card on promotion (full-screen reveal).
- HVT count indicator (HUD widget showing active vs. locked).
- Final-minute UI takeover (red border, ticker, etc.).
- Squad color-coding throughout HUD elements.

### Phase 6 _(planned)_

- Sudden death overtime (two-way ties at first place).
- Hard-out spectate for non-finalists during overtime.

---

## Template for new entries

```markdown
## vX.Y.Z — Phase Name (YYYY-MM-DD)

### Added
- ...

### Changed
- ...

### Fixed
- ...

### Removed
- ...
```
