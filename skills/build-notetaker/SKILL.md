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
- Errors return `{ "message": "..." }`. Always surface that message. A few endpoints
  (in-meeting commands, the credit check, the prepaid 429) still send `error`, and older
  bot data responses were a bare string: read `message`, then `error`, then the raw body.

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
- **`video_required` defaults to `false`.** Still send `"video_required": false` on every
  bot unless the user explicitly asked to record video, so the intent is explicit.
  `audio_required` (default `true`) is a real field; turning off both audio and video with
  nothing else to do (no live stream, `realtime_endpoints`, `socket_connection_url`,
  `bot_message` or `agent_config_id`) returns 400.
- **When video is on, send `recording_config.video_layout: "speaker_view"`.** The API
  defaults to `"grid_view"`. Both work on Google Meet, Teams and Zoom; Google Meet and
  Teams also take `"speaker_view_default"` / `"grid_view_default"` (Zoom: 400). The field
  is ignored when video is off, and WhatsApp is grid only.
- **Zoom needs no setup.** Do not ask for a Zoom app. `zoom.use_own_app` (`true` /
  `false` / omitted) only matters when the user has their own Zoom app.
- **`get_speaker_timeline` has two shapes.** Large timelines return
  `{bot_id, speaker_timeline_url, size_bytes, truncated: false}`; download the URL (no API
  key, valid 1 hour). Always check for `speaker_timeline_url`.
- **URLs you hand MeetStream must be public.** `callback_url`, `bot_image_url` (https
  PNG/JPEG/GIF/WebP, else 400), image URLs and camera signaling hosts must resolve to
  public addresses, or delivery/download is refused with no 400.
- **`recording_permission_denied_timeout`** (Zoom) is 60-600 seconds, default 600.
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
returns `200`, and the same key with a different meeting returns `409`. While the first
request with a key is still saving, a concurrent one can get `409` with `Retry-After: 2`;
retry it with the same key.

**Prepaid rate limit:** prepaid accounts can create 5 bots per rolling minute and 50 per
rolling hour (bots with `join_at` 10+ minutes ahead are not counted). Over it: `429`,
header `Retry-After: <seconds>`, body
`{"error": "rate_limit_exceeded", "message", "limit", "window", "retry_after"}`, and no
bot is created.

```python
import os, time, uuid, requests

BASE = "https://api.meetstream.ai/api/v1"
HEADERS = {"Authorization": f"Token {os.environ['MEETSTREAM_API_KEY']}"}

def error_text(resp):
    try:
        body = resp.json()
    except ValueError:
        return resp.text
    return (body.get("message") or body.get("error") or str(body)) if isinstance(body, dict) else str(body)

def create_bot(body, key=None, attempts=5):
    headers = {**HEADERS, "Idempotency-Key": key or str(uuid.uuid4())}  # one key for every attempt
    for i in range(attempts):
        resp = requests.post(f"{BASE}/bots/create_bot", headers=headers, json=body, timeout=30)
        if resp.status_code in (201, 507):  # 507 = idempotent replay, success
            return resp.json()
        wait = int(resp.headers.get("Retry-After", "0") or 0)
        if resp.status_code in (429, 409) and wait and i < attempts - 1:
            time.sleep(wait)
            continue
        raise RuntimeError(f"create_bot {resp.status_code}: {error_text(resp)}")
```

```typescript
const BASE = 'https://api.meetstream.ai/api/v1'
const HEADERS = { Authorization: `Token ${process.env.MEETSTREAM_API_KEY}`, 'Content-Type': 'application/json' }

async function errorText(res: Response): Promise<string> {
  const raw = await res.text()
  try {
    const body = JSON.parse(raw)
    return typeof body === 'object' && body ? (body.message ?? body.error ?? raw) : String(body)
  } catch { return raw }
}

async function createBot(body: object, key: string = crypto.randomUUID(), attempts = 5) {
  for (let i = 0; i < attempts; i++) {
    const res = await fetch(`${BASE}/bots/create_bot`, {
      method: 'POST', headers: { ...HEADERS, 'Idempotency-Key': key }, body: JSON.stringify(body)
    })
    if (res.status === 201 || res.status === 507) return res.json()  // 507 = replay, success
    const wait = Number(res.headers.get('retry-after') ?? 0)
    if ((res.status === 429 || res.status === 409) && wait > 0 && i < attempts - 1) {
      await new Promise(r => setTimeout(r, wait * 1000))
      continue
    }
    throw new Error(`create_bot ${res.status}: ${await errorText(res)}`)
  }
}
```

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
