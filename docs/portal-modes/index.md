# Portal Modes

Custom Battlefield 6 Portal game modes built by the 189th. All modes are written in TypeScript against the Portal scripting runtime, with map layouts authored in Godot and exported as spatial JSON.

## Modes

<div class="grid cards" markdown>

- :material-target: **[Vendetta](vendetta/index.md)**

    HVT-based mode: each round one player is randomly chosen as the High Value Target. Killing the HVT scores big; the HVT gets buffs to compensate. Built from scratch in TypeScript, currently at v0.3.0. Eastwood map.

- :material-flag: **[CTF](ctf/index.md)**

    Single-flag capture-the-flag with team switch stations, post-capture cooldown, per-player scoreboard, and clan info embed. Multiple maps targeted (MP_Abbasid, MP_Hagental, MP_Outskirts, MP_Subsurface).

</div>

## Cross-cutting reference

If you're working on any Portal mode, the common pitfalls and runtime constraints live in **[Portal Scripting](../portal-scripting/index.md)**. Read that first; it'll save hours.

## Build & deploy

All modes build to a single TypeScript bundle that gets uploaded to the Portal editor. See [Build & Deploy](../portal-scripting/build-deploy.md).
