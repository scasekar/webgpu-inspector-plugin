---
description: Launch the WebGPU Inspector to debug a WebGPU web application. Provide a URL to inspect.
argument-hint: <url>
allowed-tools: Bash(webgpu-inspector-cli:*), Bash(webgpu-inspector-mcp:*), Bash(pip install:*), Bash(python -m playwright:*), Read, Write
---

# /webgpu-inspect

Debug a WebGPU web application.

## Prerequisites

```bash
pip install webgpu-inspector-cli
python -m playwright install chromium
```

This installs both the CLI (`webgpu-inspector-cli`) and the MCP server (`webgpu-inspector-mcp`).

## Recommended path: MCP server

If the user has not yet configured the MCP server in Claude Code, instruct them to add this to `~/.claude/mcp.json` (or project `.mcp.json`):

```json
{
  "mcpServers": {
    "webgpu-inspector": {
      "command": "webgpu-inspector-mcp"
    }
  }
}
```

…and restart Claude Code. Then call the inspector tools directly: `browser_launch`, `browser_eval`, `capture_frame`, `capture_buffer`, etc. The browser stays alive across tool calls.

## Fallback: CLI REPL (terminal)

If MCP isn't an option, drive the inspector through one REPL session — **don't** chain separate `webgpu-inspector-cli ...` shell invocations because each one spawns a fresh browser and the second command will fail with "No active browser session".

```bash
webgpu-inspector-cli                          # opens REPL
> browser launch --url <URL>
> browser wait --condition 'window.app !== undefined' --timeout 10
> --json errors list
> --json status summary
> --json objects list
> --json objects list --type Buffer           # decoded usage flags
> capture frame
> capture buffer --id <ID> --format f32-mat4
> shaders view --id <ID>
> capture texture --id <ID> -o debug.png
> shaders compile --id <ID> --file fixed.wgsl
> browser close
> exit
```

## Workflow steps (apply to MCP or REPL)

1. **Launch** with the target URL. Pass `capture_console_path` (MCP) / `--capture-console` (CLI) to record pre-navigation console output.

2. **Drive the page if needed.** If the app requires interaction past initial load (clicking "Load scene", logging in, etc.), use:
   - `browser_eval` / `browser eval --js '...'` — run any JS expression
   - `browser_click` / `browser click '<sel>'`
   - `browser_type` / `browser type '<sel>' '<text>'`
   - `browser_wait` / `browser wait --condition '...'`

   Don't tell the user to add URL-param load hacks to their app.

3. **Diagnose** — check validation errors first, then state overview, then object lists.

4. **Investigate** based on findings:
   - **Validation errors:** Read the stacktrace to find the offending API call.
   - **Missing objects:** filter `objects_list` by type.
   - **Shader bugs:** `shaders_view(id)` to read WGSL.
   - **Visual bugs:** `capture_frame` then `capture_texture(id, output_path="debug.png")`.
   - **Buffer OOB / suspicious values:** `capture_frame` then `capture_buffer(id, format="u32-list")` or with a `struct_spec`.

5. **Fix and verify:**
   - Hot-reload shaders: `shaders_replace(id, code=...)` (MCP) / `shaders compile --id <ID> --file fixed.wgsl` (CLI).
   - Re-check: `errors_list`, `status_summary`.

6. **Clean up:** `browser_close()`.

## Tips

- Always parse JSON output (`--json` flag on CLI; MCP tools return JSON natively).
- Object IDs are integers — get them from `objects_list` or `shaders_list`.
- Frame capture is async — the tool polls until complete (default 30s).
- Use `--headless` (CLI) / `headless=true` (MCP) with `--gpu-backend swiftshader` / `gpu_backend="swiftshader"` for headless environments without a GPU.
- `capture_buffer` requires a prior `capture_frame` — buffer data is collected during frame capture, not on demand.
