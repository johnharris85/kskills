# Stage 4: Consumer + Key Auth

## Concept

Explain briefly:

> Kong's **Consumer** object represents a user or application that will call your API.
> Consumers are identity, not authentication — they're the "who" that owns credentials.
>
> **Key Authentication** is a plugin that enforces API keys on a service or route.
> Here's how it works:
> 1. You enable the key-auth plugin on the service — Kong now rejects requests without a key
> 2. You create a consumer — the identity that will own the key
> 3. You create a key credential for that consumer — the actual secret they'll send
>
> We're enabling key-auth on the **service** level so it applies to all routes on the service.

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
