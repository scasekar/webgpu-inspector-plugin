# WebGPU Inspector Plugin for Claude Code

Debug WebGPU applications from Claude Code (and any other MCP-capable LLM client) via either the **`webgpu-inspector-mcp`** server or the **`webgpu-inspector-cli`** terminal CLI. Both share the same Bridge — same Playwright session, same buffer decoders. The MCP server keeps a single browser alive across tool calls, which is what you want when an agent is driving the debug session.

## What's Included

- **Skill**: `webgpu-inspector` — Triggers when debugging WebGPU rendering issues, GPU validation errors, shader bugs, or performance problems
- **Command**: `/webgpu-inspect <url>` — Launch a full debugging session against a WebGPU app
- **Agent**: `webgpu-debugger` — Specialized subagent for WebGPU diagnosis

## Installation

### From Claude Code

```
/plugin marketplace add scasekar/webgpu-inspector-plugin
/plugin install webgpu-inspector
/reload-plugins
```

### Prerequisites

```bash
pip install webgpu-inspector-cli
python -m playwright install chromium
```

This installs both `webgpu-inspector-cli` (CLI + REPL) and `webgpu-inspector-mcp` (MCP server).

### Configure the MCP server

**Recommended for agent-driven debugging.** The server is a long-lived process: one browser, one inspector session, every tool call lands in the same context.

**Claude Code** — add to `~/.claude/mcp.json` or project `.mcp.json`:

```json
{
  "mcpServers": {
    "webgpu-inspector": {
      "command": "webgpu-inspector-mcp"
    }
  }
}
```

**Claude Desktop** — add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or the equivalent on your platform:

```json
{
  "mcpServers": {
    "webgpu-inspector": {
      "command": "webgpu-inspector-mcp"
    }
  }
}
```

Restart the client. Tools appear with `browser_launch`, `browser_eval`, `capture_frame`, `capture_buffer`, etc.

## Usage

### Automatic (via skill)

When you ask Claude to debug a WebGPU app, the skill activates and uses the MCP tools (or CLI REPL) to inspect GPU state, drive the page, and diagnose problems.

### Manual (via command)

```
/webgpu-inspect https://your-webgpu-app.com
```

### As a subagent

The `webgpu-debugger` agent can be dispatched to investigate GPU issues in the background while you continue working.

## How It Works

The MCP server / CLI uses Playwright to launch Chromium with WebGPU enabled, injects the [WebGPU Inspector](https://github.com/brendan-duncan/webgpu_inspector) JavaScript directly into the page, and intercepts every WebGPU API call. A collector script accumulates GPU state (objects, errors, shader source, captured buffers/textures) and exposes it to Python via CDP.

This gives you the same inspection capabilities as the Chrome DevTools panel, but fully programmable from an LLM client (MCP) or a terminal (CLI).

### Why MCP instead of shelling out to the CLI?

Each `webgpu-inspector-cli ...` invocation is a separate Python process, so the browser dies between calls — `browser launch` followed by a separate `capture frame` won't work. The MCP server is a single long-lived process, so the browser persists for the whole debug session. If you're driving from an LLM, MCP is the right interface.

The CLI is still the right tool for terminal use; just run `webgpu-inspector-cli` (no subcommand) to enter REPL mode where the browser persists for the whole session.

## Tool / Command surface

The MCP server exposes 24 tools that mirror the CLI command groups:

| Group | MCP tools | CLI commands |
|---|---|---|
| `browser` | launch, close, navigate, screenshot, status, **eval**, **click**, **type**, **wait** | same |
| `objects` | list, inspect, search, memory | same |
| `capture` | frame, commands, texture, buffer | same |
| `shaders` | list, view, replace, revert | list, view, compile, revert |
| `errors` | list, clear | list, watch, clear |
| `status` | summary | summary, fps, memory |

**Page-driving primitives** (`browser_eval`, `browser_click`, `browser_type`, `browser_wait`) let an agent drive any app that requires interaction past initial load — no need to add `?autoload=1` URL hacks to your app for inspector visits.

**Buffer decoding** (`capture_buffer`) supports `hex`, `hex-dump`, `u32-list`, `i32-list`, `f32-list`, `f32-mat4`, `raw` (base64), or a struct-spec like `'mat4x4 m; u32 chunkId; pad12'` for arbitrary record layouts.

**Buffer usage flags** are decoded in `objects_list(type="Buffer")` so `Storage | Indirect | CopyDst` is visible at a glance.

**Pre-navigation console capture**: `browser_launch(capture_console_path=...)` writes every console message and page error to a file, including those that fire during page bootstrap.

**Persistent Chrome profiles**: `browser_launch(user_data_dir=...)` reuses an existing browser profile (cookies, localStorage, extensions). Useful when the target app needs existing browser state.

## Links

- [CLI Tool Repository](https://github.com/scasekar/webgpu-inspector-cli)
- [WebGPU Inspector](https://github.com/brendan-duncan/webgpu_inspector)

## License

MIT
