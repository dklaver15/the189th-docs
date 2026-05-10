# Configuration Reference

Configuration lives in `appsettings.json` (committed) and `appsettings.Production.json` (production overrides). Any value can be overridden by an environment variable using the `Section__Key` convention — e.g. `BotConfig__MinMessages=10`.

Secrets (token, API keys, base64 credentials) are **never** committed. They come from environment variables sourced from the droplet's `.env` file.

## `BotConfig` section

### Authentication & core

| Key | Default | Description |
|---|---|---|
| `Token` | _(env only)_ | Discord bot token. Set via `BotConfig__Token` env var, never in JSON. |

### AWOL system

| Key | Default | Description |
|---|---|---|
| `AwolRoleName` | `AWOL` | Role assigned to inactive users. |
| `HqChannelName` | `awol-list` | Channel where AWOL notifications are posted. |
| `WindowDays` | `28` | Default activity tracking window. |
| `ShortWindowDays` | `14` | Shorter window for roles in `ShortWindowRoles`. |
| `ShortWindowRoles` | `Guest,RCT` | Comma-separated roles that get the shorter window. |
| `MinMessages` | `5` | Min messages within window to stay active. |
| `MinVoiceHours` | `1.0` | Min voice hours within window to stay active. |
| `AwolGraceDays` | `2` | Days after AWOL assignment before HQ is notified. |
| `CheckIntervalMinutes` | `180` | How often `AwolCheckService` runs. |
| `ExemptRoles` | `Admin,Moderator,Retired,Bot,Bot Whisperer` | Comma-separated roles fully exempt from AWOL. |
| `ReserveRoleName` | `Reserve` | Reserve members are exempt from AWOL **and** `/kick-awols`. Tracked separately from `ExemptRoles` to keep the doctrinal meaning explicit. |
| `MaxSingleSessionHours` | `12.0` | Sessions exceeding this are clamped when computing activity. |

!!! note "Threshold logic"
    A user is flagged AWOL when they have **both** fewer than `MinMessages` **and** less than `MinVoiceHours`. Meeting either threshold keeps them safe.

### Google integration

| Key | Default | Description |
|---|---|---|
| `GoogleCredentialsPath` | `google-credentials.json` | Path to service account JSON. |
| `GoogleSpreadsheetId` | _(none)_ | Spreadsheet ID for the gamertag log. |
| `GoogleSheetName` | `Gamertags` | Tab name within the gamertag spreadsheet. |
| `RosterSpreadsheetId` | _(none)_ | Spreadsheet ID for the nightly roster export. Falls back to `GoogleSpreadsheetId` if empty. |
| `RosterSheetName` | `Roster` | Tab name for the roster export. |
| `RosterExportHourUtc` | `6` | UTC hour when the daily roster export runs. |
| `RosterWindowDays` | `30` | Activity window represented in the roster export. |
| `RecruitSpreadsheetId` | _(none)_ | Spreadsheet ID for the recruit log. |
| `RecruitSheetName` | `Recruit Log` | Tab name for the recruit log. |
| `GoogleCalendarId` | _(none)_ | Calendar ID synced with Apollo events. |

!!! warning "Sharing the sheet with the service account"
    The service account email (in the credentials JSON) must be added as an **Editor** on every spreadsheet the bot writes to. Otherwise writes fail silently with a 403.

### Events & voice channels

| Key | Default | Description |
|---|---|---|
| `EventsVoiceChannelName` | `Events` | Display name of the primary events VC. |
| `EventsVoiceChannelId` | `1419092374231056574` | Snowflake of the primary events VC. Used as fallback for sessions recorded before `CategoryId` was added to `VoiceSession`. |
| `EventsCategoryId` | `1428846421368508538` | Category ID; any VC under this category counts toward event attendance. Required because temporary event VCs may be deleted before snapshots run. |
| `EventsTextChannelName` | `events` | Text channel name. |
| `EventsTextChannelId` | `1427507402546217082` | Snowflake of the events text channel. |

### Ranks & promotion

| Key | Default | Description |
|---|---|---|
| `RankRoles` | `RCT,PVT,PFC,...,GA` | Ordered (low → high) list of rank role names. |
| `PlatoonRoles` | `Airborne Platoon,...,Havoc Platoon` | Recognised platoon roles. |
| `CompEventMinRank` | `CPT` | Min rank to use `/comp-event`. |
| `PromoteDemoteMinRank` | `MAJ` | Min rank to use `/promote` and `/demote`. |
| `AwolKickMinRank` | `MAJ` | Min rank to use `/kick-awols`. |
| `BriefingNowMinRank` | `BG` | Min rank to trigger the weekly briefing manually. |

### Auto-promotion

