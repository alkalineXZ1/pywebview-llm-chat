# Getting Started — Bittensor Chat

A step-by-step guide to setting up and running Bittensor Chat on Linux.

---

## Prerequisites

Make sure the following are available on your system before you begin.

**Required**
- Linux (Ubuntu 22.04+ recommended)
- Python 3.10 or newer
- Git

**Optional but recommended**
- Docker — enables the E2EE proxy (see [E2EE Proxy](#e2ee-proxy))
- `arecord` — enables voice input (`sudo apt install alsa-utils`)

Check your Python version:

```bash
python3 --version
```

---

## 1. Clone the Repository

```bash
git clone <repo-url>
cd bittensor-chat
```

---

## 2. Create a Virtual Environment

Using a virtual environment keeps your system Python clean and avoids dependency conflicts.

```bash
# Create the venv inside the project folder
python3 -m venv .venv

# Activate it
source .venv/bin/activate
```

Your terminal prompt will change to show `(.venv)` — this confirms the environment is active.

To deactivate later (when you're done):

```bash
deactivate
```

> **Tip:** Always activate the venv before running the app or installing packages.

---

## 3. Install Dependencies

With the venv active, install everything from `requirements.txt`:

```bash
pip install -r requirements.txt
```

This installs both required and optional packages:

| Package | Purpose |
|---|---|
| `openai` | API client for Chutes.ai |
| `pywebview` | Desktop window (no browser/Electron needed) |
| `requests` | HTTP calls |
| `fpdf2` | PDF export |
| `pypdf` | PDF attachment reading |
| `pandas` + `openpyxl` | XLSX / CSV attachment reading |
| `python-docx` | DOCX attachment reading |
| `python-pptx` | PPTX attachment reading |
| `SpeechRecognition` | Voice input transcription |

> If you ever need to reinstall on a system Python without a venv, add `--break-system-packages`:
> ```bash
> pip install -r requirements.txt --break-system-packages
> ```

---

## 4. Set Up Environment Variables

Bittensor Chat reads configuration from `~/.bittensor_chat/config.json`, which is created automatically on first launch. However, you can pre-set sensitive values using a `.env` file so they're never hardcoded.

### Create a `.env` file

In the project root, create a file named `.env`:

```bash
touch .env
```

Add your keys:

```env
# Required — get yours at https://chutes.ai
CHUTES_API_KEY=your_chutes_api_key_here

# Optional — enables web search (Brave Search API)
BRAVE_SEARCH_API_KEY=your_brave_api_key_here
```

### Load the `.env` before launching

The app does not automatically load `.env` files. Use one of these approaches:

**Option A — export manually each session:**

```bash
export $(grep -v '^#' .env | xargs)
python app.py
```

**Option B — create a launch script (`run.sh`):**

```bash
cat > run.sh << 'EOF'
#!/bin/bash
set -a
source "$(dirname "$0")/.env"
set +a
source "$(dirname "$0")/.venv/bin/activate"
exec python "$(dirname "$0")/app.py"
EOF

chmod +x run.sh
```

Then just run:

```bash
./run.sh
```

**Option C — install `python-dotenv` and load it in-app:**

```bash
pip install python-dotenv
```

Then add this near the top of `app.py`:

```python
from dotenv import load_dotenv
load_dotenv()
```

> ⚠️ Add `.env` to your `.gitignore` to avoid leaking API keys:
> ```bash
> echo ".env" >> .gitignore
> ```

---

## 5. E2EE Proxy (Optional but Recommended)

The app can route all API traffic through an end-to-end encrypted local proxy provided by Chutes.ai. This requires Docker.

### Install Docker

If Docker is not installed:

```bash
# Ubuntu / Debian
sudo apt update
sudo apt install docker.io
sudo systemctl enable --now docker

# Add your user to the docker group (avoid needing sudo)
sudo usermod -aG docker $USER
newgrp docker
```

Verify Docker is running:

```bash
docker info
```

### How the proxy works

On startup, the app automatically pulls and runs `parachutes/e2ee-proxy:latest`, exposing it on port `8443`. All API calls are then routed through this encrypted proxy. If Docker is unavailable, the app falls back to `https://llm.chutes.ai/v1` directly.

You do not need to start the container manually — the app handles it.

### Manual setup / advanced configuration

For full documentation on the E2EE proxy (including Docker Compose setup, environment variables, and self-hosting), refer to the official repository:

👉 **https://github.com/chutesai/e2ee-proxy**

---

## 6. First Launch & Configuration

With the venv active (and `.env` loaded if applicable), start the app:

```bash
python app.py
```

A desktop window will open. On the very first launch:

1. Open **Settings** with `Ctrl+/`
2. Enter your **Chutes.ai API key** (or it will be read from the environment)
3. Optionally set a **Brave Search API key** for web search
4. Choose your preferred **theme** (Void, Nebula, Abyss, Matrix)
5. Adjust **Max Tokens** (default: 10,000) and **Max Memory** (default: 50 messages)

Configuration is saved to `~/.bittensor_chat/config.json` automatically.

---

## 7. Quick Reference

### Daily workflow

```bash
# Navigate to the project
cd bittensor-chat

# Activate the environment
source .venv/bin/activate

# Load API keys and launch
./run.sh        # if you created run.sh
# or
export $(grep -v '^#' .env | xargs) && python app.py
```

### Key shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+N` | New chat |
| `Ctrl+/` | Settings |
| `Ctrl+F` | Search chats |
| `Ctrl+B` | Toggle sidebar |
| `Ctrl+E` | Export chat |
| `Ctrl+W` | Toggle web search |
| `Ctrl+U` | Attach file(s) |
| `Ctrl+M` | Compare models side-by-side |
| `?` | Keyboard shortcuts help |
| `Esc` | Close any modal |

### Data location

All data lives at `~/.bittensor_chat/`:

```
~/.bittensor_chat/
├── config.json              # App settings
├── chats/                   # Guest session chats (wiped on each launch)
└── profiles/
    ├── profiles.json        # Profile manifest (hashed passwords)
    └── <profile-id>/
        ├── config.json
        ├── chats/
        └── theme_overrides.json
```

> Guest mode chats are intentionally wiped on every launch and close. Create a **Profile** (`top-right avatar button`) to persist your conversations.

---

## Troubleshooting

**`pywebview` window doesn't open**
Install GTK/WebKit dependencies:
```bash
sudo apt install python3-gi python3-gi-cairo gir1.2-gtk-3.0 gir1.2-webkit2-4.0
```

**Docker permission denied**
```bash
sudo usermod -aG docker $USER
newgrp docker
```

**Voice input not working**
```bash
sudo apt install alsa-utils
# Test recording
arecord -d 3 test.wav && aplay test.wav
```

**PDF export fails**
Ensure `fpdf2` is installed in the active venv:
```bash
pip show fpdf2
```
