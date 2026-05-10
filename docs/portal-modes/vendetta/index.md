# Vendetta

Multi-squad HVT (High Value Target) mode inspired by Battlefield 6 Gauntlet's Vendetta variant. Each squad has at most one HVT slot. HVTs score passive points for their squad while alive, and become priority targets for everyone else. The mode runs as a single 15-minute match — no rounds. Final 60 seconds: every score event doubles.

- **Map:** Eastwood (`MP_Eastwood`)
- **Current version:** v0.4.0
- **Status:** Phases 1–4 shipped; Phase 5 (HVT identity card, final-minute UI takeover) and Phase 6 (sudden death overtime, hard-out spectate) deferred
- **Built from:** scratch in TypeScript
- **Inspired by:** BF6 Gauntlet → Vendetta

## Sections

- **[Design & State Machine](design.md)** — slot lifecycle, scoring, end-by-time tiebreakers.
- **[HVT System](hvt-system.md)** — promotion phases, +25 HP buff, WorldIcon ping.
- **[Scoreboard](scoreboard.md)** — five-column CustomFFA layout and the 6-arg cap.
- **[Changelog](changelog.md)** — phase-by-phase version history.

## Quick reference

| Element | Value |
|---|---|
| Map | `MP_Eastwood` |
| Match length | 15 minutes (hard timer, no score limit) |
| HVT slot count | `max(1, squad_count_at_match_start - 2)`, locked at start |
| HVT max HP buff | +25 (default 100 → 125) via `mod.SetPlayerMaxHealth` |
| HVT ping | Permanent WorldIcon (Alert image, head-height) updated every ~0.25s |
| Kill score | +1 per kill |
| HVT kill score | +5 per HVT-kill |
| HVT alive tick | +1 per 5s per live HVT |
| Recovery bonus | +1 on direct chain recovery |
| Final-minute multiplier | 2× on every score event in the last 60s |
| Vehicles | None |
| Tiebreaker order | Score → HVT-kills → total kills → HVT time held |
| Gamemode ObjId | `40010` (DummySpatialProp dispatch token) |
