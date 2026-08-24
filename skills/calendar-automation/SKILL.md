---
name: calendar-automation
description: >
  Connect Google or Outlook calendars to MeetStream and have bots join meetings
  automatically. Use when the user says "auto-join my meetings", "connect my
  calendar", "join every meeting on my calendar", "Google Calendar
  integration", "Outlook integration", "schedule a bot for this event",
  "recurring meetings", or wants bots to appear without manual API calls.
---

# Calendar automation

Two MCP tools cover the day-to-day: **`list_calendar_events`** and
**`schedule_calendar_bot`**. Everything else (connecting a calendar, auto-schedule,
disconnect) is REST only.

## The flow

1. **Connect** the calendar once, with OAuth credentials (REST).
2. **Sync/list** upcoming events - `list_calendar_events` (MCP tool).
3. Either **schedule per event** - `schedule_calendar_bot` (MCP tool) - or
   **turn on auto-schedule** so every meeting with a join link gets a bot (REST).

## Connecting

Google: `POST /api/v1/calendar/create_calendar` with `google_client_id`,
`google_client_secret`, `google_refresh_token`.

Outlook / Microsoft 365: `POST /api/v1/calendar/create_outlook_calendar` with the
Microsoft equivalents.

Getting the refresh token is the hard part and it is a one-time setup:

- **Google**: create an OAuth client in Google Cloud Console, add the calendar scopes
  (`calendar.readonly` and `calendar.events.readonly` are the usual minimum), run the
  consent flow once, keep the refresh token.
- **Microsoft**: register an app in Azure Portal, create a client secret, add the
  Microsoft Graph calendar permissions (admin consent may be required), then run the
  OAuth flow.

Walk the user through these interactively and ask them to paste back the values. Do not
guess client ids or invent scopes. Verify with `GET /api/v1/calendar`, which lists the
connected calendars.

## Auto-join every meeting

```
POST /api/v1/calendar/auto-schedule/enable
POST /api/v1/calendar/auto-schedule/disable
GET  /api/v1/calendar/auto-schedule/settings
```

With this on, any event carrying a meeting link gets a bot at its start time, including
newly added ones. This is what people mean by "just join all my meetings".

There is **no** `/calendar/setup-cron` or `/calendar/disable-cron` endpoint. Docs pages
titled "Setup Cron" describe the auto-schedule endpoints above.

## Per-event scheduling

Use the `schedule_calendar_bot` MCP tool, or `POST /api/v1/calendar/schedule/{event_id}`
with a `bot_config`. Unschedule with `DELETE` on the same path.

Scheduling the same event twice returns **HTTP 409** with the existing bot id. Treat
that as "already scheduled", not an error.

Note `bot_config` is a different schema from `create_bot`: it accepts `audio_required`
and puts the transcription provider under a top-level `transcription` key.

## Managing scheduled bots

```
GET    /api/v1/calendar/scheduled_bots            list them
PATCH  /api/v1/calendar/scheduled_bots/{bot_id}   move it - body { "scheduled_join_time": "<ISO>" }
DELETE /api/v1/calendar/scheduled_bots/{bot_id}   cancel it
```

The reschedule field is **`scheduled_join_time`**, not `join_at`. (`join_at` is the
`create_bot` field. They are different endpoints with different names.)

## Recurring meetings

`POST /api/v1/calendar/toggle-recurrence` controls whether a recurring series keeps
getting bots automatically. Be explicit with the user about whether they mean the whole
series or one occurrence - getting this wrong either spams every standup or misses them.

## Disconnecting

`POST /api/v1/calendar/disconnect`. This is destructive: it stops sync, cancels pending
schedules, and removes stored credentials. Show the user what will be lost and confirm
before calling it. There is no per-account `/calendar/connections/...` path; scope the
disconnect via the request body.

## Reference

- [Google Calendar OAuth setup](https://docs.meetstream.ai/guides/calendar-integrations/google-calendar-oauth-setup)
- [Outlook Calendar setup](https://docs.meetstream.ai/guides/calendar-integrations/outlook-calendar-setup)
- [Scheduling bots](https://docs.meetstream.ai/guides/features/scheduling-bots)
- [Create calendar API](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/create-calendar) · [Schedule event](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/schedule-event) · [List scheduled bots](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/list-scheduled-bots)
