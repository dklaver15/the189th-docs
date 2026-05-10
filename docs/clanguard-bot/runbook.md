# Runbook

When something is broken or behaving oddly, start here. Each section is "symptom → diagnosis → fix".

## Bot is offline / not responding

**Symptom:** No response to slash commands, "Bot is offline" indicator in Discord member list.

```bash
ssh root@<droplet>
docker compose ps                   # is the container even running?
docker compose logs --tail=200 clanguard
```

Common causes:

| What logs show | Cause | Fix |
|---|---|---|
| `Cannot connect to Discord` / `WebSocketException` | Token revoked or rotated | Update `DISCORD_BOT_TOKEN` in `/opt/clanguard/.env`, `docker compose up -d --force-recreate` |
| `Used disallowed intents` | Privileged intents disabled in dev portal | Re-enable Presence / Members / Message Content intents |
| `database is locked` | WAL contention or stuck process | Restart the container |
| `Cannot find google-credentials.json` | Mount missing or file empty | Re-decode from `GOOGLE_CREDENTIALS_BASE64` secret |
| Container exits immediately | Missing env var or bad config | Check `docker compose logs` for the .NET stack trace |

## Deploy succeeded but old code is still running

Almost always a stale container. Force a clean rebuild:

```bash
cd /opt/clanguard
docker compose build --no-cache
docker compose up -d --force-recreate
docker compose logs --tail=50
```

If that still doesn't pick up the change:

```bash
docker compose down
docker image prune -f
docker compose up -d --build
```

!!! danger "Don't `docker system prune -a`"
    That deletes the named volumes — including `bot-data` — and you lose the database. Always scope prunes to images.

## AWOL list keeps showing the same notification

**Symptom:** Same user gets posted to the AWOL channel every check cycle.

Possible causes:

1. **`NotificationSent` column not being written.** Check `AwolRecord` for the user — `NotificationSent` should be `1` once posted.
2. **Missing channel permissions.** If posting silently fails, `LastNotificationAttemptUtc` will keep updating but `NotificationSent` stays `0`. After 7 days the record auto-resolves as "given up" — see `AwolCheckService`.

```sql
SELECT UserId, Username, AssignedAt, NotificationSent, LastNotificationAttemptUtc
FROM AwolRecord
WHERE NotificationSent = 0
ORDER BY AssignedAt DESC;
```

## Auto-promotion didn't run / promoted the wrong people

Auto-promotion runs once a day at `AutoPromotionRunHourUtc` (default `3` UTC). To verify a run happened:

```bash
docker compose logs clanguard | grep -i "AutoPromotion"
```

The `BotState` table records cycle completion. If `AutoPromotionEnabled=false` or `AutoPromotionDryRun=true` in config, no announcements happen.

To re-trigger manually, restart the container after the configured hour. There is currently no "run now" command for auto-promotion.

!!! note "Dry-run vs. dry-run-ranks"
    `AutoPromotionDryRun=true` makes every tier dry-run. `AutoPromotionDryRunRanks=PVT,PFC` makes only those tiers dry-run while other tiers continue to promote for real. Use the per-tier flag when rolling out a tier change.

## Apollo events aren't being captured

**Symptom:** New Apollo posts in `#events` aren't appearing on the calendar or in attendance.

1. Confirm the Apollo bot user matches `BotConfig.ApolloBotName` (default `Apollo`).
2. Check `ApolloMessageLog` for the most recent entry — if recent, the capture handler is working but the parser is failing.
3. Check logs: `grep -i Apollo logs/clanguard-*.log`

Known parser landmines:

- **Smart punctuation in event titles** crashes the embed parser. Strip curly quotes / em-dashes before posting if seen.
- **Apollo edits the same message** to update RSVP counts. The bot needs `MessageCacheSize=500` and `GuildMessages` intent for `MessageUpdated` to fire.
- **Apollo embed timestamps** are parsed in `ApolloEmbedParser` — if the embed format changes, the parser may need updating.

## Voice tracking is wrong

**Symptom:** A user has voice time recorded but they were definitely never in voice / vice versa.

Most likely cause: bot restarted while the user was in voice. The `VoiceSession` row for that session never got a `LeftAt` timestamp.

`VoiceSessionCleanupService` reconciles dangling sessions on startup, but if the user joined and left during a downtime window, the session is lost entirely.

To audit one user:

```sql
SELECT JoinedAt, LeftAt, ChannelName, ROUND((julianday(COALESCE(LeftAt, datetime('now'))) - julianday(JoinedAt)) * 24, 2) AS hours
FROM VoiceSession
WHERE UserId = <user-id>
ORDER BY JoinedAt DESC
LIMIT 20;
```

To cap a single session that ran absurdly long:

```sql
-- MaxSingleSessionHours defaults to 12. Sessions exceeding this are auto-trimmed
-- by AwolCheckService when computing activity, but the raw row is unchanged.
```

## Roster export to Google Sheets is empty / outdated

Runs once a day at `RosterExportHourUtc` (default `6` UTC).

1. Verify `RosterSpreadsheetId` and `RosterSheetName` in config.
2. Check that `google-credentials.json` is mounted and the service account has **edit** access on the sheet (share the sheet with the service account email).
3. Logs: `grep -i RosterExport logs/clanguard-*.log`

To re-export immediately, run `/roster-export` (officer-only).

## /promote or /demote fails silently

The bot's own role must be **above** every rank role it's modifying. Check **Server Settings → Roles** and drag the bot's role up.

Other failure modes:

- User running the command lacks the `PromoteDemoteMinRank` rank (default `MAJ`).
- Target user is exempt (`Bot`, `Retired`, etc.).
- Target rank doesn't appear in the `RankRoles` config list.

## Reddit Leads stopped posting

```bash
docker compose logs clanguard | grep -i Reddit
```

- `403` from Reddit → User-Agent is being rejected. Reddit requires a unique, descriptive UA. Check `RedditLeads.UserAgent`.
- `429` → rate-limited. Increase `PollingIntervalMinutes`.
- No errors but no posts → `LeadMatcher` filters may be too strict, or no recent posts match.

## Briefing never posted on Sunday

`WeeklyOfficerBriefingService` runs at `WeeklyBriefing.RunOnDayUtc` / `RunAtUtc`. Check:

1. `Claude__ApiKey` is set in the environment.
2. `WeeklyBriefing.OfficerChannelId` is correct and the bot can post there.
3. `DryRun` isn't `true`.
4. Anthropic API hasn't returned an error — search logs for `Claude` or `Anthropic`.

To test on demand: `/briefing-now` (BG+ only).

## Database is corrupt

```bash
docker exec -it clanguard-bot sqlite3 /app/data/clanguard.db "PRAGMA integrity_check;"
```

If it returns anything other than `ok`:

1. Stop the bot: `docker compose stop clanguard`
2. Restore from the most recent `~/clanguard-backups/backup-*.db`:
   ```bash
   docker cp ~/clanguard-backups/backup-YYYYMMDD.db clanguard-bot:/app/data/clanguard.db
   ```
3. Restart: `docker compose start clanguard`

## "I just need to see what the bot is doing right now"

```bash
docker compose logs -f --tail=100 clanguard
```

Filter for a specific subsystem:

```bash
docker compose logs --tail=500 clanguard | grep -i "AutoPromotion\|Apollo\|Awol"
```
