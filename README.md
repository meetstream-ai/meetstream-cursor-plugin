# MeetStream for Cursor

Send AI meeting bots into **Zoom, Google Meet and Microsoft Teams** from your editor. Record, transcribe, summarize and interact with live meetings using **19 tools** over the hosted [MeetStream](https://meetstream.ai) MCP server.

Once installed, every agent on your account gets the tools. There is nothing to run locally and no server to host.

## Install

**From the Cursor marketplace** - search for "MeetStream", click **Add to Cursor**, then set your API key under **Plugins -> Configure**.

Get a key at [app.meetstream.ai/api-key](https://app.meetstream.ai/api-key).

## What you can do

Ask your agent, in plain language:

> "Send a MeetStream bot to https://meet.google.com/abc-defg-hij and record it."

> "Summarize that meeting and pull out the action items with owners."

> "Who talked the most on the last call?"

> "Build me an AI notetaker in this repo that emails the summary to attendees."

> "We're on Recall.ai. Migrate this project to MeetStream."

## Tools

The plugin connects to `https://mcp.meetstream.ai/mcp` and exposes 19 tools:

| Group | Tools |
|---|---|
| **Bot lifecycle** | `create_bot`, `list_bots`, `get_bot_status`, `get_bot_detail`, `get_bot_summary`, `remove_bot`, `delete_bot_data` |
| **Transcription** | `get_transcript`, `list_transcriptions`, `transcribe_audio` |
| **Meeting data** | `get_media_urls`, `get_participants`, `get_chats`, `get_speaker_timeline` |
| **Live interaction** | `send_chat_message`, `send_image` |
| **Calendar & reference** | `list_calendar_events`, `schedule_calendar_bot`, `webhook_events_guide` |

## Skills

Eleven skills teach the agent how to use MeetStream well - they load automatically when the conversation matches:

| Skill | Triggers on |
|---|---|
| **join-meeting** | "join this meeting", "send a bot to...", "record this call", "make the bot leave" |
| **meeting-brief** | "summarize that meeting", "action items", "what did we decide", "who talked the most" |
| **mia-voice-agents** | "voice agent", "AI agent in the meeting", "talking bot", "wake word", "MIA" |
| **webhooks** | "webhook", "callback_url", "bot events", "my webhook isn't firing" |
| **realtime-streaming** | "live transcription", "real-time captions", "stream the audio", "websocket" |
| **recordings-and-media** | "download the recording", "per-participant audio", "pause recording", "retention", "own S3 bucket" |
| **calendar-automation** | "auto-join my meetings", "connect my calendar", "recurring meetings" |
| **platform-setup** | "can't join Zoom", "waiting room", "recording permission", "signed-in bots", "Teams lobby" |
| **troubleshooting** | "401", "403", "stuck on 202", "507", "transcript never arrives", "duplicate bots" |
| **build-notetaker** | "build a notetaker", "integrate MeetStream into my app", "set up webhooks" |
| **migrate-from-recall** | "migrate from Recall", "move off recall.ai", "Recall alternative" |

Some MeetStream capabilities (MIA agent configs, calendar connection, signed-in bots, storage config) are REST-only and not MCP tools. The skills know the difference and route correctly.

## Your API key

Declared as a plugin variable and set in the Cursor dashboard, never in this repo. It is sent per request to the hosted MCP server, which is multi-tenant and holds no key of its own.

If you would rather the key never left your machine, run the server locally instead:

```bash
npx -y @meetstream/mcp    # stdio, reads MEETSTREAM_API_KEY from the environment
```

## Local development

```bash
git clone https://github.com/meetstream-ai/meetstream-cursor-plugin.git
ln -s "$PWD/meetstream-cursor-plugin" ~/.cursor/plugins/local/meetstream
```

Restart Cursor. The symlink means edits apply on the next restart without re-copying.

## Also available

- **MCP server** - [`@meetstream/mcp`](https://www.npmjs.com/package/@meetstream/mcp) - the same tools for any MCP client
- **CLI** - [`@meetstream/cli`](https://www.npmjs.com/package/@meetstream/cli) - drive MeetStream from your terminal
- **Claude Code plugin** - `/plugin marketplace add meetstream-ai/claude-plugin`
- **Labs** - [60+ runnable templates](https://github.com/meetstream-ai/labs) covering every endpoint
- **Migration kit** - [`@meetstream/migrate`](https://www.npmjs.com/package/@meetstream/migrate) - move off Recall.ai in one command

## Links

- [Documentation](https://docs.meetstream.ai)
- [API reference](https://docs.meetstream.ai/api-reference)
- [MCP server setup](https://docs.meetstream.ai/build-with-ai/meetstream-mcp-server)
- Support: [support@meetstream.ai](mailto:support@meetstream.ai)

MIT licensed.
