# Stage 4: Consumer + Key Auth (decK mode)

## Concept

Explain briefly:

> **Consumers** represent identities that call your API. **Key authentication** is a
> plugin that enforces API keys — requests without a valid key get rejected.
>
> In decK, consumers and their credentials live in the state file alongside services
> and routes. This means your entire access control list is in version control — you
> can add, rotate, or revoke keys through a pull request.

### About the API key value

In decK, you specify the key value directly in the YAML. Unlike the API path where
Konnect auto-generates one, here you choose it. Ask the user:

> What would you like to use as your API key for testing? You can use anything — I'd
> suggest something like `jungle-demo-key` to keep it memorable. (Use a strong random
> value in production.)

Save their choice as `JUNGLE_API_KEY`. If they don't have a preference, use
`jungle-demo-key`.

---

## Update jungle.yaml

Update `./jungle.yaml` to add:
1. The `key-auth` plugin scoped to the service (so all routes require a key)
2. A consumer with a key credential

Write the full updated file:

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

consumers:
  - username: jungle-user
    keyauth_credentials:
      - key: JUNGLE_API_KEY_VALUE
```

Replace `JUNGLE_API_KEY_VALUE` with the actual key the user chose.

Show the full file after writing. Highlight the two new sections: the `plugins` block
nested under the service, and the top-level `consumers` block.

---

## Explain the Placement

> Notice that `plugins` sits **inside** the service definition — that's decK's way of
> scoping a plugin to a specific service. You could also put `plugins` inside a route
> to scope it even more narrowly, or at the top level to make it global.
>
> `hide_credentials: true` strips the `apikey` header before forwarding to httpbin —
> your upstream never sees the key.

---

## Validate and Sync

```bash
# Validate
deck file validate jungle.yaml --konnect-compatibility

# Preview
deck gateway diff jungle.yaml
```

Expected diff output: decK shows it will create the `key-auth` plugin and a new consumer
`jungle-user` with one key credential.

```bash
# Apply
deck gateway sync jungle.yaml
```

**Wait for the user to confirm sync completed before continuing.**

---

## Verify Auth is Enforcing

Run via Bash to confirm unauthenticated requests are now rejected:

```bash
curl -s -w "\nHTTP %{http_code}\n" http://localhost:8000/demo/get
```

Expected: `HTTP 401` with `{"message":"No API key found in request"}`

If still 200, Konnect is still pushing the config — wait 5 seconds and retry.

---

## Bridge

> Your API is now locked down. API key: `<JUNGLE_API_KEY>` — remember this for the
> final verification. Next: rate limiting.
