# CTF (Single-Flag)

189th's custom single-flag capture-the-flag mode. One neutral flag in the middle of the map; teams race to capture it back to their base. Built in TypeScript against the Portal runtime, with map layouts authored in Godot and converted to Portal spatial JSON.

## Maps

| Map | Internal name |
|---|---|
| Abbasid | `MP_Abbasid` |
| Hagental | `MP_Hagental` |
| Outskirts | `MP_Outskirts` |
| Subsurface | `MP_Subsurface` |

Each map has its own Godot scene and exported spatial JSON. The TypeScript code is map-agnostic; map-specific data (flag position, base positions, team-switch station positions) comes from the spatial JSON.

## Sections

- **[Design](design.md)** — flag flow, scoring, post-capture cooldown, scoreboard.
- **[Team Switch Stations](team-switch.md)** — runtime-spawned interaction points for swapping teams mid-match.

## Key features

- Single neutral flag (not symmetric two-flag)
- **Post-capture cooldown** — after a successful capture, the flag stays at base briefly before respawning at center
- **Per-player scoreboard** with custom columns
- **Player name announcements** for flag pickup, drop, and capture (added in v4.5.9–v4.5.11)
- **Buried CapturePoint minimap marker** — the flag location pings on the minimap (gotcha: this required a workaround discovered via Mike on the BFPortal Discord)
- **Clan info embed** with the Discord invite URL, shown in the round preamble
- **Team switch stations** — interactable props (`ComputerMonitor`) at each base that let players swap to the other team
