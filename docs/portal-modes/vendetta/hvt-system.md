# HVT System

Three subsystems make HVT play viable as a target *and* fair to play:

1. **Promotion** — how a squad's slot fills
2. **+25 HP buff** — extra survivability so HVTs aren't deleted in 0.4 seconds
3. **WorldIcon ping** — permanent head-height marker visible to everyone

## Promotion phases

The active-HVT count vs. the locked slot count determines which promotion rule is in play.

### Bootstrap phase (`activeHvts < hvtSlots`)

The system is "looking to fill slots." Any cross-squad kill by a player whose squad has no current HVT promotes that killer to fill their squad's slot.

- Squad A's player kills Squad B's player → if Squad A has no HVT, Squad A's killer becomes HVT.
- If Squad A already has an HVT, nothing changes.
- Friendly fire kills do not promote (they're treated as a non-kill HVT-loss if the victim was the squad's HVT).

Toast on promotion: *"PlayerName claims an HVT slot"*.

### Locked phase (`activeHvts == hvtSlots`)

All slots are filled. The only way for the slot machine to transition is an HVT death.

- **HVT killed by non-HVT (cross-squad)** → victim's slot clears (Vacant). Killer is *not* promoted automatically; the system drops back into Bootstrap, and the next bootstrap kill fills the slot.
- **HVT killed by HVT (cross-squad)** → victim's slot clears, killer keeps their existing HVT status. No transfer — the HVT count drops by one and the system re-enters Bootstrap.
- **HVT dies to non-kill cause** (fall damage, suicide, friendly fire, disconnect) → slot clears immediately, no killer recorded.

Toast on HVT-kill promotion: *"PlayerName took the HVT slot by elimination"*.

## Recovery bonus

When a squad loses its HVT and then snaps back to fill the slot via a direct chain kill, the promoting kill grants a +1 recovery bonus on top of the standard kill / HVT-kill score. This rewards squads for not leaving open slots for the rest of the match.

Toast on recovery: *"PlayerName recovered the HVT! Bonus point"*.

## +25 HP buff

The HVT's max health goes from the default 100 to 125 via `mod.SetPlayerMaxHealth(player, 125)`. Applied on promotion, removed on demotion / death.

```typescript
// Apply on promotion
mod.SetPlayerMaxHealth(promotedPlayer, 125);
// Heal them up to the new max so the bonus is immediate
mod.SetPlayerCurrentHealth(promotedPlayer, 125);

// Remove on slot clear
mod.SetPlayerMaxHealth(formerHvt, 100);
```

The buff is small enough that good play still wins fights — an HVT eats roughly one extra body shot before going down. Larger buffs (50+) made HVTs feel like sponges in early playtests; +25 is the tested sweet spot.

!!! warning "Heal-up at promotion"
    If you only set max HP without setting current HP, the player walks around with 100/125 — the extra ceiling exists but they're effectively at 80% health. Always heal-up immediately after the max change. Same on demotion: dropping max to 100 doesn't lower current HP if the player was already below 100.

## WorldIcon ping

Every player on the server can see a marker over each live HVT's head. The marker is a `WorldIcon` (Alert image, head-height) parented to the HVT player. It is **always visible** while the HVT is alive — there is no on/off interval.

The icon's position is refreshed via the `HvtIconUpdateLoop` async iterator at roughly 4 Hz (every ~0.25 seconds), which is fast enough that the marker tracks the HVT's movement smoothly.

```mermaid
sequenceDiagram
    participant Slot as Slot system
    participant WIM as WorldIconManager
    participant Loop as HvtIconUpdateLoop
    participant Icon as WorldIcon

    Slot->>WIM: HVT promoted, request icon
    WIM->>Icon: SetWorldIconOwner(hvtPlayer)
    WIM->>Icon: SetWorldIconImage(Alert)
    WIM->>Icon: Show
    loop every ~0.25s
        Loop->>Icon: SetWorldIconPosition(headHeight(hvt.Position))
    end
    Slot->>WIM: HVT lost, release icon
    WIM->>Icon: Hide / disown
```

The `WorldIconManager` class was lifted from CTF v4.5.11 — same pattern, just owning Alert icons instead of flag icons.

### `SetWorldIconOwner` ordering

!!! danger "Owner before everything else"
    `SetWorldIconOwner` must be the first call after creating or repurposing a `WorldIcon`. Calling position / image / color setters first results in the icon attaching incorrectly. See [Portal Scripting Gotchas](../../portal-scripting/gotchas.md#setworldiconowner-must-come-first).

## Per-player stats

Tracked separately from squad scoring (Phase 3, v0.3.0):

| Stat | Increments when… |
|---|---|
| `score` | Per-player accumulator: `+1 per kill, +5 per HVT kill, +1 per 5s while you're an HVT` |
| `hvtKills` | You killed an enemy HVT |
| `kills` | You killed any enemy (including HVTs — these double-count, intentionally) |
| `hvtTime` | Seconds you've personally held HVT status (sum of intervals) |

These appear on the [scoreboard](scoreboard.md). Squad scoring is a separate concept; per-player stats exist purely for individual recognition and tiebreaking.

## Disconnect handling

If an HVT disconnects mid-match, the slot clears immediately as a non-kill cause. Bootstrap resumes for that squad. Toast: *"Squad N HVT disconnected"* (debug-only, not shown to non-officers).

## Friendly fire

If an HVT is killed by a squadmate, the slot clears and the system treats it as a non-kill loss — no promotion to the friendly-fire shooter, no HVT-kill bonus. Toast: *"An HVT was lost to friendly fire"*.

This prevents griefing-via-FF as a way to harvest HVT-kill points within your own squad.
