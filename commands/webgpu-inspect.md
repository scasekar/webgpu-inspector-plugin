---
description: Launch the WebGPU Inspector CLI to debug a WebGPU web application. Provide a URL to inspect.
argument-hint: <url>
allowed-tools: Bash(webgpu-inspector-cli:*), Bash(pip install:*), Bash(python -m playwright:*), Read, Write
---

# /webgpu-inspect

Debug a WebGPU web application using the WebGPU Inspector CLI.

## Prerequisites

Ensure the CLI is installed. Check with `which webgpu-inspector-cli`. If not found, install:
```bash
pip3 install webgpu-inspector-cli
python3 -m playwright install chromium
```

## Workflow

1. **Launch** the browser with the target URL:
   ```bash
   webgpu-inspector-cli browser launch --url <URL>
   ```

2. **Diagnose** — Run these checks in order:
   ```bash
   # Check for validation errors (most common issue)
   webgpu-inspector-cli --json errors list

   # Get GPU state overview
   webgpu-inspector-cli --json status summary

   # List all GPU objects
   webgpu-inspector-cli --json objects list
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
   webgpu-inspector-cli browser close
   ```

## Tips

- Always use `--json` flag for structured output you can parse
- Object IDs are integers — get them from `objects list` or `shaders list`
- Frame capture is async — the `capture frame` command polls until complete
- Use `--headless` with `--gpu-backend swiftshader` for headless environments
