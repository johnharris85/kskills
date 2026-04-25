# Stage 4: Consumer + Key Auth

## Concept

Explain before creating anything:

> Kong's **Consumer** object represents a user or application that calls your API.
> Consumers are *identity*, not authentication — they're the "who". The auth method
> (key-auth, OAuth, JWT, etc.) is separate and plugged in independently.
>
> This separation pays off at scale: you can have one consumer with multiple credential
> types, or swap auth methods without losing the consumer's history. It also means
> Kong can track analytics, apply rate limits, and enforce ACLs *per consumer* — not
> just per IP or globally.
>
> **Kong's plugin execution order** matters here. Plugins run in a defined priority:
> global plugins fire first, then service-level, then route-level, then consumer-level.
> Within a level, Kong uses the plugin's declared priority number. `key-auth` runs at
> priority 1250, which means it runs before most transformation plugins but after
> things like request-size-limiting. You generally don't need to manage this manually —
> just know that auth plugins run early in the pipeline.
>
> **Key Authentication** flow:
> 1. Enable key-auth on the service → Kong checks every request for a valid key
> 2. Create a consumer → the identity that will own credentials
> 3. Create a key credential for that consumer → what they actually send
>
> We're enabling key-auth at the **service** level so it applies to all routes.
> If you had a public route on the same service that shouldn't require auth, you'd
> use a plugin exception — but for this tutorial, service-level is exactly right.

Render this diagram when explaining the concept:

```
  Incoming request
         │
         ▼
  ┌──────────────────┐
  │    key-auth      │  ← service-level plugin
  │                  │
  │  missing key? ──────→ 401
  │  unknown key? ──────→ 401
  └────────┬─────────┘
           │ key valid → consumer identified (jungle-user)
           ▼
  ┌──────────────────┐
  │  rate-limiting   │  ← route-level plugin (added in next step)
  │                  │
  │  over limit? ───────→ 429
  └────────┬─────────┘
           │ within limit
           ▼
       upstream (httpbin.org)
```

---

## Step 4a: Create the Consumer

Check schema if needed:
```
mcp__Konnect__get_schema  →  operation_id: "create_consumer"
```

Create:
```json
{
  "control_plane_id": "<JUNGLE_CP_ID>",
  "username": "jungle-user"
}
```

**Capture:** `id` → `JUNGLE_CONSUMER_ID`

Show:
```
✓ Consumer created
  ID:       <JUNGLE_CONSUMER_ID>
  Username: jungle-user
```

---

## Step 4b: Enable Key Auth on the Service

This is what enforces the authentication. Without the plugin, Kong won't check for keys.

Check schema if needed:
```
mcp__Konnect__get_schema  →  operation_id: "create_plugin_with_service"
```

Create:
```json
{
  "control_plane_id": "<JUNGLE_CP_ID>",
  "service_id": "<JUNGLE_SERVICE_ID>",
  "name": "key-auth",
  "config": {
    "key_names": ["apikey"],
    "hide_credentials": true
  }
}
```

`key_names: ["apikey"]` means clients send their key in the `apikey` header (or query param).
`hide_credentials: true` strips the key header before forwarding to the upstream — your
backend never sees the API key.

Show:
```
✓ Key Auth plugin enabled on jungle-service
  Clients must now include: apikey: <their-key>
```

---

## Step 4c: Create an API Key for the Consumer

Check schema if needed:
```
mcp__Konnect__get_schema  →  operation_id: "create_key_auth_with_consumer"
```

Create:
```json
{
  "control_plane_id": "<JUNGLE_CP_ID>",
  "consumer_id": "<JUNGLE_CONSUMER_ID>"
}
```

Konnect will auto-generate a key. **Capture the `key` field from the response** →
`JUNGLE_API_KEY`

Show it prominently:
```
✓ API key created for jungle-user

  Your API key:  <JUNGLE_API_KEY>

  Keep this — you'll use it in Stage 6 to authenticate requests.
```

---

## Verify Auth is Enforced

Run a quick check via Bash to confirm unauthenticated requests are now rejected:

```bash
curl -s -w "\nHTTP %{http_code}\n" http://localhost:8000/demo/get
```

Expected: `HTTP 401` with `{"message":"No API key found in request"}`

If you still get 200, Konnect may still be pushing the config — wait 5 seconds and retry.

---

## Bridge to Stage 5

> Authentication is working — your API is locked down. Now let's add rate limiting to
> prevent any single consumer from hammering it.
