# GlmBridge 🌉

**Let the Glm Agent in your chat drive your Roblox Studio — directly, live, no extensions.**

GlmBridge connects an AI agent (GLM, or any AI that can send HTTP requests) straight into **your** Roblox Studio. The AI builds parts, writes game logic, spawns particles, takes screenshots to check its own work, and iterates with you in real time — while you watch it happen in the editor.


No browser extension. No special command language. Just a tiny Python relay, Roblox Studio's own built-in MCP server, and a URL.

---

##  What the AI can do once connected

Everything Studio's MCP server exposes — typically 28 tools, including:

| Tool | What it means |
|---|---|
| `execute_luau` | Run real Luau in your Place: build parts, tween, rotate, kill bricks, leaderstats, sounds, particles… (Edit / Server / Client contexts) |
| Game tree inspection | See every Instance, property, and script in your Place |
| Script read/write | View source, rewrite, create or delete scripts (e.g. drop a game-logic `Script` into `ServerScriptService`) |
| `screen_capture` | Grab a screenshot from Studio's viewport — the AI can *see* what it built and fix it |
| Play-test control | Start/stop play sessions, read output and errors |
| Creator Store | Search / insert free models, meshes, audio |

The live list (with parameter schemas) is always at `GET /tools`, or open the bridge URL in your browser for the status page.

**Real examples built end-to-end with GlmBridge:** a full obby with kill bricks / checkpoints / moving platforms, an incremental idle shop with 25 clickable upgrade pads + prestige economy, background music + buy sounds, right-click-buy-max, and a classic house with a working click-to-open door.

---

##  Quick start 

