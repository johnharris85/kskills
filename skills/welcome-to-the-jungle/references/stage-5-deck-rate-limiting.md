# Stage 5: Rate Limiting (decK mode)

## Concept

Explain briefly:

> The **rate-limiting plugin** counts requests within a time window and returns
> `429 Too Many Requests` once the limit is hit. We'll attach it to the route
> so it applies to all traffic hitting `/demo`, with counters tracked per consumer.
>
> With `policy: local`, each DP node tracks its own counter in memory — no external
> datastore needed. For multi-node deployments, you'd switch to `redis` policy so
> counts are shared. For this tutorial, local is perfect.

---

## Ask the User for a Rate Limit

> How many requests per minute should we allow? I'd suggest **5** — low enough to
> trigger easily during testing. Pick whatever you'd like to experiment with.

Save their answer as `JUNGLE_RATE_LIMIT`.

---

## Update jungle.yaml

Update `./jungle.yaml` — this is the final version of the file. Add a `plugins` block
**inside the route** definition (nested under the route, not the service):

```yaml
_format_version: "3.0"
_konnect:
  control_plane_name: jungle-cp

services:
  - name: jungle-service
    url: https://httpbin.org
    plugins:
      - name: key-auth
        config:
          key_names:
            - apikey
          hide_credentials: true
    routes:
      - name: jungle-route
        paths:
          - /demo
        strip_path: true
        protocols:
          - http
          - https
        plugins:
          - name: rate-limiting
            config:
              minute: JUNGLE_RATE_LIMIT_VALUE
              policy: local
              limit_by: consumer

consumers:
  - username: jungle-user
    keyauth_credentials:
      - key: JUNGLE_API_KEY_VALUE
```

Replace `JUNGLE_RATE_LIMIT_VALUE` with the user's chosen number and
`JUNGLE_API_KEY_VALUE` with the key from Stage 4.

Show the full file. Highlight the newly added `plugins` block inside the route —
it's nested differently from the key-auth plugin (which is in the service).
This is a good moment to explain that in decK, plugin scoping is reflected by
where in the hierarchy the plugin block appears.

---

## Validate and Sync

Run each command via Bash and show the output inline.

**1. Validate:**
```bash
deck file validate jungle.yaml --konnect-compatibility
```

**2. Preview:**
```bash
deck gateway diff jungle.yaml
```
Expected: decK reports creating the `rate-limiting` plugin scoped to `jungle-route`.
No other changes. Show the output.

**3. Confirm then apply:**
Ask: *"Diff looks good — ready to apply? I'll run `deck gateway sync` now."*
Use `AskUserQuestion` if available. After confirmation:
```bash
deck gateway sync jungle.yaml
```
Show the sync output.

---

## What You've Built in One File

Point out that the complete `jungle.yaml` now describes the entire gateway config:
service, route, two plugins, one consumer with credentials. One file, one sync command,
reproducible anywhere. This is the value of decK.

> This file is now your gateway's source of truth. You could `git init`, commit it, and
> have a full audit trail of every change to your API gateway config.
