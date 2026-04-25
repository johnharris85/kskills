# Stage 5: Rate Limiting

## Concept

Explain before creating the plugin:

> Kong's **rate-limiting plugin** counts requests within a time window and returns
> `429 Too Many Requests` once the cap is reached.
>
> **Counters and the `policy` option:**
> - `local` — each DP node keeps its own counter in memory. Fast and zero-dependency,
>   but if you have multiple DP nodes, each tracks independently. A client hitting two
>   nodes could exceed your intended limit. Perfect for dev/single-node.
> - `redis` — all DP nodes share a counter via Redis. Accurate across a cluster but
>   requires a Redis instance. Use this in production multi-node setups.
> - `cluster` — uses the Kong database (DB-full mode only). Not applicable here.
>
> **What `limit_by` controls:**
> - `consumer` — each authenticated consumer gets their own counter. This is what we use.
> - `ip` — falls back to IP if there's no authenticated consumer.
> - `credential` — counts per API key rather than per consumer identity.
>
> **Response headers.** When rate limiting is active, Kong adds standard headers to
> every response so clients can self-throttle:
> ```
> RateLimit-Limit: 5
> RateLimit-Remaining: 3
> RateLimit-Reset: 47
> X-RateLimit-Limit-Minute: 5
> X-RateLimit-Remaining-Minute: 3
> ```
> Once the limit is hit, the `429` response also includes `Retry-After`.
>
> **Window behavior.** Kong uses a *fixed window* by default — the counter resets at
> the start of each minute (or hour/day depending on which window you configure).
> This means a burst at minute:59 and another at minute:00 can both be "within limit"
> even though they're close together. For stricter enforcement, the `sliding`
> window option is available in Kong EE.
>
> We're attaching this to the **route** level (rather than the service) to show you
> scoped placement. The route-level plugin only applies to `/demo` — if you added more
> routes later, they wouldn't be rate-limited unless you added the plugin there too.

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
