# Gotchas

Hard-won knowledge from shipping Portal modes. Each entry here cost real time to figure out the first time. Save your future self some hours.

## File format & encoding

### CRLF line endings are required

!!! danger "CRLF, not LF"
    Portal's TypeScript parser expects **CRLF (`\r\n`)** line endings. Files saved with LF (`\n`) — the default on macOS and Linux editors — get rejected or parsed incorrectly. The error messages from the editor are misleading: they often look like syntax errors on lines that look fine.

**Symptoms:**

- Mode fails to load with vague parse errors
- Errors point at lines that are syntactically correct
- Pasting code from a fresh download "fixes" it temporarily until you save again

**Fix:**

- Configure your editor to save with CRLF.
- For VS Code: bottom-right line-endings indicator → CRLF.
- Add a `.gitattributes` to the repo to force CRLF on `.ts` files:
  ```
  *.ts text eol=crlf
  ```
- When automating file edits with Python or shell tooling, **read and write in binary mode** to preserve CRLF. See [Godot Workflow](godot-workflow.md#preserving-crlf-when-editing-with-python).

### Smart punctuation crashes the parser

!!! danger "ASCII only"
    Curly quotes (`"`, `"`, `'`, `'`), em-dashes (`—`), en-dashes (`–`), ellipsis (`…`), and other smart-punctuation characters in TypeScript source files crash the Portal parser. Editors that auto-correct quotes (Pages, Word, default macOS notes) silently insert these.

**Symptoms:**

- File looks syntactically correct
- Parser errors point at character positions that look like normal quotes
- Copy-paste from an external doc breaks the file

**Fix:**

- Use a code editor (VS Code, Cursor, Rider, Nova) for all `.ts` editing — they don't auto-substitute.
- If you have to paste from a doc, paste through a plain-text intermediate first.
- Add a pre-commit grep to catch these:
  ```bash
  grep -P '[\u2018\u2019\u201C\u201D\u2013\u2014\u2026]' src/*.ts && exit 1
  ```

## Console output

### `console.log` doesn't work on PS5

!!! danger "No console.log on PS5"
    `console.log` (and `console.error`, `console.warn`) work in the PC test environment but are **silently dropped on PS5**. Modes that rely on console output for debugging will look broken on console even if they work on PC.

**Implication:**

- You cannot use `console.log` for production telemetry or debugging that needs to work on PS5.
- Use in-world UI (banner messages, scoreboard columns, world text) for any state you need to see during console testing.

**Workaround pattern:**

```ts
// Wrap a debug-banner helper that conditionally compiles to no-op
const DEBUG = false;
function debug(msg: string) {
    if (DEBUG) {
        // banner notification or world text — works on PS5
        ShowBanner(msg);
    }
}
```

## Scoreboard

### `SetScoreboardPlayerValues` hard caps at 6 arguments

!!! danger "Six columns max, hard fail at runtime"
    The Portal API `SetScoreboardPlayerValues` accepts at most 6 values. Passing 7 throws **at runtime, not at parse time**. TypeScript types don't catch this. The error surfaces only when the mode goes live.

**Symptoms:**

- Mode crashes when the scoreboard is rendered
- Error references `SetScoreboardPlayerValues` argument count
- The crash happens at first scoreboard update, not at load

**Plan column layouts under this constraint from the start.** If you need more data, fold related stats into a single column (combined string like `"3K / 1HVT"`) rather than splitting.

## WorldIcon

### `SetWorldIconOwner` must come first

!!! danger "Owner before everything else"
    After creating or repurposing a `WorldIcon`, **`SetWorldIconOwner` must be the first call**. Calling `SetWorldIconPosition`, `SetWorldIconText`, `SetWorldIconColor`, or any other setter before `SetWorldIconOwner` results in the icon attaching to the wrong owner — or no owner — and silently rendering incorrectly (or not at all).

**Correct pattern:**

```ts
const icon = CreateWorldIcon(...);
SetWorldIconOwner(icon, targetPlayer);   // FIRST
SetWorldIconPosition(icon, position);
SetWorldIconText(icon, "HVT");
SetWorldIconColor(icon, RED);
```

**Symptoms when violated:**

- Icon renders for the wrong player (or all players)
- Icon doesn't render at all
- Icon orphans visually after the intended owner moves

## AreaTrigger

### AreaTrigger reliability on PS5

!!! warning "AreaTrigger is unreliable on PS5"
    `AreaTrigger` volumes have been observed to fire inconsistently on PS5 — sometimes missing entries, sometimes firing late, sometimes not at all. The PC test environment doesn't reliably reproduce the issue.

**Affected use cases:**

- Spawn protection volumes (player walks into trigger, gets invuln)
- Capture point detection
- Any "is player inside this region" check that needs to be authoritative

**Workaround:** AABB (axis-aligned bounding box) check in script, evaluated each tick:

```ts
function isInsideAabb(pos: Vector, min: Vector, max: Vector): boolean {
    return pos.X >= min.X && pos.X <= max.X
        && pos.Y >= min.Y && pos.Y <= max.Y
        && pos.Z >= min.Z && pos.Z <= max.Z;
}

// On each tick / position update:
if (isInsideAabb(player.Position, spawnZoneMin, spawnZoneMax)) {
    applySpawnProtection(player);
}
```

This costs a per-tick computation but is deterministic across platforms.

### Duplicate ObjIds break AreaTrigger lookups silently

!!! danger "Duplicate ObjIds silently break the spatial JSON"
    Two `AreaTrigger` (or any node type) entries with the same `ObjId` in a spatial JSON file → the Portal runtime resolves the lookup to one or the other, non-deterministically. **No error is logged.** The mode just behaves as if half its triggers don't exist.

**How this happens:**

- Duplicating a node in Godot copies the ObjId
- Hand-editing JSON and forgetting to renumber
- Merging spatial JSON files

**Detection:**

- Run the [`convert_ctf_spatial.py`](godot-workflow.md#convert_ctf_spatialpy) script — it detects duplicates and renumbers.
- For ad-hoc check:
  ```bash
  jq '[.objects[].ObjId] | group_by(.) | map(select(length > 1))' spatial.json
  ```

## Vector components

### `mod.Vector` requires explicit component names

!!! warning "X/Y/Z don't work; XComponentOf does"
    The Portal `mod.Vector` API doesn't accept positional `(x, y, z)` constructors and doesn't expose `.X`, `.Y`, `.Z` directly the way you'd expect. You read components via `XComponentOf(v)` / `YComponentOf(v)` / `ZComponentOf(v)`.

**Pattern:**

```ts
const pos = player.Position;
const x = XComponentOf(pos);
const y = YComponentOf(pos);
const z = ZComponentOf(pos);

// Construction (verify against current API)
const v = mod.Vector(x, y, z);   // or whatever the constructor pattern is
```

If your IDE autocompletes `pos.X` and the code "looks right" but doesn't work, this is the cause.

## `mod.GetSpatialObject` limitations

### Doesn't return positions for `AreaTrigger` or `InteractPoint`

`mod.GetSpatialObject()` works for visible props (`ComputerMonitor`, etc.) but **doesn't retrieve positions for `AreaTrigger` or `InteractPoint`** node types. Even if those nodes have valid positions in the spatial JSON, the API returns null/undefined for the position.

**Workaround:**

1. Author a visible prop (like `ComputerMonitor`) at the desired location in Godot.
2. Read its position at runtime via `GetSpatialObject`.
3. Spawn the `InteractPoint` / `AreaTrigger` programmatically from that position with a calculated offset.

This is the pattern used by [CTF team switch stations](../portal-modes/ctf/team-switch.md).

## Vehicle handling

### Seat events spam errors with null seat indices

The vehicle seat-event handler occasionally fires with null/undefined seat indices. If unguarded, this spams errors to the console (or whatever telemetry path is active) and can hide real errors.

**Fix:**

```ts
function onSeatChange(player, vehicle, seatIndex) {
    if (seatIndex == null) return;   // guard early
    // ... rest of handler
}
```

## Splash screens

### Splash screen indefinite display

There's been a reported issue with splash screens not dismissing reliably on round transition. If your mode shows a splash on round start and it persists past the transition, this may be the same bug. Verify whether reproducible in current Portal builds.

**Workaround if you hit it:** explicit hide call on every state transition, even when the state machine "shouldn't" need it.

## Quick reference card

| Don't do | Do |
|---|---|
| Save with LF line endings | CRLF (`\r\n`) |
| Use smart quotes / em-dashes | ASCII only in `.ts` files |
| Rely on `console.log` for PS5 debugging | In-world UI |
| Pass 7+ values to `SetScoreboardPlayerValues` | Plan for 6 columns |
| Set WorldIcon properties before `SetWorldIconOwner` | Owner first, always |
| Trust `AreaTrigger` on PS5 | AABB check in script |
| Duplicate ObjIds across nodes | Run the conversion script |
| Access `vec.X` directly | `XComponentOf(vec)` |
| `GetSpatialObject` on `AreaTrigger` / `InteractPoint` | Spawn from a visible prop's position |
