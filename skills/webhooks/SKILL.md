---
name: webhooks
description: >
  Handle MeetStream webhooks correctly - the full event lifecycle, signature
  verification, redelivery, and local tunnels. Use when the user says
  "webhook", "callback_url", "bot events", "how do I know when the meeting
  ended", "my webhook isn't firing", "verify the signature", "ngrok", or is
  writing any handler that reacts to bot state. Read this before writing a
  webhook handler.
---

# MeetStream webhooks

Call the **`webhook_events_guide`** MCP tool for the live-verified event reference
before writing a handler. This skill covers what that reference does not: the traps.

## The envelope

```json
{ "event": "bot.inmeeting", "bot_id": "...", "bot_status": "InMeeting",
  "message": "...", "status_code": 200, "custom_attributes": {} }
```

The key is **`event`**. Any documentation, blog post or older sample that says
`bot_event` is wrong. If you branch on `bot_event` your handler silently does nothing.

There is no global webhook endpoint. You set `callback_url` **per bot** on `create_bot`.

## The lifecycle

```
bot.joining -> bot.in_waiting_room -> bot.inmeeting -> bot.recording
            -> bot.leaving -> bot.stopped          (terminal)
            -> manifest.completed -> audio.processed
            -> transcription.processed | transcription.failed
            -> video.processed -> bot.done -> data_deletion
```

Five things that break handlers:

1. **`bot.stopped` is the only terminal event.** There is no `bot.kicked`,
   `bot.denied`, `bot.notallowed` or `bot.failed`. The reason is in `bot_status`:
   `Stopped` (normal) | `NotAllowed` (waiting-room timeout) | `Denied` (host refused)
   | `Error` (crash).
2. **`bot.stopped` always has `status_code: 200`**, even when the bot never got in.
   Do not treat 200 as "it worked" - read `bot_status`.
   `500` appears only on `transcription.failed` and a failed `bot.done`.
3. **`bot.error` is NOT terminal.** It means a streaming provider hiccuped upstream;
   the bot is still in the meeting. Do not tear down state on it.
4. **Streaming-only providers end at `audio.processed`** and never emit `bot.done`.
   Waiting for `bot.done` on those hangs forever.
5. **`transcript_id` is not in the webhook.** Get it from the `create_bot` response,
   `get_bot_detail`, or `list_transcriptions`.

## Writing the handler

- **Return 200 fast.** Acknowledge, then do the real work asynchronously. Slow handlers
  cause retries and duplicate processing.
- **Expect redelivery.** Dedupe on `bot_id` + `event`. `bot.error` can legitimately
  repeat, so include a hash of `message` in its dedupe key.
- **Capture the raw body before JSON parsing** if you plan to verify signatures -
  re-serializing the parsed object will not match the signature.
- **Do not assume ordering.** Treat lifecycle states as a forward-only ladder and
  ignore anything that moves backwards.

## Signature verification

Per-bot `callback_url` deliveries are **unsigned**; workspace-level endpoints are
signed with an HMAC signature. Verify when you are on a workspace endpoint, and when
you are not, protect the route another way: a secret path segment, an allowlist, or a
shared token in the query string. Never treat an unauthenticated public webhook route
as trusted input.

## Local development

`callback_url` must be a **public HTTPS URL**. Localhost will never receive anything.

```bash
ngrok http 3000          # then use the https URL + your path
cloudflared tunnel --url http://localhost:3000
```

Two failure modes to check before debugging your code:
- The tunnel returns an interstitial page and MeetStream sees a 200 that never reached
  your handler. Verify with a request that echoes a value your handler generated.
- The URL is `http`, not `https`.

## Quick triage

| Symptom | Likely cause |
|---|---|
| No events at all | `callback_url` not public HTTPS, or not set on that bot |
| Events stop after `audio.processed` | Streaming-only provider - there is no `bot.done` |
| Handler never runs but delivery succeeds | Branching on `bot_event` instead of `event` |
| Everything looks successful but no recording | `bot.stopped` with `bot_status` `NotAllowed` or `Denied` |
| Duplicate processing | No dedupe on `bot_id` + `event`, or handler too slow |
