# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.1.0] — 2026-04-12

### Added
- **Critical pitfalls section** at top of SKILL.md — real-world lessons from deployment:
  - Trailing slash requirement on all directory endpoints (`/vault/` not `/vault`)
  - Correct root endpoint is `/` only — `/api/healthz` and similar do not exist and return 40400
  - Shell variable expansion caveat — `$OBSIDIAN_URL` may not inherit into exec child shells; test first
- **Output formatting rules** — explicit table of how to respond to each API result in plain English; agents must never dump raw JSON to the user
- **Improved troubleshooting table** — added `40400 Not Found` row with root cause (missing trailing slash / wrong endpoint) and fix
- **Improved search output** — search pattern now prints match count and context snippets
- **Improved status check** — self-test now prints `Auth: True/False` explicitly
- **Variable expansion test** — added `echo` check to verify env vars before curl

### Changed
- Troubleshooting table expanded with likely cause column
- API reference section headers clarified with trailing slash notes
- Workflow guide updated to include env var check as first step

---

## [1.0.0] — 2026-04-12

### Added
- Initial release
- Full vault access via Obsidian Local REST API plugin (v3.2.x)
- Operations: list, read, create (PUT), append (POST), patch at heading (PATCH), delete, search, active file, commands
- URL encoding helper and quick reference
- Workflow guide for agents (save, find, append, create folder)
- Common patterns with ready-to-run bash examples
- Troubleshooting table covering all common failure modes
- Windows Firewall setup guidance for cross-machine deployments
- ClawHub-compatible frontmatter with `requiredEnv` declarations
- Full setup guide for systemd-based OpenClaw installs
