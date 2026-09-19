---
name: platform-setup
description: >
  Platform-specific setup and quirks for Zoom, Google Meet and Microsoft
  Teams. Use when the user says "the bot can't join Zoom", "waiting room",
  "the host has to admit it", "recording permission", "Zoom app setup", "OBF",
  "signed-in bots", "the bot shows as guest", "Teams lobby", or hits a
  platform-specific failure rather than an API error.
---

# Platform setup and quirks

Most "the bot didn't join" reports are platform behaviour, not API failures. Diagnose
from the terminal webhook first: every ending arrives with `event: "bot.stopped"`, and
**`bot_event`** says why.

| `bot_event` | Meaning | Fix |
|---|---|---|
| `bot.notallowed` | Sat in the waiting room until timeout | Have someone admit it, raise `waiting_room_timeout`, or use a signed-in bot |
| `bot.denied` | A host actively refused it | Human decision. Do not auto-retry |
| `bot.kicked` | A participant removed it | Human decision. Do not auto-retry |
| `bot.failed` | Session crashed | Create a fresh bot |

Without webhooks, `get_bot_status` shows the same outcome as `NotAllowed`, `Denied` or `Error`.

## Automatic leave timeouts

Set on the REST `create_bot` request under `automatic_leave`. The MCP `create_bot` tool
does not expose these, so use the REST API when you need them:

| Field | Notes |
|---|---|
| `waiting_room_timeout` | How long to wait for admission |
| `everyone_left_timeout` | Leave after the humans go |
| `voice_inactivity_timeout` | Leave on silence |
| `in_call_recording_timeout` | **Minimum 600 seconds** - below that the API returns HTTP 400 |
| `recording_permission_denied_timeout` | **Zoom only**, accepted range 60-300 |

## Zoom

Zoom does the one thing the others do not: it can require **explicit recording
permission** from the host. If nobody grants it,
`recording_permission_denied_timeout` decides how long the bot waits before giving up.

Zoom also needs app-level setup before bots can join meetings outside your own account:

- A Zoom Marketplace app, with credentials added on the MeetStream side.
- Development mode restricts bots to meetings hosted by the app owner. Production use
  requires submitting the app for review.
- **Authenticated joins** use the `zoom` object on the REST `create_bot` request, with
  exactly one of `zak_url` (the bot joins signed in as a Zoom user) or `obf_url` (the bot
  joins on behalf of a user who is already in the meeting). Each is an HTTPS URL on
  **your** server that returns a fresh token when MeetStream calls it at join time.
  You run the Zoom OAuth flow and keep the refresh tokens; MeetStream does not store them.
- `use_zoom_obf` and `zoom_oauth_connection_user_id` are **rejected** by the API, and the
  old `/zoom/oauth/*` connection endpoints are no longer documented. Do not use them.
- Omit `zoom` (or send `{}`) for a guest join.

Full walkthrough: https://docs.meetstream.ai/guides/app-integrations/zoom-marketplace-app-setup

## Google Meet

The common complaint is the bot appearing as an unverified guest and getting stuck in
the lobby. The fix is a **signed-in bot**: the bot authenticates as a real Google
Workspace user before joining.

Setup is a one-time Workspace configuration, and the `google_meet` fields below go on the
REST `create_bot` request (the MCP tool does not expose them):

1. Configure a SAML SSO profile in Google Workspace Admin.
2. Generate a certificate pair:
   `openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -sha256 -days 3650 -nodes`
3. Register the domain and logins via `/google-login-domains` and `/google-logins`.
4. On `create_bot`, pass:
   ```json
   "google_meet": { "login_required": true, "google_login_domain": "yourcompany.com",
                    "sign_in_email": "bot@yourcompany.com" }
   ```

**Capacity:** Google limits concurrent meetings per account. Rule of thumb -
`logins = peak concurrent Google Meet sessions / 20`. At 100 concurrent, create 5
logins; MeetStream distributes bots across them round-robin.

Guide: https://docs.meetstream.ai/guides/app-integrations/google-signed-in-bots

## Microsoft Teams

Teams admission depends on tenant lobby policy, which the meeting organiser's admin
controls. There is no API-side override - if a tenant sends external participants to
the lobby, the bot goes to the lobby.

Tune `waiting_room_timeout` accordingly and handle `NotAllowed` gracefully rather than
retrying in a loop. Do not promise a customer that a Teams bot will always self-admit;
that is their admin's decision.

## Before blaming the platform

Check these first - they look like platform problems and are not:

- Meeting link is expired, or is a calendar invite URL rather than a join URL.
- `in_call_recording_timeout` set below 600, so `create_bot` returned 400 and no bot
  was ever created.
- The bot did join, but a streaming-only provider means there is no post-call
  transcript to find.

## Reference

- Platform guides: [Zoom](https://docs.meetstream.ai/guides/platforms/zoom) · [Google Meet](https://docs.meetstream.ai/guides/platforms/google-meet) · [Microsoft Teams](https://docs.meetstream.ai/guides/platforms/microsoft-teams)
- [Google Meet lobby admission](https://docs.meetstream.ai/guides/app-integrations/gmeet-lobby-admission)
- [Zoom authenticated bots (ZAK and OBF)](https://docs.meetstream.ai/guides/app-integrations/zoom-authenticated-bots) · [Zoom app production submission](https://docs.meetstream.ai/guides/app-integrations/zoom-app-production-submission)
- [Automatic leave configuration](https://docs.meetstream.ai/guides/features/automatic-leave-configuration)
