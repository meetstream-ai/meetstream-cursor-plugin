# AGENTS.md

Guidance for AI coding agents working in this repository. This repo is the **MeetStream plugin for Cursor** - a thin wrapper around the hosted MCP server plus 10 skills.

## Always use the MeetStream MCP

When building anything against the MeetStream API, install and use the MeetStream MCP. It is the current source of truth for endpoints, request and response fields, webhook events, provider names and bot behaviour.

Do not treat the files in this repository as authoritative for MeetStream API behaviour. They can lag the API. If local code and the MCP disagree, the MCP wins unless the user says otherwise.

Server name: `meetstream`. Hosted URL: `https://mcp.meetstream.ai/mcp`.

### Claude Code

```sh
claude mcp add --transport http meetstream https://mcp.meetstream.ai/mcp \
  --header "Authorization: Bearer $MEETSTREAM_API_KEY"
```

Or install the plugin, which bundles the server plus skills:

```sh
/plugin marketplace add meetstream-ai/claude-plugin
```

### Cursor

Add to `~/.cursor/mcp.json`, or install the [MeetStream Cursor plugin](https://github.com/meetstream-ai/meetstream-cursor-plugin):

```json
{
  "mcpServers": {
    "meetstream": {
      "url": "https://mcp.meetstream.ai/mcp",
      "headers": { "Authorization": "Bearer YOUR_MEETSTREAM_API_KEY" }
    }
  }
}
```

### Windsurf

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "meetstream": {
      "serverUrl": "https://mcp.meetstream.ai/mcp",
      "headers": { "Authorization": "Bearer YOUR_MEETSTREAM_API_KEY" }
    }
  }
}
```

### Claude Desktop

Settings -> Connectors -> Add custom connector. Name it `meetstream`, URL `https://mcp.meetstream.ai/mcp`.

### Codex

```sh
codex mcp add meetstream --url https://mcp.meetstream.ai/mcp \
  --header "Authorization: Bearer $MEETSTREAM_API_KEY"
```

### Run it locally instead

```sh
MEETSTREAM_API_KEY=ms_... npx -y @meetstream/mcp
```

### Use the MCP before

- calling any MeetStream endpoint
- changing webhook handling or event names
- adding or changing bot, transcription, calendar or MIA behaviour
- relying on any request field, response field, provider name or status code

## Working in this repo

There is no build and no test runner. The plugin is declarative.

```
.cursor-plugin/plugin.json   manifest, MEETSTREAM_API_KEY plugin variable
mcp.json                     Cursor connector  -> mcp.meetstream.ai
.mcp.json                    Claude Code / agent-plugin connector
skills/<name>/SKILL.md        10 skills
```

### Testing a change locally

```sh
ln -s "$PWD" ~/.cursor/plugins/local/meetstream
```

Restart Cursor. The symlink means edits apply on the next restart with no re-copying. Confirm the server is live by asking the agent to call `list_bots`.

### Manifest rules

- The API key is declared as a **plugin variable** and interpolated as `${MEETSTREAM_API_KEY}`. It is set in the Cursor dashboard and must never appear in this repo.
- `variables` must be a JSON-Schema object (`{"type":"object","properties":{...}}`).
- Interpolation only happens in `command`, `args`, `env`, `url` and `headers`.
- `mcp.json` must use **`Authorization: Bearer`**. `Token` returns 401 against the MCP server.

### Skill rules

- A skill lives at `skills/<name>/SKILL.md` and its YAML frontmatter `name:` **must match the directory name** exactly.
- The `description:` is what triggers loading. Write the phrases a user would actually say.
- Every link must resolve.
- Do not describe an MCP tool that does not exist. MIA agent configs, calendar connection, Google signed-in bots, Zoom OAuth, storage config and pause/resume are **REST-only** - route to REST rather than implying a tool.

## API rules that are easy to get wrong

These are live-verified. Do not "fix" code that follows them.

- **Auth differs by surface.** The REST API at `api.meetstream.ai` uses `Authorization: Token <key>`. The MCP server at `mcp.meetstream.ai` uses `Authorization: Bearer <key>`. Mixing them up returns 401.
- The webhook envelope key is **`event`**, not `bot_event`.
- **`bot.stopped` is the single terminal event** and always carries `status_code: 200`. The reason lives in `bot_status`: `Stopped`, `NotAllowed` (waiting-room timeout), `Denied` (host refused), `Error`.
- `bot.error` is **non-terminal** - the bot keeps running.
- **Streaming-only providers never emit `bot.done`.** They end at `audio.processed`, and a post-call transcript fetch returns `202` forever, so any polling loop needs a cap.
- Transcripts are fetched by **`transcript_id`**, not `bot_id`, and segments use **`transcript`**, not `text`.
- **`202` and `507` are not errors.** 202 means poll again; 507 means an idempotent retry replayed and is a success.
- The bot field is **`meeting_link`**, not `meeting_url`.
- `in_call_recording_timeout` has a hard floor of **600 seconds**; below it the API returns 400.
- MIA bots take **only `agent_config_id`**. Adding `socket_connection_url` or `live_audio_required` alongside it is the usual cause of a silent agent.

## Security

- Never hard-code or commit a key. `ms_...` values belong in the environment.
- Never log a key, a transcript, or participant data.
- Do not expose a server-side key to client code.
- Verify webhook signatures before acting on a payload.
- Do not persist meeting, transcript, participant or recording data unless asked.

## Before you finish

- Every skill's frontmatter `name` matches its directory.
- Every `${VAR}` placeholder is declared in `variables`.
- All three JSON files parse.
- Every link resolves.
- State which MCP tools or docs you relied on, what changed, and what you did not verify.

## Related

- Docs: https://docs.meetstream.ai
- MCP server: [`@meetstream/mcp`](https://github.com/meetstream-ai/meetstream-mcp)
- CLI: [`@meetstream/cli`](https://github.com/meetstream-ai/meetstream-cli)
- Claude Code plugin: https://github.com/meetstream-ai/claude-plugin
- Cursor plugin: https://github.com/meetstream-ai/meetstream-cursor-plugin
- Runnable examples: https://github.com/meetstream-ai/labs
