---
name: Bug report
about: Something is not working
title: "[BUG] "
labels: bug
assignees: ""
---

## Describe the bug

A clear description of what the bug is.

## Environment

- OpenClaw version:
- Obsidian version:
- Local REST API plugin version:
- OpenClaw host OS:
- Obsidian host OS:
- Skill version:

## Steps to reproduce

1.
2.
3.

## Expected behaviour

What you expected to happen.

## Actual behaviour

What actually happened. Include any error messages.

## API test result

Run this and paste the output (redact your API key):

```bash
curl -sk \
  -H "Authorization: Bearer $OBSIDIAN_API_KEY" \
  "$OBSIDIAN_URL/" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('status'), d.get('versions'))"
```

## Additional context

Anything else that might help.
