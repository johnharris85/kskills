# Stage 3: Service + Route

## Concept

Explain before creating anything:

> Two core Kong concepts here:
>
> **Service** — represents the upstream API you want to proxy. It holds the target URL
> and connection settings (timeouts, retries, TLS). You can configure:
> - `connect_timeout`, `read_timeout`, `write_timeout` — how long Kong waits at each stage
> - `retries` — how many times to retry a failed upstream request (default: 5)
> - `path` — a base path to prepend to all upstream requests
>
> **Route** — the matching rule that decides which requests get sent to which service.
> Routes can match on `paths`, `hosts`, `methods`, `headers`, or `snis` (TLS SNI).
> When multiple routes could match an incoming request, Kong picks the most specific one —
> `/demo/admin` beats `/demo` beats `/`.
>
> The reason Service and Route are separate objects is flexibility: one service can have
> many routes. For example, you could have `/v1/*` and `/v2/*` both routing to the same
> upstream, with different plugins on each route. Or you could migrate traffic by adding
> a new route to a new service without touching the old one.
>
> We're using **httpbin.org** as our test backend — it's a free HTTP echo service that
> returns request details as JSON, so you can see exactly what Kong sends upstream
> (headers added, headers stripped, path rewriting, etc.).

---

## Step 3a: Create the Service

Check schema if needed:
```
mcp__Konnect__get_schema  →  operation_id: "create_service"
```

Create:
```json
{
  "control_plane_id": "<JUNGLE_CP_ID>",
  "name": "jungle-service",
  "url": "https://httpbin.org"
}
```

**Capture:** `id` → `JUNGLE_SERVICE_ID`

Show the user:
```
✓ Service created
  ID:   <JUNGLE_SERVICE_ID>
  Name: jungle-service
  URL:  https://httpbin.org
```

---

## Step 3b: Create the Route

Check schema if needed:
```
mcp__Konnect__get_schema  →  operation_id: "create_route_with_service"
```

Create:
```json
{
  "control_plane_id": "<JUNGLE_CP_ID>",
  "service_id": "<JUNGLE_SERVICE_ID>",
  "name": "jungle-route",
  "paths": ["/demo"],
  "strip_path": true,
  "protocols": ["http", "https"]
}
```

`strip_path: true` means Kong will remove `/demo` before forwarding to httpbin, so
`GET /demo/get` becomes `GET /get` at the upstream.

**Capture:** `id` → `JUNGLE_ROUTE_ID`

Show the user:
```
✓ Route created
  ID:    <JUNGLE_ROUTE_ID>
  Path:  /demo
  → Service: jungle-service (httpbin.org)
```

---

## Step 3c: Quick Smoke Test

Run this via Bash to confirm Kong is proxying correctly (no auth yet):

```bash
curl -s -w "\nHTTP %{http_code}\n" http://localhost:8000/demo/get
```

Expected: `HTTP 200` with a JSON response from httpbin showing request details.

If you get a 404 or connection refused:
- Check the DP is still running: `docker ps | grep kong-jungle-dp`
- Give Konnect 5-10 seconds to push the config to the DP, then retry

Show the user the response and explain:

> Your gateway is live and proxying traffic to httpbin.org. But anyone can hit it right
> now — no authentication required. Let's fix that next.
