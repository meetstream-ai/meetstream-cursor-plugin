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

The [MeetStream Claude plugin](https://github.com/meetstream-ai/claude-plugin) is a separate, complementary install. It ships **skills only** and does not include the MCP server, so install both:

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

### Claude (desktop and web)

Customize -> Connectors -> + -> Add custom connector. Name it `meetstream` and set the URL to `https://mcp.meetstream.ai/mcp?key=YOUR_MEETSTREAM_API_KEY`, leaving the OAuth fields blank. Claude's custom connectors cannot send a custom auth header, so the key goes in the URL; give each person their own key so it can be revoked individually.

### Codex

Add to `~/.codex/config.toml` (the key is read from your environment):

```toml
[mcp_servers.meetstream]
url = "https://mcp.meetstream.ai/mcp"
bearer_token_env_var = "MEETSTREAM_API_KEY"
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
- Do not describe an MCP tool that does not exist. MIA agent configs, calendar connection, Google signed-in bots, Zoom authenticated joins (`zoom.zak_url` / `zoom.obf_url`), storage config and pause/resume are **REST-only** - route to REST rather than implying a tool.

## API rules that are easy to get wrong

These are live-verified. Do not "fix" code that follows them.

- **Auth differs by surface.** The REST API at `api.meetstream.ai` uses `Authorization: Token <key>`. The MCP server at `mcp.meetstream.ai` uses `Authorization: Bearer <key>`. Mixing them up returns 401.
- Every webhook carries the event name under **`event`**, and most also carry **`bot_event`** with the specific name.
- **Terminals are two-layer.** Every ending arrives once with `event: "bot.stopped"`; `bot_event` says why (`bot.stopped`, `bot.kicked`, `bot.notallowed`, `bot.denied`, `bot.failed`). Lobby timeouts, denials and failures carry `status_code: 500`, clean exits and kicks `200`. Branch on `bot_event`: a kick and a clean exit both report `bot_status: "Stopped"`.
- Every event carries a `timestamp`.
- **Streaming-only providers produce no post-call transcript**: no `transcription.processed`, though `bot.done` still fires. A post-call transcript fetch for them returns `202` indefinitely, so any polling loop needs a cap.
- Over REST, transcripts are fetched by **`transcript_id`**, not `bot_id`. The MCP `get_transcript` tool is the exception: it takes `bot_id` and resolves the `transcript_id` itself. Either way, segments use **`transcript`**, not `text`.
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
