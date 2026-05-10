# Portal Scripting

Cross-cutting reference for working in the Battlefield 6 Portal TypeScript runtime. Read this **before** starting any Portal mode — most of these pages document things that took hours of debugging to figure out, and the lessons apply to every mode.

## Sections

<div class="grid cards" markdown>

- :material-fire: **[Gotchas](gotchas.md)**

    Things that have actually broken in production. Console.log on PS5, smart punctuation crashes, CRLF line endings, AreaTrigger reliability, the SetScoreboardPlayerValues 6-arg cap, and more.

- :material-puzzle: **[API Quirks](api-quirks.md)**

    Specific Portal APIs that don't behave the way you'd expect. `mod.GetSpatialObject` limitations, `mod.Vector` component requirements, WorldIcon ordering.

- :material-rocket-launch: **[Godot Workflow](godot-workflow.md)**

    Authoring map layouts in Godot, exporting to spatial JSON, the Z-axis sign flip, the `convert_ctf_spatial.py` ObjId fixup, and the duplicate-ObjId silent failure.

- :material-package-variant: **[Build & Deploy](build-deploy.md)**

    TypeScript bundle workflow, uploading to the Portal editor, testing iteration loop.

</div>

## Why this section exists

Portal's editor and runtime have a number of constraints that aren't documented anywhere official:

- Console output behaves differently on PS5 vs. PC test sessions
- The TypeScript parser is finicky about specific characters
- Some APIs have undocumented argument limits that fail at runtime
- The Godot → spatial JSON pipeline has its own set of foot-guns

Every mode the 189th has shipped has tripped over at least three of these. This section is the running record so the next mode doesn't repeat them.
