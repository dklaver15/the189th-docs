# Slash Commands

Every command is registered in `DiscordBotService.cs` from the corresponding `*CommandHandler.cs`. Permission gates are enforced inside the handler — usually by checking `BotConfig.PromoteDemoteMinRank` / `AwolKickMinRank` / `CompEventMinRank` / `BriefingNowMinRank` against the invoker's rank.

## Member commands

| Command | Handler | Description |
|---|---|---|
| `/awol-status` | `SlashCommandHandler` | Check your own activity stats (messages, voice hours, AWOL state). |
| `/gamertags` | `GamertagCommandHandler` | Save or update your platform gamertags. Writes to the gamertag sheet. |
| `/lookup <user>` | `LookupCommandHandler` | Look up another member's gamertags from the roster sheet (ephemeral). |
| `/calendar` | `CalendarCommandHandler` | Show upcoming events from the synced Google Calendar. |
| `/patrol info` | `PatrolWatchCommandHandler` | Show your current Patrol Watch opt-out state. |
| `/patrol on` / `/patrol off` | `PatrolWatchCommandHandler` | Opt in/out of squad-formed alerts. |

## Officer / NCO commands

| Command | Min rank | Handler | Description |
|---|---|---|---|
| `/awol-check <user>` | NCO+ | `SlashCommandHandler` | Check another user's activity stats. |
| `/clear-awol <user>` | NCO+ | `SlashCommandHandler` | Remove the AWOL role and resolve any pending notifications for a user. |
| `/attendance` | varies | `AttendanceCommandHandler` | Pull event attendance for a given window/user. |
| `/comp-event` | `CompEventMinRank` (CPT) | `CompEventCommandHandler` | Create a competitive event entry on the calendar. |

## Senior officer commands

| Command | Min rank | Handler | Description |
|---|---|---|---|
| `/promote <user> [rank]` | `PromoteDemoteMinRank` (MAJ) | `PromoteCommandHandler` | Manually promote a user. |
| `/demote <user> [rank]` | `PromoteDemoteMinRank` (MAJ) | `DemoteCommandHandler` | Manually demote a user. |
| `/setnick <user> <nick>` | MAJ+ | `SetNickCommandHandler` | Force-set a member's server nickname. |
| `/kick-awols` | `AwolKickMinRank` (MAJ) | `KickAwolsCommandHandler` | Kick all members currently flagged AWOL whose grace period has elapsed. Reserve role exemption applies. |
| `/clear-awol-list` | MAJ+ | `ClearAwolListCommandHandler` | Clear the entire pending AWOL queue. |
| `/add-event-credit <user> <count>` | MAJ+ | `EventCreditCommandHandler` | Add ad-hoc event attendance credit. |
| `/remove-event-credit <user> <count>` | MAJ+ | `EventCreditCommandHandler` | Remove event attendance credit. |
| `/seed-promotion-credit` | MAJ+ | `SeedPromotionCreditCommandHandler` | One-time backfill: read the "Seed Events" column from the roster sheet and write into `RankHistory.EventsAttendedAtRankBeforeBot`. |
| `/roster-export` | MAJ+ | _(in `SlashCommandHandler`)_ | Force a roster export to Google Sheets immediately. |
| `/cleanup-calendar-dupes` | MAJ+ | `CleanupCalendarDupesCommandHandler` | Remove duplicate calendar events created by Apollo edits. |
| `/window <user> [days]` | MAJ+ | _(in `SlashCommandHandler`)_ | Override the activity window for a single user. |

## Command-group commands

### `/invite`

`InviteCommandHandler`. Subcommands:

- `/invite list` — list tracked invite codes and their attribution
- `/invite recent` — show recent joins with attributed source
- `/invite stats` — top inviters
- `/invite top` — leaderboard
- `/invite assign <code> <source>` — assign an invite code to a named source
- `/invite revoke <code>` — clear an attribution
- `/invite create <source>` — generate a new attributed invite

### `/leads`

`RedditLeadsCommandHandler`. Subcommands:

- `/leads list` — recent leads
- `/leads recent` — same, scoped to a window
- `/leads filter <subreddit>` — filter by subreddit
- `/leads stats` — match stats
- `/leads edit-notes` — edit lead notes

### `/patrol`

`PatrolWatchCommandHandler`. Subcommands listed in [Member commands](#member-commands).

### `/usage-stats`

`UsageStatsCommandHandler`. Subcommands:

- `/usage-stats command <command>` — usage stats for one command
- `/usage-stats user <user>` — usage stats for one user
- `/usage-stats top` — top commands by usage

### `/command-catalog`

`CommandsCommandHandler`. Auto-generated catalog of every command, with optional `style` (compact/detailed) and `view` (member/officer) filters.

## Senior leadership

| Command | Min rank | Description |
|---|---|---|
| `/briefing-now` | `BriefingNowMinRank` (BG) | Trigger the weekly officer briefing immediately, bypassing the schedule. |

## Deferred / placeholder commands

These are referenced in code but currently effectively superseded:

- `/recruit` — **removed**. Recruits are now auto-logged by `RankTrackingHandler` when a member gains the RCT role.

## Permission gates: how they work

```csharp
// Pseudocode of the standard pattern
var minRank = config.PromoteDemoteMinRank;
if (!UserMeetsMinRank(invoker, minRank, config.RankRoles))
{
    await interaction.RespondAsync("You don't have permission for this command.", ephemeral: true);
    return;
}
```

`RankRoles` is the ordered list (low → high). The check resolves the invoker's highest matching rank and compares its index against the index of `minRank`.

!!! warning "Rank role hierarchy"
    The bot's own role must sit **above** every rank role in the Discord role hierarchy or it can't add/remove them. This is the single most common cause of `/promote` "succeeding" but doing nothing.

## Adding a new command

1. Create `MyNewCommandHandler.cs` modelled after an existing one (e.g. `LookupCommandHandler.cs`).
2. Define a public `const string CommandName` for the slash command name.
3. Implement a `Register()` method called once from `DiscordBotService` to build and register the command via `Discord.Net`.
4. Implement an `HandleAsync(SocketSlashCommand cmd)` method called from the central `InteractionCreated` dispatch in `DiscordBotService`.
5. Register the handler as a singleton in `Program.cs`.
6. Add the command to `DiscordBotService.RegisterCommands()` and to `DiscordBotService.HandleInteractionAsync()` switch.
