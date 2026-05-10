# 189th Technical Docs

Internal reference for the 189th clan's technical work — the Discord bot, custom Battlefield 6 Portal game modes, and the lessons learned along the way.

## What's here

<div class="grid cards" markdown>

- :material-shield-account: **[ClanGuard Bot](clanguard-bot/index.md)**

    The C# Discord bot. Architecture, deployment, runbook, slash commands, and per-subsystem deep-dives (AWOL, auto-promotion, Apollo parsing, Google integration, Patrol Watch, Reddit Leads, weekly briefing).

- :material-flag-checkered: **[Portal Modes](portal-modes/index.md)**

    Custom Battlefield 6 Portal game modes built by the clan. Vendetta (HVT mode) and CTF (single-flag). Design docs, state machines, and feature changelogs.

- :material-code-tags: **[Portal Scripting](portal-scripting/index.md)**

    Cross-cutting reference for working in the Portal TypeScript runtime. Gotchas, API quirks, the Godot spatial-JSON workflow, and build/deploy notes.

</div>

## How to use these docs

- **Search** (top-right or ++slash++) hits every page on the site.
- **Edit on GitHub** is in the top-right of every page — fixes go straight to the repo.
- The left nav follows section structure; the right nav is the current page's table of contents.

## Conventions used here

!!! note "Note"
    General context or background information.

!!! warning "Warning"
    Behaviour that's easy to get wrong but recoverable.

!!! danger "Landmine"
    Things that have actually broken in production. Don't repeat the mistake.

!!! tip "Tip"
    Optional improvements or non-obvious shortcuts.
