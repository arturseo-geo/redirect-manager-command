# /redirect — Claude Code Command

> Built at [The GEO Lab](https://thegeolab.net) — where we manage 12+ Nginx redirects across a growing content architecture.


Manage Nginx 301 redirects safely — add, list, verify, or remove.

## What it does

Replaces ad-hoc sed/python editing of Nginx configs with a structured command:

- **Add**: Creates regex-based redirects that handle trailing slash variation
- **List**: Shows all active redirects from the Nginx config
- **Verify**: Tests every redirect returns 301
- **Remove**: Safely removes a redirect without breaking the config

## Install

```bash
cp redirect.md ~/.claude/commands/redirect.md
```

## Usage

```
/redirect add /old-slug /new-slug
/redirect list
/redirect verify
/redirect remove /old-slug
```

## How redirects are implemented

Always uses regex to handle trailing slash:
```nginx
location ~ ^/old-slug/?$ { return 301 /new-slug/; }
```

Never uses exact match (`location =`) — it fails on trailing slash mismatch.

## Safety rules

- Always runs `nginx -t` before reloading
- Uses `systemctl reload` (not restart) for zero downtime
- Verifies both `/slug` and `/slug/` after adding
- Checks for redirect chains and consolidates

## Requirements

- SSH access to VPS with Nginx
- Nginx config at a known path

## Licence

[MIT](LICENSE)

---

A project by [The GEO Lab](https://thegeolab.net) — Generative Engine Optimisation research
