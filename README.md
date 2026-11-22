<div align="center">

# MeetStream for Cursor

**Put an AI bot in your meetings from inside your editor.**

Join, record, transcribe and summarize calls on **Zoom**, **Google Meet** and **Microsoft Teams** - or put an agent in the room that listens and *talks back*: on [your own stack](#-agents-that-actually-talk) (any models, local too) or hosted by MeetStream (MIA). All through the hosted MCP tools and the same skills as the Claude Code plugin, over the hosted [MeetStream](https://meetstream.ai) MCP server.

[![Docs](https://img.shields.io/badge/docs-docs.meetstream.ai-fd6316?style=flat-square)](https://docs.meetstream.ai)
[![Platforms](https://img.shields.io/badge/Zoom%20%C2%B7%20Meet%20%C2%B7%20Teams-supported-2c8a61?style=flat-square)](https://docs.meetstream.ai/guides/introduction/how-bots-work)
[![License](https://img.shields.io/badge/license-MIT-867c72?style=flat-square)](LICENSE)

[Install](#-install) · [Try it](#-what-you-can-ask) · [Agents](#-agents-that-actually-talk) · [Tools](#-the-tools) · [Skills](#-skills) · [Security](#-sign-in-and-keys) · [Docs](https://docs.meetstream.ai)

</div>

---

## Three flows

1. **Recording only (Bot API, default).** A bot joins Zoom, Google Meet or Microsoft Teams. You get recordings, transcripts, participants, chat and webhooks after the call, or live data during the call. Use this for every request that does not ask for an agent in the meeting.
2. **Agents in meetings on your own stack (recommended first when the user asks for an agent, a voice agent or a talking bot).**
   - 2a. **Bridge server.** The bot opens two WebSockets to your server: live audio in (`live_audio_required.websocket_url`) and the control socket (`socket_connection_url.websocket_url`) to send audio, chat, images and interrupts back. Run any speech-to-text, LLM and TTS, local or hosted (for example Parakeet for local speech-to-text). Price: the bot price only, no agent infrastructure and no model tokens. [Bridge server docs](https://docs.meetstream.ai/guides/websockets/bridge-server-architecture), starter: [meetstream-ai/meetstream-agent](https://github.com/meetstream-ai/meetstream-agent).
   - 2b. **Your own agent as the brain.** Claude Code, Codex, Hermes, OpenClaw and others through `model.provider: "local_agent"` and the MeetStream Agent Kit. The bot plus $0.25 per hour for the agent infrastructure, no model tokens. Warning: meeting speech reaches an agent that has access to your computer and terminal (prompt injection); keep the computer, the agent and the bridge on for the whole call, and use the lowest access level. [Your own agent](https://docs.meetstream.ai/guides/features/mia-screen-sharing-and-custom-agents).
3. **Managed agent infrastructure (Agents API, API name MIA).** MeetStream hosts the voice agent: MeetStream's inbuilt keys (no provider account; Inbuilt keys work in pipeline mode only: Deepgram (speech-to-text) and OpenAI (LLM and voice). Realtime mode runs on OpenAI and needs your own OpenAI key. Any other provider: bring your own key.) or your own provider keys. The bot plus $0.25 per hour for the agent infrastructure, plus the models you pick ([pricing and keys](https://docs.meetstream.ai/guides/mia/pricing-and-keys)). A hosted agent bot takes only `agent_config_id`, never the bridge sockets.

Overview: [Integration paths](https://docs.meetstream.ai/guides/introduction/integration-paths). The skills give the fastest path first (curl, Node `@meetstream/sdk`, Python `meetstream-sdk`) and end with links to the latest docs; for complete example apps see [Tutorials and Sample Apps](https://docs.meetstream.ai/tutorials/sample-apps).

Nothing to run locally. No server to host. The MCP server is hosted at `https://mcp.meetstream.ai/mcp`, so once the plugin is installed **every agent on your account gets the tools**.

## 🚀 Install

| | Step |
|:--:|---|
| **1** | Search **"MeetStream"** in the Cursor marketplace and click **Add to Cursor** |
| **2** | In **Cursor Settings → MCP**, click **Connect** next to `meetstream` |
| **3** | Sign in at app.meetstream.ai and click **Allow** - no key to copy |

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

> "Build a voice agent on my own models with local Parakeet."

> "Host a voice agent that answers questions when someone says 'hey acme'."

</td></tr>
</table>

## 🎙 Agents that actually talk

Agents come in on request, in this order (the three flows):

| Flow | What you run | Price per meeting hour |
|---|---|---|
| 2a. Bridge server (your own stack, recommended first) | Your server takes the bot's two WebSockets (`live_audio_required.websocket_url` for audio in, `socket_connection_url.websocket_url` for `sendaudio`, `sendmsg`, `interrupt` back) and runs any speech-to-text, LLM and TTS, local or hosted. [Docs](https://docs.meetstream.ai/guides/websockets/bridge-server-architecture), [starter](https://github.com/meetstream-ai/meetstream-agent) | Bot price only |
| 2b. Your own agent | `model.provider: "local_agent"` + the [Agent Kit](https://github.com/meetstream-ai/agent-kit) (Claude Code, Codex, Hermes, OpenClaw and others) | Bot + $0.25 agent infrastructure; no model tokens from MeetStream |
| 3. Hosted Agents API (MIA) | MeetStream hosts the agent: inbuilt keys (`"key_source": "meetstream"`; Inbuilt keys work in pipeline mode only: Deepgram (speech-to-text) and OpenAI (LLM and voice). Realtime mode runs on OpenAI and needs your own OpenAI key. Any other provider: bring your own key.), then your own provider keys | Bot + $0.25 agent infrastructure + the models you pick (the dashboard agent editor shows the rate) |

Flow 2b connects meeting speech to an agent with access to your computer (prompt injection): use `readonly` access and the activation code, and keep your computer, agent and bridge on for the whole call. In flow 3, if inbuilt keys are not on for your account yet, the API says so: use your own keys. The hosted agent (below) joins as a real participant that listens, answers, and can call tools mid-conversation; MeetStream runs its audio bridge, so you host nothing.

```jsonc
// 1. POST /api/v1/mia  ->  returns agent_config_id
{
  "agent_name": "Meeting Assistant",
  "mode": "pipeline",
  "model":       { "provider": "openai", "model": "gpt-4o-mini", "key_source": "meetstream",
                   "system_prompt": "You are a concise meeting assistant." },
  "voice":       { "provider": "openai", "model": "tts-1", "voice_id": "nova", "key_source": "meetstream" },
  "transcriber": { "provider": "deepgram", "model": "nova-3", "key_source": "meetstream", "boostwords": ["Acme", "MeetStream"] },
  "agent":       { "tools": [], "mcp_servers": [], "enable_interruptions": true },
  "wake_word":   { "enabled": true, "words": ["hey acme"], "timeout": 30 }
}

// 2. create_bot  ->  attach it with ONE field
{ "meeting_link": "https://...", "bot_name": "Assistant", "agent_config_id": "<id>" }
```

| Choose | When |
|---|---|
| [`pipeline`](https://docs.meetstream.ai/guides/mia/mia-custom-configurations) | You want to pick each provider, use a **wake word**, or tune interruptions and VAD |
| [`realtime`](https://docs.meetstream.ai/guides/mia/create-an-agent) | You want lowest latency with a single speech-to-speech model |

> [!IMPORTANT]
> Attaching an agent takes **only `agent_config_id`**. Passing `socket_connection_url` or `live_audio_required` alongside it is the single most common cause of a silent agent - those are for the bridge server flow (2a), on a separate bot. The [`mia-voice-agents`](skills/mia-voice-agents/SKILL.md) skill enforces this.

Two fields worth setting that most people miss: **`boostwords`** on the transcriber (fixes "it mishears our company name") and **`mcp_servers`** on the agent (turns a talking bot into one that does work). Full reference: [Create MIA](https://docs.meetstream.ai/guides/mia/create-an-agent) · [MIA configurations](https://docs.meetstream.ai/guides/mia/mia-custom-configurations) · [API](https://docs.meetstream.ai/api-reference/api-endpoints/mia/create-agent-config).

## 🧰 The tools

<details open>
<summary><b>Bot lifecycle</b> - 7 tools</summary>

| Tool | Does | API |
|---|---|---|
| `create_bot` | Send a bot into a meeting | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/create-bot) |
| `list_bots` | List your bots | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/list-bots) |
| `get_bot_status` | Where it is right now | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-bot-status) |
| `get_bot_detail` | Full record, incl. `transcript_id` | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-bot-details) |
| `get_bot_summary` | Says there is no built-in summary; summarize from `get_transcript` | — |
| `remove_bot` | Make it leave, **keep** the data | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/remove-bot) |
| `delete_bot_data` | Permanently erase media + transcripts | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/delete-bot-data) |

</details>

<details open>
<summary><b>Transcription</b> - 3 tools</summary>

| Tool | Does | API |
|---|---|---|
| `get_transcript` | Fetch by `bot_id`; resolves the `transcript_id` for you | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/transcription/get-transcription) |
| `list_transcriptions` | All transcripts for a bot | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/transcription/get-bot-transcriptions) |
| `transcribe_audio` | Re-transcribe stored audio | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/transcription/transcribe-bot-audio) |

Provider choice matters: [post-call](https://docs.meetstream.ai/guides/transcription-recordings/post-call-transcription) · [live](https://docs.meetstream.ai/guides/transcription-recordings/live-transcription) · [all providers](https://docs.meetstream.ai/guides/transcription-recordings/providers/transcription-providers) · [diarization](https://docs.meetstream.ai/guides/transcription-recordings/diarization) · [languages](https://docs.meetstream.ai/guides/transcription-recordings/languages-and-translation)

</details>

<details open>
<summary><b>Meeting data</b> - 4 tools</summary>

| Tool | Does | API |
|---|---|---|
| `get_media_urls` | Presigned audio, video, per-participant, screenshots | [ref](https://docs.meetstream.ai/guides/transcription-recordings/retrieve-recordings) |
| `get_participants` | Who was in the room | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/fetch-participants) |
| `get_chats` | In-meeting chat | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-bot-chats) |
| `get_speaker_timeline` | Who spoke when | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/get-speaker-timeline) |

</details>

<details open>
<summary><b>Live interaction &amp; calendar</b> - 12 tools</summary>

| Tool | Does | API |
|---|---|---|
| `send_chat_message` | Post into the meeting chat | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/send-message) |
| `send_image` | Show an image as the bot's video | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/bot-endpoints/send-image) |
| `take_screenshot` | Capture the meeting (Google Meet, Teams); get it with `get_media_urls` `kind: "screenshots"` | — |
| `raise_hand` / `lower_hand` | Raise or lower the bot's hand | [guide](https://docs.meetstream.ai/guides/features/raise-lower-hand) |
| `pause_recording` / `resume_recording` | Pause capture, then resume on the same timeline | [guide](https://docs.meetstream.ai/guides/features/pause-resume-recording) |
| `list_calendar_events` | Upcoming synced events | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/fetch-sync-events) |
| `schedule_calendar_bot` | Put a bot on an event | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/schedule-event) |
| `list_calendar_connections` | Connected Google and Outlook accounts and their status (no tokens) | — |
| `disconnect_calendar` | Disconnect a calendar (`confirm: true`; also deletes its synced events and scheduled bots by default) | — |
| `webhook_events_guide` | The event lifecycle, inline | [ref](https://docs.meetstream.ai/guides/webhooks/webhooks-and-events) |

</details>

<details open>
<summary><b>Voice agents</b> - 7 tools</summary>

| Tool | Does | API |
|---|---|---|
| `list_agents` | Your agent configs and `mia_managed` (inbuilt keys on, and which providers and models); no prices | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/mia/create-agent-config) |
| `get_agent` | One agent config (keys redacted) | — |
| `create_agent` | Create an agent (`model.system_prompt` required; inbuilt keys in pipeline mode only; never takes a key or the pairing token) | [ref](https://docs.meetstream.ai/api-reference/api-endpoints/mia/create-agent-config) |
| `update_agent` | Change a saved config (a `mode` change needs the full config) | — |
| `delete_agent` | Permanently delete a config (`confirm: true`) | — |
| `check_local_agent` | Test a `local_agent` bridge; returns the status and the fix | [guide](https://docs.meetstream.ai/guides/features/mia-screen-sharing-and-custom-agents) |
| `update_live_agent` | Change a running agent mid-call | — |

</details>

<details open>
<summary><b>Credits</b></summary>

| Tool | Does | API |
|---|---|---|
| `get_balance` | Prepaid credit balance, or the billing link when the balance is not available | — |
| `top_up_link` | Billing page link to add credits (use it on `Low credits please recharge`) | — |

</details>

## 📚 Skills

The skills teach the agent how to use MeetStream *well*. They load automatically when the conversation matches - no command to remember.

| Skill | Loads when you say | Deep dive |
|---|---|---|
| [**join-meeting**](skills/join-meeting/SKILL.md) | "join this meeting", "send a bot to…", "record this call" | [First bot](https://docs.meetstream.ai/guides/get-started/create-your-first-bot) |
| [**meeting-brief**](skills/meeting-brief/SKILL.md) | "summarize that meeting", "action items", "what did we decide last week" (saves notes and remembers them) | [Speaker timeline](https://docs.meetstream.ai/guides/features/participants-and-speaker-timeline) |
| [**mia-voice-agents**](skills/mia-voice-agents/SKILL.md) | "voice agent", "agent in the meeting", "talking bot", "wake word", "make Grok join my call", "standup agent", "AI interviewer", "MIA" | [Pricing and keys](https://docs.meetstream.ai/guides/mia/pricing-and-keys) · [Create MIA](https://docs.meetstream.ai/guides/mia/create-an-agent) |
| [**webhooks**](skills/webhooks/SKILL.md) | "webhook", "callback_url", "my webhook isn't firing" | [Events](https://docs.meetstream.ai/guides/webhooks/webhooks-and-events) · [Signatures](https://docs.meetstream.ai/guides/webhooks/webhook-signature-verification) |
| [**realtime-streaming**](skills/realtime-streaming/SKILL.md) | "live captions", "stream the audio", "websocket" | [Live audio](https://docs.meetstream.ai/guides/websockets/real-time-audio-streaming) · [Control patterns](https://docs.meetstream.ai/guides/websockets/meeting-control-patterns) |
| [**recordings-and-media**](skills/recordings-and-media/SKILL.md) | "download the recording", "per-participant audio", "retention" | [Retrieve](https://docs.meetstream.ai/guides/transcription-recordings/retrieve-recordings) · [Per-participant](https://docs.meetstream.ai/guides/transcription-recordings/per-participant-audio) |
| [**calendar-automation**](skills/calendar-automation/SKILL.md) | "auto-join my meetings", "connect my calendar", "recurring" | [Google](https://docs.meetstream.ai/guides/calendar-integrations/google-calendar-oauth-setup) · [Outlook](https://docs.meetstream.ai/guides/calendar-integrations/outlook-calendar-setup) |
| [**platform-setup**](skills/platform-setup/SKILL.md) | "can't join Zoom", "waiting room", "signed-in bots", "Teams lobby", "WhatsApp calls (Beta)" | [Zoom](https://docs.meetstream.ai/guides/platforms/zoom) · [Meet](https://docs.meetstream.ai/guides/platforms/google-meet) · [Teams](https://docs.meetstream.ai/guides/platforms/microsoft-teams) |
| [**troubleshooting**](skills/troubleshooting/SKILL.md) | "401", "403", "stuck on 202", "transcript never arrives" | [Errors](https://docs.meetstream.ai/errors) · [Debugging](https://docs.meetstream.ai/guides/help/debugging-bots) |
| [**getting-started**](skills/getting-started/SKILL.md) | "how do I start", "I don't have an API key" | [First bot](https://docs.meetstream.ai/guides/get-started/create-your-first-bot) |
| [**meetstream**](skills/meetstream/SKILL.md) | "integrate MeetStream into my app", any API, field or webhook question | [API reference](https://docs.meetstream.ai/api-reference/introduction) |
| [**notetaker**](skills/notetaker/SKILL.md) | "build a notetaker", "email the summary after each call" (fastest path, API guide) | [Post-call transcription](https://docs.meetstream.ai/guides/transcription-recordings/post-call-transcription) |
| [**for-your-customers**](skills/for-your-customers/SKILL.md) | "bots join our customers' calls", "per customer", "multi-tenant" | [Custom attributes](https://docs.meetstream.ai/guides/features/custom-attributes) · [Custom storage](https://docs.meetstream.ai/guides/features/custom-storage-configurations/overview) |
| [**sales-coach**](skills/sales-coach/SKILL.md) | "real-time sales coach", "live call coaching" (fastest path, API guide) | [Live transcription](https://docs.meetstream.ai/guides/transcription-recordings/live-transcription) |
| [**verify-account**](skills/verify-account/SKILL.md) | "check my account", "which providers are configured" | [Authentication](https://docs.meetstream.ai/api-reference/authentication) |
| [**test-bot**](skills/test-bot/SKILL.md) | "send a test bot", "watch the webhooks" | [First bot](https://docs.meetstream.ai/guides/get-started/create-your-first-bot) |
| [**migrate-from-recall**](skills/migrate-from-recall/SKILL.md) | "migrate from Recall", "switch from Recall.ai" | [Docs](https://docs.meetstream.ai) |

<details>
<summary><b>Why the skills matter more than the tools</b></summary>

The tools are just an API surface. The skills carry the hard-won details that stop an agent guessing wrong:

- **MIA takes only `agent_config_id`** - adding bridge URLs silences the agent
- Every ending arrives once as **`event: "bot.stopped"`**, with the reason in **`bot_event`** (`bot.kicked`, `bot.notallowed`, `bot.denied`, `bot.failed`); failures carry `status_code: 500`
- **Streaming-only providers produce no post-call transcript**: no `transcription.processed`, and a transcript fetch returns `202` indefinitely
- `get_transcript` takes the **`bot_id`** (REST takes the `transcript_id`), and segments use **`transcript`**, not `text`
- **`202` and `507` are not errors** - 202 means poll again, 507 means your idempotent retry replayed
- **REST uses `Authorization: Token`, the MCP server uses `Bearer`** - mixing them up returns 401
- `in_call_recording_timeout` has a hard **600 second floor**

</details>

> [!NOTE]
> Some capabilities are **REST or dashboard only** and deliberately not MCP tools: [calendar connection](https://docs.meetstream.ai/api-reference/api-endpoints/calendar/create-calendar) (needs OAuth secrets), signed-in bot account management for [Google](https://docs.meetstream.ai/guides/app-integrations/google-signed-in-bots) and [Teams](https://docs.meetstream.ai/guides/app-integrations/teams-signed-in-bots) (sending a signed-in bot works over MCP via `create_bot`'s `google_login_domain` / `teams_login_domain`), [Zoom app setup](https://docs.meetstream.ai/guides/app-integrations/zoom-authenticated-bots), [custom storage](https://docs.meetstream.ai/guides/features/custom-storage-configurations/amazon-s3), provider keys and the Agent Kit pairing token (saved in the dashboard, never pasted in chat). The full tool list is in the [MCP server README](https://github.com/meetstream-ai/meetstream-mcp). The skills use the tools when the MCP is connected and give the REST call when it is not.

## 🔐 Sign-in and keys

There is no key to paste. The plugin's server entry is just the URL, so Cursor signs you in with OAuth: click **Connect**, sign in at app.meetstream.ai and click **Allow**. The dashboard then creates an API key for this connection, named after Cursor. To disconnect, delete that key on the [API keys page](https://app.meetstream.ai/api-key). The hosted server is multi-tenant and stores no key of its own.

Prefer to manage a key yourself (for example in CI)? Add the server to `~/.cursor/mcp.json` with a header instead, and turn off the plugin's own `meetstream` server so the tools do not appear twice:

```json
{
  "mcpServers": {
    "meetstream": {
      "url": "https://mcp.meetstream.ai/mcp",
      "headers": { "Authorization": "Bearer ${env:MEETSTREAM_API_KEY}" }
    }
  }
}
```

**Cursor Agent CLI (`cursor-agent`):** the CLI cannot finish the OAuth sign-in (the plugin's URL-only entry shows `needsAuth`), so use the key-header entry above in the project's `.cursor/mcp.json`.

**Old Claude plugin shadowing the skills:** Cursor also loads Claude Code plugins and skills from `~/.claude`. An old MeetStream Claude plugin there can replace this plugin's skills; remove or update it.

**OpenCode:** use the [Claude plugin](https://github.com/meetstream-ai/claude-plugin) repo's `setup --host opencode` (project `.opencode/skills` and `opencode.json`).

The MCP server is hosted only: there is nothing to install or run locally.

Auth reference: [Authentication](https://docs.meetstream.ai/api-reference/authentication) · [MCP server setup](https://docs.meetstream.ai/build-with-ai/meetstream-mcp-server)

## 🛠 Local development

```bash
git clone https://github.com/meetstream-ai/meetstream-cursor-plugin.git
ln -s "$PWD/meetstream-cursor-plugin" ~/.cursor/plugins/local/meetstream
```

Restart Cursor. The symlink means edits apply on the next restart with no re-copying.

```
.cursor-plugin/plugin.json   manifest (no variables: sign-in is OAuth)
mcp.json                     Cursor connector  -> mcp.meetstream.ai
.mcp.json                    Claude Code / agent-plugin connector
skills/*/SKILL.md            the same skills as the Claude Code plugin
```

## 🌐 The rest of the ecosystem

| | What | Where |
|:--:|---|---|
| 🔌 | **MCP server** - the same tools for any MCP client, hosted at `mcp.meetstream.ai/mcp` | [github](https://github.com/meetstream-ai/meetstream-mcp) · [docs](https://docs.meetstream.ai/build-with-ai/meetstream-mcp-server) |
| ⌨️ | **CLI** - drive MeetStream from your terminal | [`@meetstream/cli`](https://www.npmjs.com/package/@meetstream/cli) · [docs](https://docs.meetstream.ai/build-with-ai/meetstream-cli) |
| 🤖 | **Claude Code plugin** - `/plugin marketplace add meetstream-ai/claude-plugin` | [docs](https://docs.meetstream.ai/build-with-ai/claude-integration) |
| 🧪 | **Labs** - runnable end-to-end templates | [github](https://github.com/meetstream-ai/labs) |
| 📖 | **Docs for agents** - machine-readable docs for your own tooling | [docs](https://docs.meetstream.ai/build-with-ai/docs-for-agents) |

## 🔗 Links

**Start here** · [Documentation](https://docs.meetstream.ai) · [API reference](https://docs.meetstream.ai/api-reference/introduction) · [API playground](https://docs.meetstream.ai/guides/get-started/api-playground) · [Dashboard setup](https://docs.meetstream.ai/guides/get-started/dashboard-setup)

**Go deeper** · [How bots work](https://docs.meetstream.ai/guides/introduction/how-bots-work) · [Webhooks & events](https://docs.meetstream.ai/guides/webhooks/webhooks-and-events) · [Scheduling bots](https://docs.meetstream.ai/guides/features/scheduling-bots) · [Idempotency & dedup](https://docs.meetstream.ai/guides/features/deduplication-idempotency-keys) · [Usage & retention](https://docs.meetstream.ai/guides/features/usage-and-retention) · [Automatic leave](https://docs.meetstream.ai/guides/features/automatic-leave-configuration)

**When stuck** · [Errors](https://docs.meetstream.ai/errors) · [Debugging bots](https://docs.meetstream.ai/guides/help/debugging-bots) · [FAQ](https://docs.meetstream.ai/guides/help/faq) · [Support](https://docs.meetstream.ai/guides/help/support) · [support@meetstream.ai](mailto:support@meetstream.ai)

<div align="center">

---

Built by [MeetStream.ai](https://meetstream.ai) · [MIT](LICENSE)

</div>
