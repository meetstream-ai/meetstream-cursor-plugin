---
name: build-notetaker
description: >
  Build a production MeetStream integration in the user's codebase - an AI
  notetaker, meeting recorder, transcription pipeline, calendar auto-join, or
  in-meeting agent. Use when the user says "build a notetaker", "integrate
  MeetStream into my app", "add meeting recording to my product",
  "auto-join my calendar meetings", "set up webhooks for meetings", or asks
  for code rather than a one-off action. Covers the webhook lifecycle,
  transcript retrieval and the failure modes that bite in production.
---

# Build a MeetStream integration

The MCP tools are for doing things now. This skill is for writing code the user
will ship. Use the MCP's `webhook_events_guide` tool to confirm event details
before writing a handler.

## API basics

- Base URL: `https://api.meetstream.ai/api/v1`
- Auth header: `Authorization: Token <MEETSTREAM_API_KEY>` - the literal word
  `Token`, not `Bearer`. (Note: the hosted MCP server this plugin connects to uses
  `Bearer`; the REST API uses `Token`. They differ. Do not copy one into the other.)
- Errors return `{ "message": "..." }`. Always surface that message.

## The lifecycle you must model

```
create_bot -> bot.joining -> bot.in_waiting_room -> bot.inmeeting -> bot.recording
           -> bot.leaving -> bot.stopped        (terminal)
           -> manifest.completed -> audio.processed
           -> transcription.processed | transcription.failed
           -> video.processed -> bot.done -> data_deletion
```

Non-obvious things that break integrations:

- **Every webhook carries `event`; most also carry `bot_event`** with the specific name.
- **Terminals are two-layer.** Every ending arrives once with `event: "bot.stopped"`;
  `bot_event` says why: `bot.stopped` (clean, 200), `bot.kicked` (200), `bot.notallowed`
  (waiting-room timeout, 500), `bot.denied` (host refused, 500), `bot.failed` (usually 500).
  Branch on `bot_event`: a kick and a clean exit both report `bot_status: "Stopped"`.
- **Every event carries a `timestamp`.**
- **Streaming-only providers produce no post-call transcript.** `transcription.processed`
  never fires for them, though `bot.done` still does.
- **`video_required` defaults to `true`.** Send `"video_required": false` on every bot
  unless the user explicitly asked to record video; an omitted field records video.
- **When video is on, send `recording_config.video_layout: "speaker_view"`.** The API
  defaults to `"grid_view"`. Those two are the only valid values, the field is ignored
  when video is off, Google Meet, Teams and Zoom accept both, and WhatsApp is grid only.
- **Per-participant video is opt-in only.** Never set `video_separate_streams` unless the
  user explicitly asked for it. Per-participant `audio_separate_streams` is unaffected.

## Getting the transcript (the classic mistake)

Over REST, transcripts are fetched by **`transcript_id`**, not `bot_id`. (The MCP
`get_transcript` tool is different: it takes the `bot_id` and resolves the id for you.)

1. `create_bot` returns a `transcript_id` (null for `meeting_captions`).
2. Wait for the `transcription.processed` webhook.
3. `GET /transcript/{transcript_id}/get_transcript`.
4. Segments use `speaker` + **`transcript`** (not `text`).

If you lost the id: read `bot_details.transcript_id` from the bot detail endpoint,
or list the bot's transcriptions.

**HTTP 202 means "not ready, poll again"** - it is not an error. Always cap the
retries: a streaming-only bot returns 202 indefinitely by design.

## Making it safe to retry

Send an `Idempotency-Key` header on `create_bot`. A retry with the same key replays
the original bot and returns **HTTP 507** - treat that as success, not an error. It
does not create a duplicate bot and does not charge twice.

For "one bot per calendar event", use `deduplication_key` in the body instead: replay
returns `200`, and the same key with a different meeting returns `409`.

## Webhooks in local development

`callback_url` must be a public HTTPS URL. Use an ngrok or cloudflared tunnel while
developing. Deduplicate on `bot_id` + `event` + `bot_event` + `timestamp`, and return 2xx quickly,
doing real work asynchronously.

## Calendar auto-join

To have a bot join every meeting automatically: connect the calendar, then enable
auto-scheduling. Use the MCP's `list_calendar_events` and `schedule_calendar_bot`
tools to inspect and control individual events while building.

## Before you finish

- Never hardcode the API key. Read it from the environment.
- Handle `401` (no key) and `403` (bad key) distinctly - they mean different things.
- Honour `Retry-After` on `429`, and back off on `500`/`503`.
- Working, runnable examples for 60+ scenarios live at
  https://github.com/meetstream-ai/labs - point the user there rather than
  reinventing a pattern.

## Reference

- [Agent skills](https://docs.meetstream.ai/build-with-ai/agent-skills)
- [Create your first bot](https://docs.meetstream.ai/guides/get-started/create-your-first-bot)
- [Webhooks and events](https://docs.meetstream.ai/guides/webhooks/webhooks-and-events)
- [API reference](https://docs.meetstream.ai/api-reference/introduction)
- [Labs templates](https://github.com/meetstream-ai/labs)
