---
name: obsidian-rest
description: Read, write, search, create, append, patch, and manage notes in any Obsidian vault via the Local REST API plugin (Windows, macOS, or Linux). Use when asked to save notes, find information, read a document, append to a file, search the vault, write a runbook, update documentation, or manage Obsidian content. Triggers: "save this to Obsidian", "note this", "add to obsidian", "read my note on X", "find in obsidian", "update the runbook", "what did I save about X", "list my vault", "create a note", "append to", "sv".
homepage: https://github.com/nj070574-gif/openclaw-obsidian-vault-skill
metadata:
  {
    "openclaw":
      {
        "emoji": "💎",
        "requires":
          {
            "env": ["OBSIDIAN_URL", "OBSIDIAN_API_KEY"],
          },
        "config":
          {
            "requiredEnv": ["OBSIDIAN_URL", "OBSIDIAN_API_KEY"],
            "example": "Environment=OBSIDIAN_URL=https://YOUR_HOST:27124\nEnvironment=OBSIDIAN_API_KEY=your_api_key_here",
          },
      },
  }
---

# Obsidian Local REST API Skill

Control any Obsidian vault from OpenClaw using the
[Local REST API plugin](https://github.com/coddingtonbear/obsidian-local-rest-api).
Works on any OS where Obsidian Desktop runs (Windows, macOS, Linux).
No extra CLI tools needed — just curl.

---

## Prerequisites

1. **Obsidian Desktop** installed and running with a vault open.
2. **Local REST API plugin** installed and enabled in Obsidian:
   - Open Obsidian → Settings → Community plugins → Browse → search "Local REST API" → Install → Enable.
3. **API Key** copied from: Settings → Local REST API → API Key.
4. **Port** noted (default: 27124). HTTPS is strongly recommended.
5. **Env vars** set in your OpenClaw service (see Setup below).

---

## Setup

### 1. Add env vars to your OpenClaw systemd service

```bash
sudo nano /etc/systemd/system/openclaw.service
```

Add these two lines in the [Service] block:

```ini
Environment=OBSIDIAN_URL=https://YOUR_OBSIDIAN_HOST:27124
Environment=OBSIDIAN_API_KEY=your_api_key_here
```

Then reload and restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart openclaw.service
```

### 2. Verify env vars are live

```bash
PID=$(pgrep -f "openclaw-gateway" | head -1)
cat /proc/$PID/environ | tr "\0" "\n" | grep "^OBSIDIAN"
```

### 3. Test the connection

```bash
curl -sk \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  "$OBSIDIAN_URL/" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['versions']['obsidian'], d['versions']['self'])"
```

### 4. Install the skill

```bash
git clone https://github.com/nj070574-gif/openclaw-obsidian-vault-skill.git
cp -r openclaw-obsidian-vault-skill/skill ~/.openclaw/workspace/skills/obsidian-rest
```

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| OBSIDIAN_URL | Yes | Full base URL e.g. https://192.168.1.100:27124 |
| OBSIDIAN_API_KEY | Yes | API key from Obsidian Settings → Local REST API |

Always use curl -sk — the plugin uses a self-signed certificate by default.

---

## API Reference

All requests require: Authorization: Bearer $OBSIDIAN_API_KEY

### Check API status
```bash
curl -sk -H "Authorization: Bearer $OBSIDIAN_API_KEY" "$OBSIDIAN_URL/"
```

### List vault root
```bash
curl -sk -H "Authorization: Bearer $OBSIDIAN_API_KEY" "$OBSIDIAN_URL/vault/" \
  | python3 -c "import json,sys; [print(f) for f in sorted(json.load(sys.stdin)['files'])]"
```

### List a subfolder
```bash
curl -sk -H "Authorization: Bearer $OBSIDIAN_API_KEY" "$OBSIDIAN_URL/vault/My%20Folder/" \
  | python3 -c "import json,sys; [print(f) for f in json.load(sys.stdin)['files']]"
```

URL encoding: spaces = %20, forward slash within a segment = %2F

Helper to encode a path:
```bash
python3 -c "import urllib.parse; print(urllib.parse.quote('My Folder/My Note.md', safe=''))"
```

### Read a note
```bash
curl -sk -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  "$OBSIDIAN_URL/vault/PATH%2FTO%2FNOTE.md"
```

### Create or overwrite a note (PUT)
```bash
curl -sk -X PUT \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  -H "Content-Type: text/markdown" \
  --data-binary "# My Note Title

