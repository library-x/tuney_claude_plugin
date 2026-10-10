# Tuney plugin for Claude

Hand-made, modular music fitted to your exact length, inside Claude. This plugin points Claude at the
Tuney MCP connector (`.mcp.json`) and teaches it how to use the tools (`skills/tuney-music/SKILL.md`).
It contains no code; the connector itself is a remote server run by Tuney.

## Install in Claude Code

```bash
claude plugin marketplace add library-x/tuney_claude_plugin
claude plugin install tuney@tuney
```

Then run `/mcp` in Claude Code and sign in with your Tuney account. The first tool call prompts for
sign-in if you skipped it.

## What you get

- `find_tracks`: instant, free previews for a brief or a genre, mood and tempo
- `generate_track`: a new track composed to an exact length
- `adjust_track`: change length, tempo, key, or mute stems
- `download_track`: mp3, wav or stems (uses your Tuney credits)

## Connector URL

`https://api.tuney.io/cue-engine2/connector/mcp`. You can also add it directly without the plugin:

```bash
claude mcp add --transport http tuney https://api.tuney.io/cue-engine2/connector/mcp
```


## Other assistants and editors

The same Tuney MCP server works in claude.ai, Claude Desktop, ChatGPT, Cursor, VS Code, Codex CLI, Gemini CLI and any other MCP client. Install instructions for each are in [library-x/tuney_mcp](https://github.com/library-x/tuney_mcp).

## Validate

```bash
claude plugin validate --strict .
```
