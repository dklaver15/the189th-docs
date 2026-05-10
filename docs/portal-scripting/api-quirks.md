# API Quirks

Specific Portal APIs that don't behave the way you'd expect from the type signatures or general documentation. Many of these are also covered in [Gotchas](gotchas.md); this page is the reference-card view.

## `mod.Vector` and its components

### Constructing a vector

Don't use positional `(x, y, z)` access if you're inspecting the result via `.X` / `.Y` / `.Z` — the Portal runtime exposes components through accessor functions, not member fields.

### Reading components

```ts
const pos = player.Position;

// These do NOT work:
const x_wrong = pos.X;
const { X, Y, Z } = pos;

// This is the correct way:
const x = XComponentOf(pos);
const y = YComponentOf(pos);
const z = ZComponentOf(pos);
```

### Writing / mutating

`mod.Vector` instances are not always trivially mutable. Construct a new vector with the desired components rather than trying to set components in place.

## `mod.GetSpatialObject`

Returns the runtime handle for a named object in the loaded spatial JSON.

### Works for

- Visible props with model geometry (`ComputerMonitor`, light fixtures, etc.)
- Most static placeable entities

### Doesn't work for

- **`AreaTrigger`** — returns null/undefined position even when the node exists in the JSON with a valid position
- **`InteractPoint`** — same as AreaTrigger

### Workaround

Author a visible prop at the location, read its position from `GetSpatialObject`, then **spawn** the `AreaTrigger` / `InteractPoint` at that position programmatically:

```ts
const anchor = mod.GetSpatialObject("MyTriggerAnchor");
const pos = anchor.Position;   // a ComputerMonitor or similar visible prop
const trigger = mod.CreateAreaTrigger(pos, /* dimensions */);
```

This is the pattern used in [CTF team switch stations](../portal-modes/ctf/team-switch.md).

### Lookup name conventions

Names in the spatial JSON's `Name` field must match exactly (case-sensitive). Recommended convention:

```
<Mode>_<Purpose>_<Variant>
```

Examples:

- `CTF_TeamSwitchMonitor_TeamA`
- `CTF_FlagSpawnAnchor_Center`
- `Vendetta_HVTPingAnchor`

## `WorldIcon` API ordering

After creating or repurposing a `WorldIcon`, **`SetWorldIconOwner` must be the first call**. Other setters (position, text, color) called before SetOwner will not behave correctly.

```ts
const icon = CreateWorldIcon(...);
SetWorldIconOwner(icon, player);   // must be first
SetWorldIconPosition(icon, position);
SetWorldIconText(icon, "Label");
SetWorldIconColor(icon, RED);
```

If you reuse a `WorldIcon` for a new owner mid-match, call `SetWorldIconOwner` first again before re-setting other properties.

## `SetScoreboardPlayerValues`

Hard-capped at **6 values**. Passing more throws at runtime, not at parse time. There is no compile-time check.

Plan column count up front. To convey more data than 6 columns can carry:

- Combine related values into a single string column (`"3K / 1HVT"`)
- Cycle column meaning over time (alternating displays — UX cost, but possible)

## Console output

`console.log`, `console.error`, `console.warn` all work in PC test sessions and **do not work on PS5**. Don't rely on them for production debugging.

## TypeScript parser

The Portal TypeScript parser:

- Requires CRLF line endings
- Rejects smart-punctuation characters (`"` `"` `'` `'` `—` `–` `…`)
- Some non-ASCII characters in strings are accepted (verify per-character if you hit issues)

These constraints are not standard TypeScript — they're specific to the Portal runtime's parser.

## Vehicle seat events

Seat-change events occasionally fire with null or undefined `seatIndex`. Guard your handler:

```ts
function onSeatChange(player, vehicle, seatIndex) {
    if (seatIndex == null) return;
    // ... real handler
}
```

Without the guard, the runtime emits an error per spurious event, drowning useful output.

## Player object

### Position

`player.Position` is a `mod.Vector`. Read components via `XComponentOf` / `YComponentOf` / `ZComponentOf` (see [above](#modvector-and-its-components)).

### Team / faction

Team assignment APIs vary by mode setup. Verify the actual call signature in the current Portal docs — names and signatures have shifted between Portal updates.

### Health

Setting `player.MaxHealth` and `player.CurrentHealth` works. Be aware that some health-change paths trigger UI animations; setting both in the same tick keeps the bar from flashing.

## Coordinates and axes

Portal world coordinates use a left-handed system with Y as up. Godot uses a different convention — see [Godot Workflow § Z-axis flip](godot-workflow.md#z-axis-must-be-negated).

- **X**: east-west (or whatever the map's primary axis is)
- **Y**: up-down
- **Z**: north-south (negated relative to Godot's Z)

Bearings (yaw rotation) are in degrees, with 0° pointing along the +X (or +Z, depending on convention) axis. Verify per-map by placing a known-orientation prop and inspecting in-game.

## What this list isn't

This page documents quirks **observed in production**. The official Portal API reference (in the editor) is still the source of truth for full method signatures and parameter types — read both.
