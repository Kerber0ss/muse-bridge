<p align="center">
  <img src="docs/logo.svg" width="88" alt="MuseBridge Logo">
</p>

<h1 align="center">MuseBridge</h1>

<p align="center">
  An open-source Muse AI API reverse proxy 
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10%2B-3478F6?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/License-MIT-3478F6" alt="License">
  <img src="https://img.shields.io/badge/version-1.7.1-3478F6" alt="Version">
  <img src="https://img.shields.io/badge/API-OpenAI%20Compatible-0F8B72" alt="OpenAI Compatible">
</p>

---

## 📖 Contents

  - [Introduction](#introduction)
  - [Features](#features)
  - [Supported Clients](#supported-clients)
  - [Getting Started](#getting-started)
  - [Security](#security)
  - [Disclaimer](#disclaimer)

---

Screenshot

<p align="center">
  <img src="docs/screenshot.png" width="860">
</p>

<p align="center">
  MuseBridge provider · Codex CLI chatting through the proxy · the MuseBridge web dashboard
</p>

---

### Introduction

MuseBridge is an open-source API reverse proxy and management tool for Muse AI.

It provides a unified API access layer and a web-based dashboard for managing accounts, API keys, request logs, and usage statistics.

MuseBridge aims to simplify the integration of Muse AI with developer tools, including CC Switch and Codex CLI, while providing a lightweight and self-hostable management experience.

### Features

- **Muse AI API Proxy** — Forward API requests through a unified interface.
- **CC Switch Support** — Configure API access for compatible CC Switch workflows.
- **Codex CLI Support** — Connect Muse AI API access to the Codex CLI client.
- **Emulated Tool Calling** — OpenAI `tools` are described in the prompt and the marker-formatted reply is translated back into standard `tool_calls`. Best-effort (muse.ai has no native function calling); disable with `MUSE2API_TOOL_EMULATION=false`.
- **Account Management** — Manage configured accounts through a web dashboard.
- **In-Dashboard Browser Login** — open a login window from the dashboard; after signing in to muse.ai, all session cookies (HttpOnly included) are captured automatically and the account is created or refreshed. No more copying cookies from devtools. Requires a GUI on the server.
- **API Key Management** — Manage API keys used to access the proxy.
- **Request Monitoring** — View request counts, error rates, and response latency.
- **Model Usage Statistics** — Review available model usage statistics.
- **Request Logs** — Inspect recorded API requests for troubleshooting.
- **Web Dashboard** — Monitor service status and API activity through a clean interface.

Available features and compatibility depend on the current code version and deployment configuration.

### Supported Clients

#### CC Switch

MuseBridge can be used with compatible CC Switch workflows that support custom API configuration. Add a custom provider in the Codex section of CC Switch with:

- **Base URL**: `http://127.0.0.1:18610/v1`
- **API Key**: created in the MuseBridge dashboard (prefixed with `m2a-`)
- **Model**: `gpt-4o` or other aliases (see `GET /v1/models`)

#### Codex CLI

Add a custom provider to `~/.codex/config.toml`:

```toml
model = "gpt-4o"
model_provider = "musebridge"

[model_providers.musebridge]
name = "MuseBridge"
base_url = "http://127.0.0.1:18610/v1"
wire_api = "chat"
```

Then set the `OPENAI_API_KEY` environment variable to an API key created in the dashboard.

> **Note**: MuseBridge exposes OpenAI-compatible chat endpoints (streaming and non-streaming) plus image and video generation endpoints. Tool calling is emulated at the prompt level: `tools` from the request are injected into the prompt and the model's marker-formatted reply is converted back into standard `tool_calls`. This is best-effort — a reply that does not follow the format degrades to plain text (turn it off with `MUSE2API_TOOL_EMULATION=false`). Agentic tool calling in Codex CLI (running commands, editing files) works on top of this emulation; proprietary endpoints such as web search are out of scope. API protocols, authentication methods, and model parameters may vary between clients.

### Getting Started

**Requirements**: Python 3.10+, Google Chrome or Chromium (used by the browser driver), and a muse.ai account.

```bash
git clone git@github.com:Kerber0ss/muse-bridge.git
cd muse-bridge

python3 -m venv .venv && source .venv/bin/activate
pip install -e .

cp .env.example .env
# Edit .env and set MUSE2API_DRIVER=browser to talk to the real muse.ai
# (mock is an offline mode: no Chrome or account needed, canned replies)
sed -i 's/^MUSE2API_DRIVER=mock/MUSE2API_DRIVER=browser/' .env      # Linux (macOS: sed -i '')
python -m muse2api        # listens on http://127.0.0.1:18610 by default
```

Open [http://127.0.0.1:18610/dashboard](http://127.0.0.1:18610/dashboard) for the management dashboard. On first start an API key is generated and stored in `data/api_key`; it also acts as the dashboard admin key.

**Add a muse.ai account (recommended)**: click **Browser login** on the dashboard's Accounts page and sign in inside the window that opens — the cookies are captured automatically. Alternatively, collect `hatch_sess` and `hatch_native_auth_device` (required) plus `hatch_gw` / `hatch_vml` if present (optional — a fresh login may not issue them) from the developer tools of a signed-in browser and use the **Add account** button — fill in each value or paste the whole `Cookie` header. You can also import via the admin API:

```bash
curl http://127.0.0.1:18610/admin/accounts \
  -H "Authorization: Bearer <ADMIN_KEY>" -H "Content-Type: application/json" \
  -d '{"label":"acc1","cookies":{"hatch_sess":"...","hatch_gw":"...","hatch_vml":"...","hatch_native_auth_device":"..."}}'
```

**Test request**:

```bash
curl http://127.0.0.1:18610/v1/chat/completions \
  -H "Authorization: Bearer <API_KEY>" -H "Content-Type: application/json" \
  -d '{"model":"gpt-4o","messages":[{"role":"user","content":"Hello"}]}'
```

### Security

For a secure deployment:

- Never commit API keys, access tokens, or other sensitive credentials to a public repository.
- Protect access to the management dashboard.
- Avoid exposing the proxy service publicly without appropriate access controls.
- Use HTTPS in production environments.
- Review request logs for unusual activity.
- Keep upstream account credentials secure.
- Follow the applicable terms of service for Muse AI and related services.

### Disclaimer

MuseBridge is an open-source technical project intended for API integration, request forwarding, and service management.

- This project is not officially affiliated with or endorsed by Muse AI, CC Switch, or OpenAI unless explicitly stated otherwise.
- Users are responsible for ensuring that their use complies with applicable terms of service and laws.
- Users are responsible for their deployment, configuration, credential security, and usage.
- Do not use this project for unauthorized access or unlawful activities.

---

<p align="center">
  <strong>MuseBridge</strong><br>
  Bridging Muse AI with developer tools.
</p>

## License

[MIT](LICENSE)
