# Stage 5: Rate Limiting

## Concept

Explain briefly:

> Kong's **rate-limiting plugin** counts requests and enforces a cap — per consumer,
> per IP, or globally. Once a consumer hits the limit, Kong returns `429 Too Many
> Requests` and stops forwarding to your upstream.
>
> We're attaching this to the **route** level (rather than the service) to show you
> scoped plugin placement. The `local` policy uses in-memory counters on the DP node —
> simple and fast, no external datastore needed.

---

## Ask the User for a Rate Limit

Before creating the plugin, ask:

> How many requests per minute should we allow? I'd suggest **5** — low enough to
> trigger easily when testing, but feel free to change it.

Wait for their answer and save it as `JUNGLE_RATE_LIMIT`.

---

## Create the Rate Limiting Plugin

Check schema if needed:
```
mcp__Konnect__get_schema  →  operation_id: "create_plugin_with_route"
```

Create:
```json
{
  "control_plane_id": "<JUNGLE_CP_ID>",
  "route_id": "<JUNGLE_ROUTE_ID>",
  "name": "rate-limiting",
  "config": {
    "minute": <JUNGLE_RATE_LIMIT>,
    "policy": "local",
    "limit_by": "consumer"
  }
}
```

`limit_by: "consumer"` means authenticated consumers each get their own counter.
If you remove key-auth, Kong falls back to limiting by IP instead.

Show:
```
✓ Rate limiting plugin created
  Route:  jungle-route
  Limit:  <JUNGLE_RATE_LIMIT> requests/minute per consumer
  Policy: local (in-memory)
```

---

## Explain What Happens Next

> Wait a few seconds for Konnect to push the config update to your DP. In Stage 6
> we'll fire a burst of requests and watch the 429 kick in.
