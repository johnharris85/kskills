# Stage 3: Service + Route

## Concept

Explain briefly:

> Two core Kong concepts here:
>
> **Service** — represents the upstream API you want to proxy. It holds the URL of the
> real backend. When a request matches a route, Kong forwards it to the service's URL.
>
> **Route** — the matching rule that decides which requests get sent to which service.
> A route can match on paths, hosts, methods, or headers. We'll match on `/demo`.
>
> We're using **httpbin.org** as our test backend — it's a free HTTP testing service
> that echoes back request info, perfect for seeing what Kong is doing to your traffic.

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
