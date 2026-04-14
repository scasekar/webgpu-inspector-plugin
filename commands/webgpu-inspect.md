---
description: Launch the WebGPU Inspector CLI to debug a WebGPU web application. Provide a URL to inspect.
argument-hint: <url>
allowed-tools: Bash(cli-anything-webgpu-inspector:*), Bash(pip install:*), Bash(python -m playwright:*), Read, Write
---

# /webgpu-inspect

Debug a WebGPU web application using the WebGPU Inspector CLI.

## Prerequisites

Ensure the CLI is installed:
```bash
pip install cli-anything-webgpu-inspector 2>/dev/null || pip3 install cli-anything-webgpu-inspector
python -m playwright install chromium
```

If `cli-anything-webgpu-inspector` is not available via pip, install from source:
```bash
git clone --recurse-submodules https://github.com/scasekar/webgpu-inspector-cli /tmp/webgpu-inspector-cli
cd /tmp/webgpu-inspector-cli/agent-harness && pip install -e .
python -m playwright install chromium
```

## Workflow

1. **Launch** the browser with the target URL:
   ```bash
   cli-anything-webgpu-inspector browser launch --url <URL>
   ```

2. **Diagnose** — Run these checks in order:
   ```bash
   # Check for validation errors (most common issue)
   cli-anything-webgpu-inspector --json errors list

   # Get GPU state overview
   cli-anything-webgpu-inspector --json status summary

   # List all GPU objects
   cli-anything-webgpu-inspector --json objects list
   ```

3. **Investigate** based on findings:
   - **Validation errors found:** Read the stacktrace to find the offending API call
   - **Missing objects:** Check if expected pipelines/textures exist with `objects list --type <TYPE>`
   - **Shader issues:** `shaders view --id <ID>` to read WGSL source
   - **Visual bugs:** `capture frame` then `capture texture --id <ID> -o debug.png`

4. **Fix and verify:**
   - Hot-reload shaders: `shaders compile --id <ID> --file fixed.wgsl`
   - Re-check: `errors list` and `status summary`

5. **Clean up:**
   ```bash
   cli-anything-webgpu-inspector browser close
   ```

## Tips

- Always use `--json` flag for structured output you can parse
- Object IDs are integers — get them from `objects list` or `shaders list`
- Frame capture is async — the `capture frame` command polls until complete
- Use `--headless` with `--gpu-backend swiftshader` for headless environments
