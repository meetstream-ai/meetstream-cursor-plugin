<div align="center">

# MeetStream for Cursor

**Put an AI bot in your meetings from inside your editor.**

Join, record, transcribe and summarize calls on **Zoom**, **Google Meet** and **Microsoft Teams** — or deploy a [**MIA voice agent**](#-mia-voice-agents-that-actually-talk) that listens and *talks back* in the room. All through **19 tools** and **10 skills**, over the hosted [MeetStream](https://meetstream.ai) MCP server.

[![Docs](https://img.shields.io/badge/docs-docs.meetstream.ai-fd6316?style=flat-square)](https://docs.meetstream.ai)
[![MCP](https://img.shields.io/badge/MCP-19%20tools-6b4ea8?style=flat-square)](https://docs.meetstream.ai/build-with-ai/meetstream-mcp-server)
[![Skills](https://img.shields.io/badge/skills-10-45689f?style=flat-square)](#-skills)
[![Platforms](https://img.shields.io/badge/Zoom%20%C2%B7%20Meet%20%C2%B7%20Teams-supported-2c8a61?style=flat-square)](https://docs.meetstream.ai/guides/introduction/how-bots-work)
[![License](https://img.shields.io/badge/license-MIT-867c72?style=flat-square)](LICENSE)

[Install](#-install) · [Try it](#-what-you-can-ask) · [MIA](#-mia-voice-agents-that-actually-talk) · [Tools](#-the-19-tools) · [Skills](#-skills) · [Security](#-your-api-key) · [Docs](https://docs.meetstream.ai)

</div>

---

Nothing to run locally. No server to host. The MCP server is hosted at `https://mcp.meetstream.ai/mcp`, so once the plugin is installed **every agent on your account gets the tools**.

## 🚀 Install

| | Step |
|:--:|---|
| **1** | Search **"MeetStream"** in the Cursor marketplace and click **Add to Cursor** |
| **2** | Grab a key from [app.meetstream.ai/api-key](https://app.meetstream.ai/api-key) |
| **3** | Paste it under **Plugins → Configure → `MEETSTREAM_API_KEY`** |

That's it. Ask your agent to *"list my MeetStream bots"* to confirm it's wired up.

> [!TIP]
> New to MeetStream? [Create your first bot](https://docs.meetstream.ai/guides/get-started/create-your-first-bot) walks the whole flow in a few minutes, and [How bots work](https://docs.meetstream.ai/guides/introduction/how-bots-work) explains what's happening underneath.

## 💬 What you can ask

Plain language. The right skill loads itself based on what you say.

<table>
<tr><td width="50%" valign="top">

**Record and understand a call**
> "Send a MeetStream bot to https://meet.google.com/abc-defg-hij and record it."

> "Summarize that meeting and pull out action items with owners."

> "Who talked the most on the last call?"

> "Download the recording and give me each speaker's audio separately."

</td><td width="50%" valign="top">

**Build and automate**
> "Build me an AI notetaker in this repo that emails the summary to attendees."

> "Auto-join every meeting on my calendar and post transcripts to Slack."

> "Stream live captions to my app while the call is happening."

> "Set up a voice agent that answers questions when someone says 'hey acme'."

</td></tr>
</table>

## 🎙 MIA: voice agents that actually talk

Most meeting bots sit silently and record. **MIA** joins as a real participant that listens, answers, and can call tools mid-conversation. MeetStream runs the audio bridge, so **you host nothing**.

```jsonc
// 1. POST /api/v1/mia  ->  returns agent_config_id
{
  "agent_name": "Meeting Assistant",
  "mode": "pipeline",                          // or "realtime" for speech-to-speech
  "model":       { "provider": "openai", "model": "gpt-4.1", "first_message": "Hi, I'm an AI assistant on this call." },
  "voice":       { "provider": "openai", "voice_id": "nova" },
  "transcriber": { "provider": "deepgram", "model": "nova-3", "boostwords": ["Acme", "MeetStream"] },
  "agent":       { "tools": [], "mcp_servers": [], "enable_interruptions": true },
  "wake_word":   { "enabled": true, "words": ["hey acme"], "timeout": 30 }
}

// 2. create_bot  ->  attach it with ONE field
{ "meeting_link": "https://...", "bot_name": "Assistant", "agent_config_id": "<id>" }
```

| Choose | When |
|---|---|
| [`pipeline`](https://docs.meetstream.ai/guides/mia/mia-configurations) | You want to pick each provider, use a **wake word**, or tune interruptions and VAD |
| [`realtime`](https://docs.meetstream.ai/guides/mia/create-mia) | You want lowest latency with a single speech-to-speech model |

> [!IMPORTANT]
> Attaching an agent takes **only `agent_config_id`**. Passing `socket_connection_url` or `live_audio_required` alongside it is the single most common cause of a silent agent — those are for bring-your-own-bridge setups. The [`mia-voice-agents`](skills/mia-voice-agents/SKILL.md) skill enforces this.

Two fields worth setting that most people miss: **`boostwords`** on the transcriber (fixes "it mishears our company name") and **`mcp_servers`** on the agent (turns a talking bot into one that does work). Full reference: [Create MIA](https://docs.meetstream.ai/guides/mia/create-mia) · [MIA configurations](https://docs.meetstream.ai/guides/mia/mia-configurations) · [API](https://docs.meetstream.ai/api-reference/api-endpoints/mia/create-agent-config).

## 🧰 The 19 tools

<details open>
<summary><b>Bot lifecycle</b> — 7 tools</summary>

| Tool | Does | API |
|---|---|---|
| `create_bot` | Send a bot into a meeting | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/create-bot) |
| `list_bots` | List your bots | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/list-bots) |
| `get_bot_status` | Where it is right now | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-bot-status) |
| `get_bot_detail` | Full record, incl. `transcript_id` | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-bot-details) |
| `get_bot_summary` | AI summary of the meeting | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-bot-summary) |
| `remove_bot` | Make it leave, **keep** the data | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/remove-bot) |
| `delete_bot_data` | Permanently erase media + transcripts | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/delete-bot-data) |

</details>

<details open>
<summary><b>Transcription</b> — 3 tools</summary>

| Tool | Does | API |
|---|---|---|
| `get_transcript` | Fetch by `transcript_id` | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/transcription/get-transcription) |
| `list_transcriptions` | All transcripts for a bot | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/transcription/get-bot-transcriptions) |
| `transcribe_audio` | Re-transcribe stored audio | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/transcription/transcribe-bot-audio) |

Provider choice matters: [post-call](https://docs.meetstream.ai/guides/transcription-recordings/post-call-transcription) · [live](https://docs.meetstream.ai/guides/transcription-recordings/live-transcription) · [all providers](https://docs.meetstream.ai/guides/transcription-recordings/providers/transcription-providers) · [diarization](https://docs.meetstream.ai/guides/transcription-recordings/diarization) · [languages](https://docs.meetstream.ai/guides/transcription-recordings/languages-and-translation)

</details>

<details open>
<summary><b>Meeting data</b> — 4 tools</summary>

| Tool | Does | API |
|---|---|---|
| `get_media_urls` | Presigned audio, video, per-participant, screenshots | [ref](https://docs.meetstream.ai/guides/transcription-recordings/retrieve-recordings) |
| `get_participants` | Who was in the room | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/fetch-participants) |
| `get_chats` | In-meeting chat | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-bot-chats) |
| `get_speaker_timeline` | Who spoke when | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-speaker-timeline) |

</details>

<details open>
<summary><b>Live interaction &amp; calendar</b> — 5 tools</summary>

| Tool | Does | API |
|---|---|---|
| `send_chat_message` | Post into the meeting chat | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/send-message) |
| `send_image` | Show an image as the bot's video | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/send-image) |
| `list_calendar_events` | Upcoming synced events | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/fetch-sync-events) |
| `schedule_calendar_bot` | Put a bot on an event | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/schedule-event) |
| `webhook_events_guide` | The event lifecycle, inline | [ref](https://docs.meetstream.ai/guides/webhooks/webhooks-and-events) |

</details>

## 📚 Skills

Ten skills teach the agent how to use MeetStream *well*. They load automatically when the conversation matches — no command to remember.

| Skill | Loads when you say | Deep dive |
|---|---|---|
| [**join-meeting**](skills/join-meeting/SKILL.md) | "join this meeting", "send a bot to…", "record this call" | [First bot](https://docs.meetstream.ai/guides/get-started/create-your-first-bot) |
| [**meeting-brief**](skills/meeting-brief/SKILL.md) | "summarize that meeting", "action items", "who talked the most" | [Speaker timeline](https://docs.meetstream.ai/guides/features/participants-and-speaker-timeline) |
| [**mia-voice-agents**](skills/mia-voice-agents/SKILL.md) | "voice agent", "talking bot", "wake word", "MIA" | [Create MIA](https://docs.meetstream.ai/guides/mia/create-mia) |
| [**webhooks**](skills/webhooks/SKILL.md) | "webhook", "callback_url", "my webhook isn't firing" | [Events](https://docs.meetstream.ai/guides/webhooks/webhooks-and-events) · [Signatures](https://docs.meetstream.ai/guides/webhooks/webhook-signature-verification) |
| [**realtime-streaming**](skills/realtime-streaming/SKILL.md) | "live captions", "stream the audio", "websocket" | [Live audio](https://docs.meetstream.ai/guides/websockets/real-time-audio-streaming) · [Control patterns](https://docs.meetstream.ai/guides/websockets/meeting-control-patterns) |
| [**recordings-and-media**](skills/recordings-and-media/SKILL.md) | "download the recording", "per-participant audio", "retention" | [Retrieve](https://docs.meetstream.ai/guides/transcription-recordings/retrieve-recordings) · [Per-participant](https://docs.meetstream.ai/guides/transcription-recordings/per-participant-audio) |
| [**calendar-automation**](skills/calendar-automation/SKILL.md) | "auto-join my meetings", "connect my calendar", "recurring" | [Google](https://docs.meetstream.ai/guides/calendar-integrations/google-calendar-oauth-setup) · [Outlook](https://docs.meetstream.ai/guides/calendar-integrations/outlook-calendar-setup) |
| [**platform-setup**](skills/platform-setup/SKILL.md) | "can't join Zoom", "waiting room", "signed-in bots", "Teams lobby" | [Zoom](https://docs.meetstream.ai/guides/platforms/zoom) · [Meet](https://docs.meetstream.ai/guides/platforms/google-meet) · [Teams](https://docs.meetstream.ai/guides/platforms/microsoft-teams) |
| [**troubleshooting**](skills/troubleshooting/SKILL.md) | "401", "403", "stuck on 202", "transcript never arrives" | [Errors](https://docs.meetstream.ai/errors) · [Debugging](https://docs.meetstream.ai/guides/help/debugging-bots) |
| [**build-notetaker**](skills/build-notetaker/SKILL.md) | "build a notetaker", "integrate MeetStream into my app" | [Agent skills](https://docs.meetstream.ai/build-with-ai/agent-skills) |

<details>
<summary><b>Why the skills matter more than the tools</b></summary>

The tools are just an API surface. The skills carry the hard-won details that stop an agent guessing wrong:

- **MIA takes only `agent_config_id`** — adding bridge URLs silences the agent
- The webhook envelope key is **`event`**, and **`bot.stopped`** is the single terminal event, always at `status_code: 200`
- **Streaming-only providers never emit `bot.done`** and return `202` forever on a post-call transcript fetch
- Transcripts are fetched by **`transcript_id`**, and segments use **`transcript`**, not `text`
- **`202` and `507` are not errors** — 202 means poll again, 507 means your idempotent retry replayed
- **REST uses `Authorization: Token`, the MCP server uses `Bearer`** — mixing them up returns 401
- `in_call_recording_timeout` has a hard **600 second floor**

</details>

> [!NOTE]
> Some capabilities are **REST-only** and deliberately not MCP tools: [MIA agent configs](https://docs.meetstream.ai/api-reference/api-endpoints/mia/create-agent-config), [calendar connection](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/create-calendar), [Google signed-in bots](https://docs.meetstream.ai/guides/app-integrations/google-signed-in-bots), [Zoom OAuth/OBF](https://docs.meetstream.ai/guides/app-integrations/zoom-obf-implementation), [custom storage](https://docs.meetstream.ai/guides/features/custom-storage-configurations) and [pause/resume](https://docs.meetstream.ai/guides/features/pause-resume-recording). The skills know the difference and route to REST instead of inventing a tool that doesn't exist.

## 🔐 Your API key

Declared as a **plugin variable** and set in the Cursor dashboard — never in this repo, never in a committed file. Cursor interpolates it into the `Authorization` header per request. The hosted server is multi-tenant and stores no key of its own.

Prefer the key never leaving your machine? Run the server locally over stdio instead:

```bash
npx -y @meetstream/mcp     # reads MEETSTREAM_API_KEY from your environment
```

Auth reference: [Authentication](https://docs.meetstream.ai/api-reference/authentication) · [MCP server setup](https://docs.meetstream.ai/build-with-ai/meetstream-mcp-server)

## 🛠 Local development

```bash
git clone https://github.com/meetstream-ai/meetstream-cursor-plugin.git
ln -s "$PWD/meetstream-cursor-plugin" ~/.cursor/plugins/local/meetstream
```

Restart Cursor. The symlink means edits apply on the next restart with no re-copying.

```
.cursor-plugin/plugin.json   manifest + MEETSTREAM_API_KEY variable
mcp.json                     Cursor connector  -> mcp.meetstream.ai
.mcp.json                    Claude Code / agent-plugin connector
skills/*/SKILL.md            10 skills
```

## 🌐 The rest of the ecosystem

| | What | Where |
|:--:|---|---|
| 🔌 | **MCP server** — the same 19 tools for any MCP client | [`@meetstream/mcp`](https://www.npmjs.com/package/@meetstream/mcp) · [docs](https://docs.meetstream.ai/build-with-ai/meetstream-mcp-server) |
| ⌨️ | **CLI** — drive MeetStream from your terminal | [`@meetstream/cli`](https://www.npmjs.com/package/@meetstream/cli) · [docs](https://docs.meetstream.ai/build-with-ai/meetstream-cli) |
| 🤖 | **Claude Code plugin** — `/plugin marketplace add meetstream-ai/claude-plugin` | [docs](https://docs.meetstream.ai/build-with-ai/claude-integration) |
| 🧪 | **Labs** — runnable end-to-end templates | [github](https://github.com/meetstream-ai/labs) |
| 📖 | **Docs for agents** — machine-readable docs for your own tooling | [docs](https://docs.meetstream.ai/build-with-ai/docs-for-agents) |

## 🔗 Links

**Start here** · [Documentation](https://docs.meetstream.ai) · [API reference](https://docs.meetstream.ai/api-reference/introduction) · [API playground](https://docs.meetstream.ai/guides/get-started/api-playground) · [Dashboard setup](https://docs.meetstream.ai/guides/get-started/dashboard-setup)

**Go deeper** · [How bots work](https://docs.meetstream.ai/guides/introduction/how-bots-work) · [Webhooks & events](https://docs.meetstream.ai/guides/webhooks/webhooks-and-events) · [Scheduling bots](https://docs.meetstream.ai/guides/features/scheduling-bots) · [Idempotency & dedup](https://docs.meetstream.ai/guides/features/deduplication-idempotency-keys) · [Usage & retention](https://docs.meetstream.ai/guides/features/usage-and-retention) · [Automatic leave](https://docs.meetstream.ai/guides/features/automatic-leave-configuration)

**When stuck** · [Errors](https://docs.meetstream.ai/errors) · [Debugging bots](https://docs.meetstream.ai/guides/help/debugging-bots) · [FAQ](https://docs.meetstream.ai/guides/help/faq) · [Support](https://docs.meetstream.ai/guides/help/support) · [support@meetstream.ai](mailto:support@meetstream.ai)

<div align="center">

---

Built by [MeetStream.ai](https://meetstream.ai) · [MIT](LICENSE)

</div>
