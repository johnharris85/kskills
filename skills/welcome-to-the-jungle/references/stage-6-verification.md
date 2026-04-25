# Stage 6: End-to-End Verification

## Setup

Tell the user:

> Time to prove the whole thing works. We'll run three tests:
> 1. No key → 401 (auth is enforcing)
> 2. Valid key → 200 (proxy is working)
> 3. Burst past the rate limit → 429 (rate limiting is working)

Give the DP a few seconds to receive the latest config push, then proceed.

---

## Test 1: No Key → Expect 401

```bash
curl -s -w "\nHTTP %{http_code}\n" http://localhost:8000/demo/get
```

Assert the response is `HTTP 401`.

Show result:
```
Test 1: No API key
  Response: HTTP 401
  Body: {"message":"No API key found in request"}
  ✓ Key auth is enforcing correctly
```

If you get 200 instead, the key-auth plugin config may not have synced yet. Wait 10
seconds and retry before reporting an issue.

---

## Test 2: Valid Key → Expect 200

```bash
curl -s -w "\nHTTP %{http_code}\n" http://localhost:8000/demo/get \
  -H "apikey: <JUNGLE_API_KEY>"
```

Assert the response is `HTTP 200`.

Show result:
```
Test 2: Valid API key
  Response: HTTP 200
  ✓ Proxy is working — traffic reached httpbin.org
```

Point out something interesting from the httpbin response body — the `headers` section
will show that the `apikey` header was stripped (because `hide_credentials: true`) before
reaching the upstream. This is a nice teachable moment.

---

## Test 3: Rate Limit → Expect 429

Fire `JUNGLE_RATE_LIMIT + 1` requests in quick succession. Because `JUNGLE_RATE_LIMIT`
is a known number at this point, run individual curl calls via Bash rather than a
shell loop — this works identically on all platforms (macOS, Linux, Windows):

Run this curl command exactly `JUNGLE_RATE_LIMIT + 1` times via Bash, substituting
the real `JUNGLE_API_KEY` value:

```bash
curl -s -o /dev/null -w "HTTP %{http_code}\n" http://localhost:8000/demo/get \
  -H "apikey: <JUNGLE_API_KEY>"
```

Execute it once per line, back to back — e.g. if the rate limit is 5, run it 6 times.
The first N responses should be `HTTP 200`; the final one should be `HTTP 429`.

You should see the first N requests return `HTTP 200` and the last one return `HTTP 429`.

Show result:
```
Test 3: Rate limit burst (<JUNGLE_RATE_LIMIT> req/min)
  Request 1–<N>: HTTP 200
  Request <N+1>: HTTP 429 Too Many Requests
  ✓ Rate limiting is working
```

If all requests return 200, the config may not have synced. Wait 10 seconds and retry.

---

## Completion Banner

Once all three tests pass, print a completion summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Welcome to the Jungle — Complete! 🌴
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  [✓] 0  Account Setup
  [✓] 1  Control Plane
  [✓] 2  Data Plane (Docker)
  [✓] 3  Service + Route
  [✓] 4  Consumer + Key Auth
  [✓] 5  Rate Limiting
  [✓] 6  Verified End-to-End
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  What you built:
  • Control Plane:  jungle-cp        (<JUNGLE_CP_ID>)
  • Service:        jungle-service   (<JUNGLE_SERVICE_ID>)
  • Route:          /demo            (<JUNGLE_ROUTE_ID>)
  • Consumer:       jungle-user      (<JUNGLE_CONSUMER_ID>)
  • API Key:        <JUNGLE_API_KEY>
  • Rate limit:     <JUNGLE_RATE_LIMIT> req/min
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then close with a brief "what's next" to give them direction:

> You've got a working Kong Konnect gateway. From here you can:
> - Swap `https://httpbin.org` for your own upstream service
> - Add more plugins: request transformation, CORS, logging, OAuth 2.0
> - Create more consumers with different rate limits
> - Explore the Konnect UI to see all your entities in one place
>
> The Konnect MCP is attached — ask me to help with any of the above.
