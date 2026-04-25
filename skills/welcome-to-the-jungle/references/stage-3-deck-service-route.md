# Stage 3: Service + Route (decK mode)

## Concept

Explain the same concepts as the API path, then pivot to the decK approach:

> A **Service** represents the upstream API you want to proxy, and a **Route** is the
> matching rule that directs traffic to it. We're using **httpbin.org** as a live test
> backend.
>
> In decK mode, instead of calling the API directly, we describe these entities in
> `jungle.yaml` — Kong's declarative state file. When you run `deck gateway sync`,
> decK compares the file against what's already in Konnect and applies only the
> differences. It's idempotent: run it ten times and you get the same result.

---

## Write jungle.yaml

Write the following content to `./jungle.yaml` in the current working directory.
This is the complete file for this stage:

```yaml
_format_version: "3.0"
_konnect:
  control_plane_name: jungle-cp

services:
  - name: jungle-service
    url: https://httpbin.org
    routes:
      - name: jungle-route
        paths:
          - /demo
        strip_path: true
        protocols:
          - http
          - https
```

Show the user the full file content after writing it.

---

## Validate and Sync

Run each command via Bash and show the output inline.

**1. Validate:**
```bash
deck file validate jungle.yaml --konnect-compatibility
```
Expected: no errors. If there are errors, show them and fix the YAML before proceeding.

**2. Preview (dry run):**
```bash
deck gateway diff jungle.yaml
```
Expected: decK reports it will create `jungle-service` and `jungle-route`. Show the
output so the user can see what's about to change. Nothing is applied yet.

**3. Confirm then apply:**
Ask: *"Diff looks good — ready to apply? I'll run `deck gateway sync` now."*
Use `AskUserQuestion` if available. After confirmation:
```bash
deck gateway sync jungle.yaml
```
Show the sync output.

---

## Smoke Test

Run this via Bash to confirm Kong is proxying correctly (no auth yet):

```bash
curl -s -w "\nHTTP %{http_code}\n" http://localhost:8000/demo/get
```

Expected: `HTTP 200` with a JSON response from httpbin.

If you get 404 or connection refused:
- Check the DP is still running: `docker ps | grep kong-jungle-dp`
- Wait 5–10 seconds for Konnect to push config to the DP, then retry

---

## Bridge

> Your YAML file is the source of truth now. Every change from here will be a new
> version of this file — we'll keep adding to it. Next: lock it down with key auth.