Content goes here." \
  "$OBSIDIAN_URL/vault/PATH%2FTO%2FNOTE.md"
```
Warning: PUT replaces the entire file. Use POST to append safely.

### Append to a note (POST)
```bash
curl -sk -X POST \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  -H "Content-Type: text/markdown" \
  --data-binary "

## New Section

Content to append." \
  "$OBSIDIAN_URL/vault/PATH%2FTO%2FNOTE.md"
```

### Patch at a heading (PATCH)
```bash
curl -sk -X PATCH \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  -H "Content-Type: text/markdown" \
  -H "Obsidian-API-Operation: append" \
  -H "Heading: My Section Heading" \
  --data-binary "Content to insert under the heading." \
  "$OBSIDIAN_URL/vault/PATH%2FTO%2FNOTE.md"
```
Operations: append | prepend | replace

### Delete a note
```bash
curl -sk -X DELETE \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  "$OBSIDIAN_URL/vault/PATH%2FTO%2FNOTE.md"
```

### Search vault
```bash
curl -sk -X POST \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  "$OBSIDIAN_URL/search/simple/?query=YOUR+TERM&contextLength=150" \
  | python3 -c "
import json, sys
for r in json.load(sys.stdin)[:5]:
    print('FILE:', r['filename'])
    for m in r.get('matches', [])[:2]:
        ctx = m.get('context', '').strip()
        if ctx: print('  ...', ctx[:120])
"
```

### Get currently active file
```bash
curl -sk -H "Authorization: Bearer $OBSIDIAN_API_KEY" "$OBSIDIAN_URL/active/"
```

### List Obsidian commands
```bash
curl -sk -H "Authorization: Bearer $OBSIDIAN_API_KEY" "$OBSIDIAN_URL/commands/" \
  | python3 -c "import json,sys; [print(c['id'], '|', c['name']) for c in json.load(sys.stdin).get('commands',[])]"
```

### Execute an Obsidian command
```bash
curl -sk -X POST \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"commandId": "editor:save-file"}' \
  "$OBSIDIAN_URL/commands/execute/"
```

---

## Workflow Guide

### Saving content
1. Pick the right folder from context
2. Choose a descriptive hyphenated filename e.g. Setup-Guide-2026-04-12.md
3. Check file exists: GET /vault/PATH.md — 404 means safe to create
4. PUT to create, POST to append to existing
5. Confirm: "Saved to Infrastructure/Setup-Guide-2026-04-12.md"

### Finding a note
1. Search with /search/simple/?query=TERM
2. If multiple results, list and ask which to open
3. GET the file and return content or summary

### Updating
1. Read existing file first to understand structure
2. POST to append, or PATCH with Heading for targeted insert
3. Confirm what changed

### New folders
Folders are created automatically when you PUT a file into a non-existent path.

---

## Common Patterns

### Save a runbook
```bash
NOTE_PATH="Infrastructure%2FRunbook-$(date +%Y-%m-%d).md"
curl -sk -X PUT \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  -H "Content-Type: text/markdown" \
  --data-binary "# Runbook $(date +%Y-%m-%d)

## Steps

1. Step one
2. Step two
" \
  "$OBSIDIAN_URL/vault/$NOTE_PATH"
```

### Append a timestamped log entry
```bash
curl -sk -X POST \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  -H "Content-Type: text/markdown" \
  --data-binary "
- $(date '+%Y-%m-%d %H:%M') — Log entry here" \
  "$OBSIDIAN_URL/vault/Daily%2FLog.md"
```

---

## URL Encoding Reference

| Character | Encoded |
|-----------|---------|
| Space     | %20     |
| /         | %2F     |
| #         | %23     |
| &         | %26     |
| +         | %2B     |

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| curl: (7) Failed to connect | Obsidian not running or wrong URL/port |
| HTTP 401 Unauthorized | Wrong API key |
| SSL certificate error | Use curl -sk |
| HTTP 404 on a file | URL-encode spaces and slashes; verify path |
| Env var empty in exec | Add Environment= lines to openclaw.service, daemon-reload + restart |
| Skill shows needs setup | OBSIDIAN_URL or OBSIDIAN_API_KEY not set |

---

## Plugin Information

- Plugin: Obsidian Local REST API by Adam Coddington
- Repo: https://github.com/coddingtonbear/obsidian-local-rest-api
- Default port: 27124
- Protocol: HTTPS (self-signed) or HTTP
- Obsidian minimum version: 0.12.0
- Plugin version tested: 3.2.0
