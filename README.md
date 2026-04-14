# WebGPU Inspector Plugin for Claude Code

A Claude Code plugin that enables AI agents to debug WebGPU applications from the command line.

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

The plugin requires the WebGPU Inspector CLI tool:

```bash
pip install cli-anything-webgpu-inspector
python -m playwright install chromium
```

Or from source:
```bash
git clone --recurse-submodules https://github.com/scasekar/webgpu-inspector-cli
cd webgpu-inspector-cli/agent-harness && pip install -e .
python -m playwright install chromium
```

## Usage

### Automatic (via skill)

When you're working on a WebGPU app and ask Claude to debug rendering issues, the skill activates automatically. Claude will use the CLI to inspect GPU state, check for errors, and diagnose problems.

### Manual (via command)

```
/webgpu-inspect https://your-webgpu-app.com
```

### As a subagent

The `webgpu-debugger` agent can be dispatched to investigate GPU issues in the background while you continue working.

## How It Works

The CLI uses Playwright to launch Chrome, injects the [WebGPU Inspector](https://github.com/brendan-duncan/webgpu_inspector) JavaScript directly into the page, and intercepts all WebGPU API calls. A collector script accumulates GPU state which is queried from Python via CDP.

This gives you the same inspection capabilities as the Chrome DevTools extension, but fully programmable from the command line.

## CLI Commands

| Group | Commands | Purpose |
|-------|----------|---------|
| `browser` | launch, close, navigate, screenshot, status | Session lifecycle |
| `objects` | list, inspect, search, memory | GPU object inspection |
| `capture` | frame, commands, texture, buffer | Frame capture + data |
| `shaders` | list, view, compile, revert | Shader inspection + hot-reload |
| `errors` | list, watch, clear | Validation error tracking |
| `status` | summary, fps, memory | Runtime monitoring |

## Links

- [CLI Tool Repository](https://github.com/scasekar/webgpu-inspector-cli)
- [WebGPU Inspector](https://github.com/brendan-duncan/webgpu_inspector)

## License

MIT