| Key | Default | Description |
|---|---|---|
| `AutoPromotionEnabled` | `true` | Master kill-switch. |
| `AutoPromotionDryRun` | `false` | Global dry-run; logs decisions without applying. |
| `AutoPromotionDryRunRanks` | _(empty)_ | Comma-separated ranks that should run dry while others promote for real. |
| `AutoPromotionRunHourUtc` | `3` | UTC hour for the daily auto-promotion sweep. |
| `AutoPromotionAnnouncementChannel` | `general-chat` | Channel name for promotion announcements. |
| `AutoPromotionAnnouncementChannelId` | `1421928902963494922` | Channel snowflake. |
| `AutoPromotionEventBufferMinutes` | `30` | Buffer before/after an event window for attendance counting. |
| `AutoPromotionMinEventAttendanceMinutes` | `30` | Min minutes attended for an event to count toward promotion. |
| `EventAttendanceSnapshotIntervalMinutes` | `5` | How often the snapshot service samples voice presence. |
| `AttendanceCountingSources` | `Clan` | Comma-separated set of sources counted toward attendance. |

### Reminders

| Key | Default | Description |
|---|---|---|
| `TicketCategoryName` | `TICKET CENTER` | Category containing ticket channels. |
| `TicketReminderDelayHours` | `24.0` | Hours after open before a ticket reminder fires. |
| `GuestReminderDelayDays` | `7` | Days before a Guest is reminded to upgrade. |
| `GuestReminderChannelName` | `general-chat` | Where guest reminders are posted. |
| `OnboardingReminderDelayHours` | `24.0` | Hours after RCT assignment before onboarding reminder fires. |
| `RulesChannelName` | `rules` | Used in onboarding messaging. |

### Disboard bump reminder

| Key | Default | Description |
|---|---|---|
| `BumpReminderEnabled` | `true` | Toggle. |
| `BumpReminderDelayHours` | `2.5` | Delay between Disboard bumps before reminding. |
| `BumpReminderChannelId` | `1419819225287233547` | Where to post the reminder. |
| `DisboardBotId` | `302050872383242240` | Disboard's user ID. |
| `BumpReminderRestartGraceMinutes` | `60` | After bot restart, suppress reminders this many minutes. |

### Apollo

| Key | Default | Description |
|---|---|---|
| `ApolloBotName` | `Apollo` | Display name of the Apollo bot for capture filtering. |

## `Claude` section (Anthropic)

| Key | Default | Description |
|---|---|---|
| `ApiKey` | _(env only)_ | Anthropic API key. Set via `Claude__ApiKey`. |
| `Model` | `claude-sonnet-4-6` | Model used for the weekly briefing. |

## `WeeklyBriefing` section

| Key | Default | Description |
|---|---|---|
| `OfficerChannelId` | `1499843996032438272` | Channel where the briefing is posted. |
| `RunOnDayUtc` | `Sunday` | Day of week (UTC) for the run. |
| `RunAtUtc` | `23:30` | Time of day (UTC, `HH:mm`). |
| `DryRun` | `false` | Generate but don't post. |
| `MaxOutputTokens` | `1500` | Cap on the model's output length. |

## `RedditLeads` section

| Key | Default | Description |
|---|---|---|
| `Enabled` | `true` | Toggle. |
| `PollingIntervalMinutes` | `10` | Poll cadence. |
| `UserAgent` | `ClanGuard/1.0 by xAP3XRONINx` | Required by Reddit. Must be unique and descriptive. |
| `LeadsChannelId` | `1502432175059107973` | Where leads are posted. |
| `Subreddits` | _(see appsettings)_ | Comma-separated subreddit list. |
| `MaxPostAgeHours` | `24` | Ignore posts older than this. |

See [Reddit Leads](reddit-leads.md) for the matcher rules.

## `PatrolWatch` section

| Key | Default | Description |
|---|---|---|
| `Enabled` | `true` | Toggle. |
| `LfgChannelId` | `1410345370910851243` | Channel where squad-formed alerts are posted. |
| `MinSquadSize` | `3` | Minimum players in a VC to count as a squad. |
| `DebounceSeconds` | `12` | Wait period before firing to absorb rapid join/leave churn. |
| `StoodDownLingerSeconds` | `60` | Wait after squad falls below threshold before declaring "stood down". |
| `MatchedGames` | _(see appsettings)_ | List of games tracked, each with display name, activity substrings, accent color, thumbnail. |
| `ExcludedCategoryIds` | `[1428846421368508538]` | Categories that don't fire alerts (e.g. EVENTS). |
| `ExcludedChannelIds` | _(list)_ | Specific channels that don't fire alerts. |

## `Logging` section

Standard .NET logging config. The default is `Information`. Set `Logging__LogLevel__Default=Debug` (or per-namespace) for verbose output.

## Environment variables on the droplet

In `/opt/clanguard/.env`:

```
DISCORD_BOT_TOKEN=...
ANTHROPIC_API_KEY=...
```

These are read by `docker-compose.yml` and passed into the container as `BotConfig__Token` and `Claude__ApiKey`.

## Override precedence

```
appsettings.json  →  appsettings.Production.json  →  environment variables
                                                            ↑ wins
```
