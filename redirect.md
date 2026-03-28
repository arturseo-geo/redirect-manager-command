---
description: Manage Nginx 301 redirects on thegeolab.net VPS — add, list, verify, or remove redirects.
---

# Redirect Manager

Manage 301 redirects in the Nginx config for thegeolab.net.

## Commands

The user will say one of:
- `/redirect add /old-slug /new-slug` — add a new redirect
- `/redirect list` — show all active redirects
- `/redirect verify` — test all redirects return 301
- `/redirect remove /old-slug` — remove a redirect

## Config Location

```
VPS: root@100.87.191.9
File: /etc/nginx/sites-enabled/thegeolab
Section: Inside the HTTPS server block, before `location / {`
```

## Adding a Redirect

1. **Always use regex** to handle trailing slash variation:
```nginx
location ~ ^/old-slug/?$ { return 301 /new-slug/; }
```

2. **Never use exact match** (`location =`) — it fails on trailing slash mismatch.

3. **For date-prefixed patterns** (e.g. `/2026/03/old-path`):
```nginx
location ~ ^/2026/03/old-path { return 301 /new-path/; }
```

4. After adding, execute in order:
```bash
# Test config syntax
ssh root@100.87.191.9 "nginx -t"

# Reload (zero downtime)
ssh root@100.87.191.9 "systemctl reload nginx"

# Clear cache
ssh root@100.87.191.9 "rm -rf /var/cache/nginx/fastcgi/*"

# Verify redirect works (both with and without trailing slash)
curl -s -o /dev/null -w "%{http_code}" "https://thegeolab.net/old-slug"
curl -s -o /dev/null -w "%{http_code}" "https://thegeolab.net/old-slug/"
# Both must return 301
```

5. **If nginx -t fails:** Do NOT reload. Read the error, fix the config, test again.

## Listing Redirects

```bash
ssh root@100.87.191.9 "grep -E 'return 301' /etc/nginx/sites-enabled/thegeolab"
```

## Verifying All Redirects

Extract all redirect source paths and test each:
```bash
ssh root@100.87.191.9 "grep -oP '(?<=\^)(/[^/]+)' /etc/nginx/sites-enabled/thegeolab" | while read slug; do
  status=$(curl -s -o /dev/null -w "%{http_code}" "https://thegeolab.net${slug}/")
  echo "${status} | ${slug}"
done
```

All should return 301. Any returning 404 means the redirect was removed or the regex is broken.

## Removing a Redirect

Use Python to safely edit the Nginx config:
```python
# Read config, remove the matching location block, write back
# Always run nginx -t after editing
```

Never manually sed a redirect out — use a script that preserves the rest of the config.

## Safety Rules

- **Never edit the config without running `nginx -t` before reloading**
- **Never use `systemctl restart nginx`** — use `reload` for zero downtime
- **Always verify both `/slug` and `/slug/`** after adding
- **Document why** the redirect exists (in a comment above the location block if the reason isn't obvious)
- **Check for redirect chains** — if /a → /b and you add /b → /c, consolidate to /a → /c