> **You need:** Windows 10/11, [Roblox Studio](https://www.roblox.com/create) installed, and Python 3 or higher.
### 0 Type chat.z.ai in google search and switch to agent mode (top left) And choose your favorite model (Glm 5.3 Flash recommended)

### 1 — Install Python 3 (skip if you have it)

1. Download from [python.org/downloads](https://www.python.org/downloads/)
2. Run the installer and **tick "Add python.exe to PATH"** — this matters.
3. Verify: open Command Prompt and run `python --version` (should print 3.x).

### 2 — Enable Studio's MCP server (one-time toggle)

1. Open **Roblox Studio** and load any Place.
2. Click **Assistant AI** in the top bar.
3. Click the **⋯** (top-right of the Assistant panel) → **Manage MCP Servers**.
4. Click **Enable Studio as MCP Server**.

### 3 — Run GlmBridge

1. Download **`glmbridge.zip`** from this repo and unzip it to desktop.
2. Double-click **`START_HERE.bat`**.
3. First run auto-downloads `cloudflared.exe` (~20 MB, one time). Then wait for:

   ```                  
   PUBLIC URL:  https://random-words.trycloudflare.com
   ```

### 4 — Connect the AI

Copy that URL, paste it into your AI chat together with the connect prompt below ⬇️ — done. Keep the relay window open while building (closing it kills the tunnel).

> The URL **rotates on every restart** of the bat file. That's normal — just paste the new one in chat.

---

## 💬 The prompt for GLM (edit how you want)

Send this to the AI in your chat, with your fresh URL pasted in:

```text
I have GlmBridge running and connected to my Roblox Studio.

Bridge URL: https://PASTE-YOUR-URL-HERE.trycloudflare.com

How to talk to my Studio (plain HTTP + JSON, no SDK needed):
- Check connection:  GET  <URL>/status
- List my tools:     GET  <URL>/tools
- Run anything:      POST <URL>/call  with body: {"tool": "...", "arguments": {...}, "timeout": 60}

The main tool is execute_luau — it runs Luau code inside my open Place:
{"tool": "execute_luau", "arguments": {"code": "print('hello from Studio')", "datamodel_type": "Edit"}, "timeout": 60}

Notes:
- datamodel_type "Edit" edits the Place itself; "Server" / "Client" run in play-test context.
- Other useful tools: screen_capture (screenshot Studio's viewport — use it to visually
  verify builds), script_read / script_write, game tree search, play-test control.
- First run print('hello') via execute_luau, then GET /tools, then confirm you're connected.

House rules:
- Build idempotently: destroy-and-recreate by a fixed name, so re-running your code is safe.
- Verify your own work (list what you created, read back source) before telling me it's done.
- Ask me before deleting or rewriting anything you didn't create yourself.
- Then ask me what I want to build first.




The AI composes Luau, pushes it through the bridge, then verifies — exactly like a developer sitting in your Studio.

---

<img width="783" height="794" alt="image" src="https://github.com/user-attachments/assets/768e9a5f-772e-42e6-b332-6b9a9ae906bc" />
<img width="648" height="128" alt="image" src="https://github.com/user-attachments/assets/6c78bc47-8b7c-4979-ae16-d5148d2b1aca" />
## 🔧 Manual usage (without an AI)

```bash
# status / tools
python glm_agent.py status  https://xxxx.trycloudflare.com
python glm_agent.py tools   https://xxxx.trycloudflare.com

# call a tool (example: build a part)
python glm_agent.py call https://xxxx.trycloudflare.com execute_luau \
  '{"code":"local p=Instance.new(\"Part\") p.Position=Vector3.new(0,5,0) p.Parent=workspace","datamodel_type":"Edit"}' 60
```

Or plain `curl`:

```bash
curl -s https://xxxx.trycloudflare.com/status
curl -s -X POST https://xxxx.trycloudflare.com/call \
  -H "Content-Type: application/json" \
  -d '{"tool":"execute_luau","arguments":{"code":"print(workspace:GetFullName())","datamodel_type":"Edit"},"timeout":30}'
```

## 📡 Endpoints

| Endpoint | Description |
|---|---|
| `GET /` | Status page (HTML, green dot = Studio connected) |
| `GET /status` | JSON state: MCP alive, tool count, tunnel, call stats |
| `GET /tools` | Live tool list with JSON schemas |
| `POST /call` | `{"tool": "...", "arguments": {...}, "timeout": 60}` |

## ⚙️ Configuration

| Env var | Default | Purpose |
|---|---|---|
| `GLM_RELAY_PORT` | `8510` | Local relay port |
| `GLM_STUDIO_MCP_PATH` | auto-discovered | Override path to `StudioMCP.exe` |

## 📦 Files in the zip

| File | What it is |
|---|---|
| `glmrelay.py` | The relay: HTTP ⇄ MCP, StudioMCP discovery, cloudflared tunnel manager (stdlib only) |
| `START_HERE.bat` | One-click launcher: finds Python, runs relay + tunnel in one window |
| `glm_agent.py` | Agent-side client — also handy manually (see above) |
| `README.md` | Short bundled docs |

---


## 🔒 Safety — read once

- **The bridge has no token auth (by design).** The tunnel URL is random and unguessable, but *anyone who has it can run code in your Studio.* Treat the URL like a password: don't post it publicly, and close the relay when you're done.
- Tool calls run **real Luau in your open Place**. Save your work before big experiments — and good to know: **Ctrl+Z in Studio undoes agent edits too.**
- Quick-tunnel URLs rotate every restart. Paste the new one each session; it's not a bug.
- Everything happens in your local Studio. **Publishing to Roblox is always manual** — the AI never touches your published game unless you press publish.

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| `Python 3 was not found` | Install Python, tick **Add python.exe to PATH**, run the bat again. |
| Status page says `StudioMCP.exe not found` | Open Roblox Studio once, then restart the bat. Studio must be a normal (per-user) install so it exists in `%LOCALAPPDATA%\Roblox\Versions`. |
| `0 tools` / prompt about MCP toggle | Redo step 2 of Quick start, then restart `START_HERE.bat`. |
| The URL stopped responding | Relay window closed / laptop slept → re-run the bat and paste the **new** URL. |
| `tunnel: failed to connect` | Network blocks QUIC — it auto-falls-back to http2. If it still fails, run `cloudflared tunnel --url http://localhost:8510 --protocol http2` in a second window and start the relay without `--tunnel`. |
| Calls fail with "no Studio instance connected" | Studio's proxy blipped; the relay auto-retries once. If persistent, re-open the Place. |
| Port 8510 busy | `set GLM_RELAY_PORT=8511` before running the bat, or edit `glmrelay.py`. |

## ❓ FAQ

**Does it work on Mac/Linux?**
Built for Windows (it auto-discovers `StudioMCP.exe` under `%LOCALAPPDATA%\Roblox\Versions`). Advanced users can try `GLM_STUDIO_MCP_PATH`, but Windows is the supported path.

**Can my friends use their own Studio with it?**
Yes — each person runs their own `glmbridge.zip` → `START_HERE.bat` on their PC and pastes their own URL into the chat. One relay per Studio.

**Is my Place safe?**
The AI edits your local Studio Place. Keep the URL private, save often, and remember Ctrl+Z works on agent edits.

**Which AIs work with it?**
Any AI that can make HTTP requests with a JSON body — GLM, and most other assistant sandboxes. The protocol is plain HTTP + JSON, nothing proprietary.

---

## 🙏 Credits(Zeroscript)

Built on Roblox Studio's **built-in MCP server** (`StudioMCP.exe`). Inspired by the ZeroScript browser-extension approach — GlmBridge replaces the extension with a small direct relay so any AI can drive Studio server-to-server.

## 📄 License

None — do whatever you want, no warranty.
