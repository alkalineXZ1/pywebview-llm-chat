# pywebview-llm-chat
Tailored made for Chutes API
# Bittensor Chat

A local desktop AI chat app powered by [Chutes.ai](https://chutes.ai), built with Python and pywebview. Runs entirely on Linux as a single file — no Electron, no browser required.

---

## Features

**Chat**
- Streaming responses with real-time token display
- Reasoning block support (models like DeepSeek, Kimi) — shows thinking time
- Auto-generated chat titles
- Edit & resend any user message
- Regenerate last assistant response
- Branch any conversation from any point
- Pin chats to the top of the sidebar
- Draft auto-saved per chat when switching

**Models**
- Switch models mid-session from the top bar
- Add or remove any `org/model-name` from the model list
- Side-by-side model comparison mode (same prompt, two models simultaneously)

**Organisation**
- Folder system with custom colors — drag chats between folders
- Chat search with match navigation and highlight
- Prompt templates — save and reuse common prompts

**File attachments**
- Images (PNG, JPG, GIF, WEBP) — sent as vision input
- PDF, DOCX, PPTX, XLSX, CSV, TXT, Markdown, Python
- Attach multiple files at once
- Image folder panel — load an entire folder, preview on hover, send images directly

**Web**
- Web search via Brave Search API (toggle per message)
- Inline URL fetch — paste a URL and the content is fetched automatically

**Voice**
- Voice input via `arecord` + Google Speech Recognition
- TTS playback via Chutes Kokoro API with multiple voice options

**Export**
- Markdown (`.md`)
- Plain text (`.txt`)
- HTML (styled, self-contained)
- PDF — uses `fpdf2` if installed, falls back to built-in writer

**Profiles**
- Password-protected profiles, each with isolated chats, folders, theme and settings
- Auto-saves on app close
- Guest mode wipes on every launch for a clean slate

**Themes**
- 4 built-in themes: Void, Nebula, Abyss, Matrix
- Theme editor — customize accent color, font size, bubble radius

**Privacy**
- E2EE proxy via Docker (`parachutes/e2ee-proxy`) auto-starts on launch, routes all API traffic through an encrypted local proxy
- Falls back to direct API if Docker is unavailable

---

## Requirements

**System**
- Linux
- Python 3.10+
- Docker (optional, for E2EE proxy)
- `arecord` (optional, for voice input) — `sudo apt install alsa-utils`

**Python — required**
```
pywebview
openai
requests
fpdf2
```

**Python — optional** (each unlocks a file type for attachments)
```
pypdf          # PDF reading
pandas         # XLSX, XLS, CSV
python-docx    # DOCX
python-pptx    # PPTX
SpeechRecognition  # voice input transcription
```

---

## Installation

```bash
git clone <repo>
cd bittensor-chat
pip install -r requirements.txt --break-system-packages
python app.py
```

---

## Configuration

On first launch a config file is created at `~/.bittensor_chat/config.json`.

Open **Settings** (`Ctrl+/`) to set:

| Field | Description |
|---|---|
| API Key | Your Chutes.ai API key |
| Brave Search API Key | Optional — enables web search |
| Base URL | API endpoint (auto-set by E2EE proxy on launch) |
| Max Tokens | Maximum tokens per response (default 10 000) |
| Max Memory | How many messages to keep in context (default 50) |
| Theme | Void / Nebula / Abyss / Matrix |

---

## Data

All data is stored locally in `~/.bittensor_chat/`:

```
~/.bittensor_chat/
├── config.json          # app settings
├── chats/               # guest session chats (wiped on launch/close)
└── profiles/
    ├── profiles.json    # profile manifest (hashed passwords)
    └── <profile-id>/
        ├── config.json
        ├── chats/
        └── theme_overrides.json
```

Guest mode chats are intentionally wiped on every clean launch and close. Use a profile to persist data.

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+N` | New chat |
| `Enter` | Send message |
| `Shift+Enter` | New line in input |
| `Ctrl+/` | Settings |
| `Ctrl+F` | Search chats |
| `Ctrl+B` | Toggle sidebar |
| `Ctrl+E` | Export chat |
| `Ctrl+W` | Toggle web search |
| `Ctrl+U` | Attach file(s) |
| `Ctrl+M` | Compare models |
| `?` | Keyboard shortcuts help |
| `Esc` | Close any modal |

---

## E2EE Proxy

On startup the app attempts to pull and run `parachutes/e2ee-proxy:latest` via Docker, exposing it on port `8443`. All API calls are routed through this proxy for end-to-end encryption. If Docker is not installed or the container fails to start, the app falls back to `https://llm.chutes.ai/v1` directly.

To use E2EE: make sure Docker is running before launching the app.

---

## Profiles

Profiles let you maintain separate identities with isolated chat history, folders, templates and theme settings. Each profile is password-protected (scrypt hashing).

- Create a profile from the profile modal (top-right avatar button)
- Passwords are hashed with **scrypt** (n=16384, r=8, p=1) — not reversible
- A master key (`admin`) can unlock any profile — change `MASTER_KEY` in the source if deploying for others
