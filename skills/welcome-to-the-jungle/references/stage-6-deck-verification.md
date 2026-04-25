# Stage 6: End-to-End Verification (decK mode)

## Setup

Tell the user:

> Time to prove the whole thing works. Three tests:
> 1. No key → 401 (auth is enforcing)
> 2. Valid key → 200 (proxy is working, decK config applied correctly)
> 3. Burst past the rate limit → 429 (rate limiting is working)

The tests are identical to the API path — what changes is that the config
driving all of this came from a YAML file synced via `deck gateway sync`.

---

## Test 1: No Key → Expect 401

```bash
curl -s -w "\nHTTP %{http_code}\n" http://localhost:8000/demo/get
```

Assert: `HTTP 401`

Show:
```
Test 1: No API key
  Response: HTTP 401
  Body: {"message":"No API key found in request"}
  ✓ Key auth is enforcing correctly
```

---

## Test 2: Valid Key → Expect 200

```bash
curl -s -w "\nHTTP %{http_code}\n" http://localhost:8000/demo/get \
  -H "apikey: <JUNGLE_API_KEY>"
```

Assert: `HTTP 200`

Show:
```
Test 2: Valid API key
  Response: HTTP 200
  ✓ Proxy is working — traffic reached httpbin.org
```

Point out that the `apikey` header won't appear in httpbin's response `headers` section
because `hide_credentials: true` stripped it before the request reached the upstream.

---

## Test 3: Rate Limit → Expect 429

Because `JUNGLE_RATE_LIMIT` is a known number at this point, run individual curl
calls via Bash rather than a shell loop — this works on all platforms:

```bash
curl -s -o /dev/null -w "HTTP %{http_code}\n" http://localhost:8000/demo/get \
  -H "apikey: <JUNGLE_API_KEY>"
```

Execute it exactly `JUNGLE_RATE_LIMIT + 1` times back-to-back (e.g., 6 times if the
rate limit is 5), substituting the real `JUNGLE_API_KEY` value each time.

Expected: first N requests return `HTTP 200`, final request returns `HTTP 429`.

Show:
```
Test 3: Rate limit burst (<JUNGLE_RATE_LIMIT> req/min)
  Request 1–<N>: HTTP 200
  Request <N+1>: HTTP 429 Too Many Requests
  ✓ Rate limiting is working
```

---

## Bonus: Dump Current State

Since we're in decK mode, show the user how to capture what's currently in Konnect
back to a file — the inverse of sync:

```bash
deck gateway dump -o jungle-dump.yaml
```

The dump will include all entities with their server-assigned IDs. It's useful for
verifying what was actually applied, or for bootstrapping a decK file from an existing
environment.

---

## Completion Banner

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Welcome to the Jungle — Complete! 🌴  [decK mode]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  [✓] 0  Account Setup
  [✓] 1  Control Plane
  [✓] 2  Data Plane (Docker)
  [✓] 3  Service + Route
  [✓] 4  Consumer + Key Auth
  [✓] 5  Rate Limiting
  [✓] 6  Verified End-to-End
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  What you built (defined in ./jungle.yaml):
  • Control Plane:  jungle-cp
  • Service:        jungle-service → https://httpbin.org
  • Route:          /demo
  • Consumer:       jungle-user  (key: <JUNGLE_API_KEY>)
  • Plugins:        key-auth (service), rate-limiting (<JUNGLE_RATE_LIMIT>/min, route)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Close with "what's next":

> Your `jungle.yaml` is the source of truth. From here you can:
> - Replace `https://httpbin.org` with your own upstream
> - Add more plugins — just add them to the YAML and run `deck gateway sync`
> - Commit the file to git — you now have a full history of your gateway config
> - Use `deck gateway diff` any time to see what's drifted from your declared state
>
> The Konnect MCP is attached — ask me to help with any of the above.
