# Google Sheets & Calendar

The bot uses a single Google service account to read/write three resources:

| Resource | Used by |
|---|---|
| Gamertag spreadsheet (`GoogleSpreadsheetId`) | `/gamertags` writes, `/lookup` reads |
| Roster spreadsheet (`RosterSpreadsheetId`) | Nightly `RosterExportService`, `/seed-promotion-credit` reads "Seed Events" column |
| Recruit log spreadsheet (`RecruitSpreadsheetId`) | `RankTrackingHandler` writes when a member gains the RCT role |
| Google Calendar (`GoogleCalendarId`) | Apollo event sync; `/comp-event` writes |

## Authentication

`google-credentials.json` is a service account credentials file. It's mounted into the container read-only at `/app/google-credentials.json` from the path `BotConfig.GoogleCredentialsPath` (default `google-credentials.json`).

In production it's deployed as a base64-encoded GitHub Actions secret (`GOOGLE_CREDENTIALS_BASE64`), decoded onto the droplet during the deploy workflow.

!!! danger "Service account is not the bot owner"
    The service account is a separate Google identity. It doesn't have access to anything by default — every spreadsheet and calendar must be **explicitly shared** with the service account email (found in the credentials JSON as `client_email`).

    For spreadsheets: share as **Editor**.
    For the calendar: share with **"Make changes to events"** permission.

## Components

| File | Role |
|---|---|
| `GoogleSheetService.cs` | All Google Sheets reads/writes. Wraps `Google.Apis.Sheets.v4`. |
| `GoogleCalendarService.cs` | All Google Calendar reads/writes. Wraps `Google.Apis.Calendar.v3`. |
| `RosterExportService.cs` | Daily roster export at `RosterExportHourUtc`. |
| `GamertagCommandHandler.cs` | `/gamertags` write path. |
| `LookupCommandHandler.cs` | `/lookup` read path. |
| `SeedPromotionCreditCommandHandler.cs` | One-time `/seed-promotion-credit` read of the Seed Events column. |
| `CompEventCommandHandler.cs` | `/comp-event` calendar event creation. |

## Roster export

Runs daily at `RosterExportHourUtc` (default 6 UTC). Writes the full roster to `RosterSheetName` (default `Roster`) on `RosterSpreadsheetId`.

Columns include:

- Discord display name
- Rank
- Time-in-rank
- Activity window stats (messages + voice hours over `RosterWindowDays`)
- Last event attended
- Gamertags
- Promotable column (boolean indicator that downstream auto-promotion would qualify them)

When the export detects a rank change for a user (their current Discord roles don't match their last `RankHistory` row), it:

1. Writes a fresh `RankHistory` row.
2. **Resets** `EventsAttendedAtRankBeforeBot` to `0` and `SeedAppliedAt` to `null`. (See [Auto-Promotion seed semantics](auto-promotion.md#seed-fields).)

`/roster-export` (MAJ+) triggers the export immediately.

## Gamertag log

`/gamertags` opens a modal where members fill in their gamertags per platform. On submit, the row is upserted to `GoogleSheetName` (default `Gamertags`) on `GoogleSpreadsheetId`, keyed by user ID.

`/lookup <user>` reads the same sheet and returns the gamertags ephemerally to the invoker.

The handler also notifies `OnboardingReminderHandler` when a Guest saves gamertags, so onboarding can advance without a manual nudge.

## Recruit log

When `RankTrackingHandler` sees a member gain the RCT role, it appends a row to the `RecruitSpreadsheetId` / `RecruitSheetName` sheet (default `Recruit Log`) with:

- Discord username
- Recruited timestamp (Eastern Time)
- Recruiter (best-effort, from invite attribution if available)

This replaced the older `/recruit` slash command.

## Calendar sync

Every parsed `ApolloEvent` becomes a `CalendarEvent` row in the DB and a corresponding event on the Google Calendar identified by `GoogleCalendarId`. The calendar event ID is stored in `CalendarEvent.GoogleEventId` so updates and deletions can address it directly.

`/comp-event` (CPT+) creates a calendar event without going through Apollo — for competitive scrims and other events that aren't posted via Apollo.

!!! warning "Eastern Time hardcoding for /comp-event"
    `/comp-event` parses dates in Eastern Time. The handler uses `DateOnly.TryParseExact` with explicit US date formats to avoid locale-driven misinterpretation (`02/03` is February 3rd, not March 2nd).

## Common operational questions

??? question "Sheet writes are silently failing."
    Almost always a sharing problem. Open the sheet, click **Share**, and confirm the service account email is listed as **Editor**. The email is in `google-credentials.json` under `client_email`.

??? question "Calendar events aren't appearing."
    1. Confirm the calendar is shared with the service account with **"Make changes to events"** permission.
    2. Confirm `GoogleCalendarId` matches the calendar ID (Settings → Integrate calendar → Calendar ID).
    3. Check logs for `GoogleCalendar` errors.

??? question "Apollo posted an event but it's not on the calendar."
    See [Apollo Integration runbook](apollo-integration.md#common-operational-questions). The pipeline goes Apollo → `ApolloMessageLog` → parser → `ApolloEvent` → calendar. Find the first stage where the event is missing.

??? question "How do I rotate the service account?"
    1. Generate a new credentials JSON in the Google Cloud Console.
    2. Base64-encode it: `base64 -i new-credentials.json | tr -d '\n'`
    3. Update the `GOOGLE_CREDENTIALS_BASE64` GitHub Actions secret.
    4. Re-share every spreadsheet and the calendar with the new service account email.
    5. Trigger a deploy. The workflow decodes the secret on the droplet and the next container restart picks it up.

    Old credentials remain valid until you delete them from the Cloud Console — leave them active until after you've confirmed the new ones work.
