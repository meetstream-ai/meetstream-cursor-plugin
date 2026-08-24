---
name: recordings-and-media
description: >
  Get media out of a MeetStream meeting - audio, video, per-participant
  streams, screenshots - and control recording and retention. Use when the
  user says "download the recording", "get the audio", "per-participant
  audio", "separate speaker tracks", "screenshots", "pause recording",
  "how long are recordings kept", "delete the recording", or "store
  recordings in my own S3 bucket".
---

# Recordings and media

## Getting files

**`get_media_urls`** (MCP tool) returns presigned URLs for audio, video,
per-participant streams and screenshots. That is the one call for most needs.

Media is only ready after the matching webhook fires: `audio.processed` for audio,
`video.processed` for video. Asking earlier returns "not ready" - wait, do not retry
in a tight loop.

## Per-participant streams

Separate tracks per speaker are far more useful than a mixed recording for diarization,
per-speaker analysis, or re-mixing. Ask for them **at bot creation**:

- `audio_separate_streams: true` - one audio track per participant
- `video_separate_streams: true` - one video track per participant

You cannot add this after the fact; the bot has to be told before it records.

These endpoints return **HTTP 202 while the bot is still in the meeting**. That is
"not finished yet", not an error. Poll with a cap.

## Video

Video is off by default. Set `video_required: true` on `create_bot`. It costs more and
takes longer to process, so do not enable it reflexively - ask whether they actually
need pictures.

## Pausing mid-meeting

```
POST /api/v1/bots/{bot_id}/pause_recording     (empty body)
POST /api/v1/bots/{bot_id}/resume_recording    (empty body)
```

REST only, not MCP tools. Use these for privacy windows - a break, an off-record
discussion, someone reading out a credential. The bot stays in the meeting.

## Retention

Set on `create_bot`:

```json
"recording_config": { "retention": { "type": "timed", "hours": 72 } }
```

Omit it and media is retained per the account default, with storage billed by volume.
For anything containing customer conversations, set an explicit window - it is the
cheapest compliance win available.

## Deleting

**`delete_bot_data`** (MCP tool) permanently erases a bot's audio, video and
transcripts and fires a `data_deletion` webhook. It is irreversible. Always confirm
with the user first, and never call it to "clean up" without being asked.

`remove_bot` is different: it makes the bot leave the meeting and **keeps** the data.

## Bring your own bucket

MeetStream can write media straight into your own S3 (or S3-compatible) bucket instead
of its storage:

```
PUT    /api/v1/admin/configs?config_type=storage
GET    /api/v1/admin/configs
DELETE /api/v1/admin/configs?key_name=aws
```

The body needs `provider`, `bucket_name`, `region`, `access_key_id`, `secret_key`, plus
optional `access_mode`, `prefix`/`prefixes` and `endpoint_url` (for R2/MinIO).

Two things to tell the user before they run it:

- This **stores cloud credentials on your MeetStream account**. Use a dedicated IAM user
  scoped to that one bucket, never a root or broad-permission key.
- With `access_mode: write_only`, MeetStream writes to your bucket but its own fetch
  endpoints return 403 for that media - you read it from your bucket, not from the API.

Objects land under `{prefix}/{bot_id}_<file>`.
