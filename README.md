# 💎 openclaw-obsidian-vault-skill

> An [OpenClaw](https://github.com/openclaw/openclaw) workspace skill that gives your AI agent full read/write access to any [Obsidian](https://obsidian.md) vault via the [Local REST API plugin](https://github.com/coddingtonbear/obsidian-local-rest-api).

Works on **any OS** where Obsidian Desktop runs (Windows, macOS, Linux).  
No extra CLI tools required — just `curl` and two environment variables.

---

## Why this skill?

The bundled OpenClaw `obsidian` skill requires `obsidian-cli`, which is macOS-only and depends on the Obsidian desktop URI handler. This skill takes a different approach: it talks directly to the **Local REST API plugin** running inside Obsidian, making it:

- ✅ **Cross-platform** — works wherever Obsidian runs, called from any Linux/macOS/WSL OpenClaw host
- ✅ **No extra binaries** — uses only `curl` and `python3` (standard on all platforms)
- ✅ **Full vault access** — read, write, append, patch, search, delete, list, run commands
- ✅ **Real-time** — reads the live vault, not a snapshot or cache
- ✅ **Simple setup** — two env vars and you are done

---

## What your agent can do

| Action | API Method |
|--------|-----------|
| List vault root or any subfolder | `GET /vault/` |
| Read any note | `GET /vault/PATH.md` |
| Create or overwrite a note | `PUT /vault/PATH.md` |
| Append to an existing note | `POST /vault/PATH.md` |
| Insert content at a specific heading | `PATCH /vault/PATH.md` |
| Delete a note | `DELETE /vault/PATH.md` |
| Full-text search across the vault | `POST /search/simple/` |
| Get the currently open file in Obsidian | `GET /active/` |
| List all available Obsidian commands | `GET /commands/` |
| Execute an Obsidian command by ID | `POST /commands/execute/` |

---

## Requirements

- **Obsidian Desktop** installed and running with a vault open
- **Local REST API plugin** installed and enabled in Obsidian  
  *(Settings → Community plugins → Browse → search "Local REST API")*
- **OpenClaw** `2026.3.x` or later (tested on `2026.4.x`)
- **curl** and **python3** available on the OpenClaw host (standard on all Linux/macOS)

---

## Quick Start

### Step 1 — Install the Local REST API plugin in Obsidian

1. Open Obsidian
2. Settings → Community plugins → turn off Safe Mode if needed → Browse
3. Search **"Local REST API"** → Install → Enable
4. Go to Settings → Local REST API
5. Copy your **API Key**
6. Note the **port** (default: `27124`)
7. Leave HTTPS enabled (recommended)

### Step 2 — Add env vars to your OpenClaw service

Open your OpenClaw systemd service file:

```bash
sudo nano /etc/systemd/system/openclaw.service
```

Add these two lines inside the `[Service]` block:

```ini
Environment=OBSIDIAN_URL=https://YOUR_OBSIDIAN_HOST:27124
Environment=OBSIDIAN_API_KEY=your_api_key_here
```

Replace:
- `YOUR_OBSIDIAN_HOST` — IP address or hostname of the machine running Obsidian (e.g. `192.168.1.50`)
- `your_api_key_here` — the key you copied from the plugin settings

Then reload and restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart openclaw.service
```

### Step 3 — Install the skill

**Via OpenClaw (once published to ClawHub):**
```bash
openclaw skills install obsidian-rest
```

**Or manually:**
```bash
git clone https://github.com/nj070574-gif/openclaw-obsidian-vault-skill.git
cp -r openclaw-obsidian-vault-skill/skill ~/.openclaw/workspace/skills/obsidian-rest
```

Start a new OpenClaw session. The skill should appear as:

```
✓ ready  💎 obsidian-rest  Read, write, search ...  openclaw-workspace
```

### Step 4 — Test it

Ask your agent:
> *"List my Obsidian vault"*

or

> *"Search my vault for meeting notes"*

---

## Verify env vars are set correctly

```bash
# Check env vars are in the live process
PID=$(pgrep -f "openclaw-gateway" | head -1)
cat /proc/$PID/environ | tr '\0' '\n' | grep "^OBSIDIAN"

# Test the API connection directly
curl -sk \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  "$OBSIDIAN_URL/" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print('OK — Obsidian', d['versions']['obsidian'], '| Plugin', d['versions']['self'])"
```

Expected: `OK — Obsidian 1.x.x | Plugin 3.x.x`

---

## Example agent interactions

| You say | Agent does |
|---------|-----------|
| *"Save this to Obsidian"* | Creates a note in the most appropriate folder |
| *"Note this in my vault"* | Saves current conversation content as a note |
| *"Find my notes on Docker"* | Searches vault full-text, returns matching files |
| *"Read my Infrastructure runbook"* | Searches, finds, and returns the note content |
| *"Append this to my daily log"* | Appends timestamped content to an existing note |
| *"List my Security folder"* | Lists all notes in the `Security/` subfolder |
| *"Update the setup guide under the Troubleshooting heading"* | Patches content at a specific heading |

---

## Configuration

| Variable | Example | Description |
|----------|---------|-------------|
| `OBSIDIAN_URL` | `https://192.168.1.50:27124` | Full URL including protocol and port |
| `OBSIDIAN_API_KEY` | `abc123...` | API key from the plugin settings |

**TLS note:** The plugin uses a self-signed certificate by default. Always use `curl -sk`.

---

## File structure

```
openclaw-obsidian-vault-skill/
├── skill/
│   └── SKILL.md          # The OpenClaw/ClawHub skill
├── README.md             # This file
├── CHANGELOG.md          # Version history
├── LICENSE               # MIT
└── .github/
    └── ISSUE_TEMPLATE/
        ├── bug_report.md
        └── feature_request.md
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `curl: (7) Failed to connect` | Obsidian not running or wrong host/port |
| `HTTP 401 Unauthorized` | Wrong API key — check plugin settings |
| SSL certificate error | Use `curl -sk` (self-signed cert) |
| `HTTP 404` on a file | URL-encode spaces (`%20`) and slashes (`%2F`) |
| Env var empty inside exec | Add `Environment=` lines to `openclaw.service`, daemon-reload + restart |
| Skill shows `△ needs setup` | `OBSIDIAN_URL` or `OBSIDIAN_API_KEY` not set |
| Obsidian on Windows, agent on Linux | Use Windows LAN IP; allow port `27124` in Windows Firewall |

### Windows Firewall (if Obsidian is on Windows)

```
Windows Defender Firewall → Advanced Settings → Inbound Rules → New Rule
→ Port → TCP → 27124 → Allow → All profiles → Name: "Obsidian Local REST API"
```

---

## Publishing to ClawHub

```bash
npm i -g clawhub
clawhub login
clawhub publish ./skill \
  --slug obsidian-rest \
  --name "Obsidian Local REST API" \
  --version 1.0.0 \
  --tags latest \
  --changelog "Initial release"
```

---

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgements

- [Adam Coddington](https://coddingtonbear.net/) — author of the [obsidian-local-rest-api](https://github.com/coddingtonbear/obsidian-local-rest-api) plugin
- [OpenClaw](https://github.com/openclaw/openclaw) — the open-source AI agent gateway this skill is built for
