# Reminders & Bumps

A handful of background services and handlers that nudge people on a delay: ticket replies, guest follow-ups, RCT onboarding, and Disboard server bumps.

## Ticket reminders

| Component | Detail |
|---|---|
| Handler | `TicketReminderHandler.cs` |
| Entity | `TicketReminder` |
| Trigger | New ticket channel created in `TicketCategoryName` (default `TICKET CENTER`) |
| Delay | `TicketReminderDelayHours` (default 24.0) |
| Cleared by | Officer reply in the ticket, or ticket channel deletion |

When a new channel appears under the ticket category, a `TicketReminder` row is created with `RemindAt = now + delay`. A periodic check fires reminders to officers when `RemindAt` has elapsed and no officer has replied yet.

## Guest reminders

| Component | Detail |
|---|---|
| Handler | `GuestReminderHandler.cs` |
| Entity | `GuestReminder` |
| Trigger | Guest role assigned |
| Delay | `GuestReminderDelayDays` (default 7) |
| Posted to | `GuestReminderChannelName` (default `general-chat`) |

Reminds a Guest after a week to consider upgrading. Cleared if they leave Guest status (promote to RCT, kick, leave server) before the delay elapses.

## Onboarding reminders

| Component | Detail |
|---|---|
| Handler | `OnboardingReminderHandler.cs` |
| Entity | `OnboardingReminder` |
| Trigger | RCT role assigned |
| Delay | `OnboardingReminderDelayHours` (default 24.0) |
| Posted to | RCT's DMs (best-effort) or fallback channel |

Nudges an RCT to read the rules (`RulesChannelName`, default `rules`), set their gamertags via `/gamertags`, and complete onboarding. Cleared early if the RCT saves gamertags via `/gamertags` (the gamertag handler notifies the onboarding handler explicitly — this is why `OnboardingReminderHandler` is registered before `GamertagCommandHandler` in `Program.cs`).

## Bump reminders (Disboard)

| Component | Detail |
|---|---|
| Handler | `BumpReminderHandler.cs` |
| Entity | `BumpState` |
| Trigger | Disboard bot (`DisboardBotId = 302050872383242240`) confirms a successful bump |
| Delay | `BumpReminderDelayHours` (default 2.5) |
| Posted to | `BumpReminderChannelId` |

Disboard's `/bump` slash command boosts the server's listing on disboard.org, with a 2-hour cooldown. We wait an extra 30 minutes (2.5h total) and remind whoever's around to bump again.

`BumpState` records the last successful bump time per guild. The reminder loop checks "is it time?" against `LastBumpAt + BumpReminderDelayHours`.

!!! note "Restart grace"
    `BumpReminderRestartGraceMinutes` (default 60) suppresses the reminder for the first hour after bot startup. This avoids spamming a reminder if the bot was offline through the bump's natural reminder time and just came back up.

## Common operational questions

??? question "Reminder fired but the user already replied / handled it."
    Possible races between "reply happened" and "reminder loop fired the message". Mitigation: check the relevant entity table for whether the reminder is still flagged as pending. If yes → the resolution event didn't get persisted. If no → the reminder fired before persistence completed.

??? question "Disboard bump reminders never appear."
    1. Confirm Disboard is still bumping (its own confirmation message must be parseable).
    2. Confirm `DisboardBotId` matches Disboard's user ID — if Disboard re-deploys with a new bot account, this changes.
    3. Confirm `BumpReminderEnabled = true` and `BumpReminderChannelId` is correct.

??? question "Onboarding reminder fired but RCT had already set gamertags."
    The gamertag handler explicitly notifies the onboarding handler on save, but only when the user holds the Guest or RCT role at save time. If gamertags were set before RCT was assigned, no notification fires; the reminder will run on schedule. This is rare in practice but worth knowing.
