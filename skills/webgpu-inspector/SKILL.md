---
name: webgpu-inspector
description: Use when debugging WebGPU rendering issues, investigating GPU validation errors, inspecting GPU objects (buffers, textures, shaders, pipelines), profiling frame performance, or diagnosing visual artifacts in a web application that uses the WebGPU API
---

# WebGPU Inspector CLI

Debug WebGPU applications from the command line using `cli-anything-webgpu-inspector`. Launches a browser, injects the WebGPU Inspector, and provides structured access to all GPU state.

## Setup

```bash
pip install cli-anything-webgpu-inspector
python -m playwright install chromium
```

If installing from source:
```bash
git clone --recurse-submodules https://github.com/scasekar/webgpu-inspector-cli
cd webgpu-inspector-cli/agent-harness
pip install -e .
python -m playwright install chromium
```

Verify: `cli-anything-webgpu-inspector --help`

## Debugging Workflow

Always use `--json` for machine-readable output.

```bash
# 1. Launch browser with target app
cli-anything-webgpu-inspector browser launch --url <URL>

# 2. Check for validation errors first (most common issue)
cli-anything-webgpu-inspector --json errors list

# 3. Get state overview
cli-anything-webgpu-inspector --json status summary

# 4. Inspect specific objects
cli-anything-webgpu-inspector --json objects list
cli-anything-webgpu-inspector --json objects list --type Texture
cli-anything-webgpu-inspector --json objects inspect --id <ID>

# 5. Read shader source code
cli-anything-webgpu-inspector --json shaders list
cli-anything-webgpu-inspector shaders view --id <ID>

# 6. Capture a frame to inspect GPU commands
cli-anything-webgpu-inspector --json capture frame
cli-anything-webgpu-inspector --json capture commands

# 7. Inspect texture contents (save as PNG)
cli-anything-webgpu-inspector capture texture --id <ID> -o debug.png

# 8. Hot-reload a shader fix
cli-anything-webgpu-inspector shaders compile --id <ID> --file fixed_shader.wgsl

# 9. Clean up
cli-anything-webgpu-inspector browser close
```

## Command Reference

| Group | Commands | Purpose |
|-------|----------|---------|
| `browser` | launch, close, navigate, screenshot, status | Session lifecycle |
| `objects` | list, inspect, search, memory | GPU object inspection |
| `capture` | frame, commands, texture, buffer | Frame capture + data |
| `shaders` | list, view, compile, revert | Shader inspection + hot-reload |
| `errors` | list, watch, clear | Validation error tracking |
| `status` | summary, fps, memory | Runtime monitoring |

## Object Types

Adapter, Device, Buffer, Texture, TextureView, Sampler, ShaderModule, BindGroup, BindGroupLayout, PipelineLayout, RenderPipeline, ComputePipeline, RenderBundle

## Key Flags

- `--json` on root command: all output as JSON
- `--type TYPE` on `objects list`: filter by GPU object type
- `--id ID`: target a specific object (integer)
- `--timeout N` on capture/watch: seconds to wait (default 30)
- `--headless` on `browser launch`: headless Chrome (needs GPU or `--gpu-backend swiftshader`)
- `-o PATH` on `capture texture`: save texture as PNG

## Common Debugging Scenarios

**Validation errors:** `errors list --json` shows error message + creation stacktrace pinpointing the bad API call.

**Missing objects:** `objects list --json` shows all live GPU objects. Check if expected pipelines/textures exist. `objects inspect --id N` shows the full descriptor and creation stacktrace.

**Shader bugs:** `shaders view --id N` to read WGSL source. `shaders compile --id N --file fix.wgsl` to hot-reload without restarting the app. `shaders revert --id N` to undo.

**Performance:** `status fps` for frame rate. `status memory` for GPU memory breakdown. `objects list --type Buffer` and `--type Texture` to find large allocations.

**Visual artifacts:** `capture frame` then `capture texture --id N -o output.png` to inspect render targets pixel-by-pixel.
