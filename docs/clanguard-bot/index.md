# ClanGuard Bot

C# / .NET 8 Discord bot that handles activity tracking, AWOL management, auto-promotion, event attendance, Reddit recruitment leads, and a stack of QoL automations for the 189th.

- **Repo:** `dklaver15/189th-clanbot`
- **Stack:** .NET 8 Worker, Discord.Net 3.17, EF Core (SQLite, WAL mode), Serilog
- **Hosting:** DigitalOcean droplet (`ubuntu-clanbot-nyc3`), Docker Compose
- **Deploy:** GitHub Actions → SSH to droplet → `docker compose up -d --force-recreate`

## Where to start

- New to the codebase? → [Architecture](architecture.md)
- Trying to deploy or debug a deployment? → [Deployment & Infra](deployment.md)
- Something is on fire? → [Runbook](runbook.md)
- Need to know what a config key does? → [Configuration Reference](configuration.md)
- Looking up a slash command? → [Slash Commands](commands.md)
- Tracing data through the DB? → [Database Schema](database.md)

## Subsystems at a glance

| Subsystem | Purpose | Entry points |
|---|---|---|
| [AWOL](awol-system.md) | Flag inactive members, kick after grace period | `AwolCheckService`, `KickAwolsCommandHandler` |
| [Auto-Promotion](auto-promotion.md) | Tier-based automatic rank advancement | `AutoPromotionService`, `PromotionService` |
| [Apollo Integration](apollo-integration.md) | Parse Apollo bot embeds for event tracking | `ApolloEventHandler`, `ApolloEmbedParser` |
| [Event Attendance](event-attendance.md) | Snapshot voice presence during events | `EventAttendanceSnapshotService` |
| [Google Sheets/Calendar](google-integration.md) | Roster export, gamertag log, calendar sync | `GoogleSheetsService`, `GoogleCalendarService` |
| [Patrol Watch](patrol-watch.md) | Detect when squads form in LFG voice | `PatrolWatchService` |
| [Reddit Leads](reddit-leads.md) | Recruitment lead detection from subreddits | `RedditLeadService`, `LeadMatcher` |
| [Weekly Briefing](weekly-briefing.md) | AI-generated officer briefing every Sunday | `WeeklyOfficerBriefingService` |
| [Reminders & Bumps](reminders.md) | Tickets, guests, onboarding, Disboard bumps | `*ReminderHandler`, `BumpReminderHandler` |
| [Invites & Attribution](invite-tracking.md) | Track which invite link a join came from | `InviteCacheService`, `InviteAttributionService` |
