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

Show these commands in order. Tell the user to run each one:

**1. Validate the file (local, no network call):**
```bash
deck file validate jungle.yaml --konnect-compatibility
```
Expected output: no errors.

**2. Preview what will change (dry run):**
```bash
deck gateway diff jungle.yaml
```
Expected: decK shows `creating service jungle-service` and `creating route jungle-route`.
This is just a preview — nothing has changed yet.

**3. Apply:**
```bash
deck gateway sync jungle.yaml
```
decK will confirm before applying if run interactively. The user should see the same
changes listed as in the diff, then a success message.

**Wait for the user to confirm sync completed successfully before continuing.**

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
