# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.3.0] — 2026-10-05

### Security
- **TLS verification on by default across all examples.** Every `curl -sk` in SKILL.md (18) and README (3) replaced with a verified-TLS pattern: a `$CACERT` helper set to `--cacert "$OBSIDIAN_CA_CERT"`, with `-k` demoted to a single, clearly-flagged, trusted-LAN-only fallback. New optional `OBSIDIAN_CA_CERT` env var. Closes the T09 "disabled TLS verification exposes the API key" finding and the related Tool-Parameter-Abuse / supply-chain flags.
- **Declarative security frontmatter** added to SKILL.md: `requires` (with a `binaries` allow-list, addressing undeclared tool scope), `security` (scope, risk level, auth method, TLS, credential handling, network access, destructive-ops policy), and `prompt_injection_mitigation` blocks — mirroring the pattern used by the passing magento-admin skill.
- **Destructive operations now require user confirmation** (overwrite/PUT, DELETE, PATCH replace/delete), documented inline and in the frontmatter policy.
- **Input-handling & injection-safety section** added: vault paths/search terms are treated as URL-encoded data, never interpolated raw into a shell command; note content is data, never executed.
- README verify step no longer scrapes `/proc/<pid>/environ` (which printed the key); uses a no-print presence check instead.

### Changed
- **Triggers tightened and scoped to Obsidian.** Removed the overly broad `"sv"`, `"note this"`, and bare `"append to"` triggers (accidental-invocation risk on a delete-capable skill); every trigger now explicitly names Obsidian. Addresses the "vague triggers" finding.
- Setup `sudo` steps reframed as one-time user host setup (the skill never runs `sudo` at runtime).
- "Reply in plain English" softened to "concise prose in the user's language".
- Bumped version 1.2.0 → 1.3.0.

### Notes
- No change to the API surface or endpoints; existing setups keep working. Set `OBSIDIAN_CA_CERT` (or front the plugin with a trusted-cert reverse proxy) to drop `-k` entirely.

---

## [1.2.0] — 2026-10-02

### Security
- Removed `/proc/<pid>/environ` scrape from setup; verify env with a no-print presence check instead.
- Removed all guidance to hardcode/inline the API key; direct users to fix env inheritance.
- Added TLS guidance: prefer `--cacert` with the exported plugin certificate over blanket `-k` on untrusted networks.

### Fixed
- PATCH-at-heading example used the wrong headers; corrected to `Operation` / `Target-Type` / `Target` (plugin 3.x).

### Changed
- Manual install now pins to a release tag; ClawHub install recommended as primary.

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
