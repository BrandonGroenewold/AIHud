# JARVIS

A local, offline, privacy-first AI assistant inspired by Iron Man's JARVIS. It runs entirely on my own PC, talks by voice, and shows its state on an animated HUD.

This is a hobby project. The goal is a capable everyday coding assistant that costs nothing to run and keeps my data on my machine.

## Status

Early development. The backend scaffold exists and the HUD is a prototype running on dummy data. Wiring the two together is the current milestone.

- [x] FastAPI backend with REST and WebSocket chat
- [x] Voice input (Whisper) and output (Piper)
- [x] Session memory and sandboxed code execution
- [x] Provider-agnostic `ModelService` (local Ollama or Anthropic API)
- [x] Animated HUD prototype (Jarvis core, dummy data)
- [ ] HUD connected to the backend over WebSocket
- [ ] Real system metrics (GPU, CPU, RAM) on the HUD
- [ ] Panel system (maps, generated apps)
- [ ] Wake word and barge-in
- [ ] Hand gestures

## Stack

| Piece | Tool |
| --- | --- |
| Local model | Ollama running Qwen2.5-Coder 14B (Q4_K_M) |
| Backend | Python, FastAPI |
| Speech to text | Whisper |
| Text to speech | Piper |
| HUD | Single HTML file, canvas, anime.js |
| Hardware target | Windows PC, RTX 5070 (12 GB VRAM) |

## Cost

Everything runs locally and uses free, open-source software. The Claude API path exists in `ModelService` but is **disabled by default**: with no API key configured, nothing is ever sent to a remote service.

## Repo layout

```
jarvis/
  backend/          FastAPI app, voice pipeline, model service
  hud/
    index.html      Jarvis HUD (single file)
    vendor/
      anime.min.js  Vendored so the HUD works fully offline
  .env.example      Template for local configuration
  .gitignore
  README.md
```

## Setup

Prerequisites: Python 3.11+, [Ollama](https://ollama.com), and the Whisper and Piper models downloaded locally.

```bash
# 1. Pull the local model
ollama pull qwen2.5-coder:14b

# 2. Create a virtual environment and install dependencies
cd backend
python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt

# 3. Configure
copy ..\.env.example ..\.env  # then edit as needed

# 4. Run the backend
uvicorn app.main:app --reload
```

Then open the HUD at the URL the backend serves, or open `hud/index.html` directly to see the demo mode.

> Adjust the module path and commands above to match the actual backend layout.

## Configuration

Settings live in `.env` (never committed). Copy `.env.example` to get started.

| Variable | Purpose | Default |
| --- | --- | --- |
| `MODEL_PROVIDER` | `local` (Ollama) or `anthropic` | `local` |
| `OLLAMA_MODEL` | Local model name | `qwen2.5-coder:14b` |
| `ANTHROPIC_API_KEY` | Leave empty to keep Claude disabled | empty |

## HUD

The HUD is a single self-contained page: a geodesic sphere core with honeycomb rings, a segmented ring, and a dense particle shell, plus slim edge instruments (time tape and activity ticker). The core reacts to four states (idle, listening, thinking, speaking) and shifts from teal to amber when a request is escalated to Claude.

Controls in demo mode let me force each state and adjust dot density.

## Planned WebSocket events

The HUD will be driven by events from the backend instead of the scripted demo.

| Event | Meaning |
| --- | --- |
| `state` | `idle`, `listening`, `thinking`, or `speaking` |
| `brain` | `local` or `claude` |
| `transcript` | Partial and final speech-to-text |
| `reply_delta` | Streamed reply tokens |
| `tool_call` | Something Jarvis is running or reading |
| `metrics` | GPU, CPU, RAM, and context usage |

This protocol may change as it gets built.

## Roadmap

1. Connect the HUD to the backend (events above)
2. Replace dummy metrics with real ones (`pynvml`, `psutil`)
3. Panel system: draggable windows for maps and generated apps, sandboxed in iframes
4. Voice polish: wake word, streaming TTS, barge-in
5. Hand gestures via MediaPipe
6. Additional core designs selectable from a picker

## Security notes

- Secrets go in `.env`, which is git-ignored.
- Generated code runs in a sandbox, and generated UI panels run in sandboxed iframes without network access by default.
- Model weights, voice files, and local databases are not committed.
