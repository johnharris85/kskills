---
name: welcome-to-the-jungle
description: >
  Use when a user wants to learn Kong Konnect by building a complete gateway setup from scratch.
  This is a hands-on guided tutorial that walks through every step: Konnect account creation,
  PAT token generation, control plane setup, local Docker data plane, service + route creation,
  consumer identity, API key authentication, and rate limiting — with live end-to-end curl
  verification at the end. Supports two paths: direct API calls via the Konnect MCP, or
  declarative config using decK (Kong's GitOps tool). Trigger on: "Konnect tutorial",
  "learn Kong", "get started with Konnect", "welcome to the jungle", "Kong quickstart",
  "try Konnect", "set up Konnect", "new to Kong", "Kong gateway tutorial",
  "I want to learn Konnect", "learn decK". Use whenever a user is starting fresh with
  Kong/Konnect and wants a guided experience, even if they don't explicitly say "tutorial".
---

# Welcome to the Jungle 🌴

You're guiding a user through a complete Kong Konnect setup — from zero account to a
secured, rate-limited API proxy with live verification.

**Before doing anything else, ask the user which path they want to take:**

---

## Mode Selection — Ask This First

Present the two options and wait for a choice before proceeding:

> **How would you like to build your Kong setup?**
>
> **A) API mode** — I'll call the Konnect API on your behalf at each step using the
> Konnect MCP. You see what gets created; I handle the calls.
>
> **B) decK mode** — Each step builds an incremental `jungle.yaml` file in Kong's
> declarative format, and I'll show you the `deck` commands to run. This mirrors a
> real GitOps workflow where your gateway config lives in version-controlled YAML.

Save their answer as `JUNGLE_MODE`:
- API answer → `JUNGLE_MODE = api`
- decK answer → `JUNGLE_MODE = deck`

---

## How This Skill Works

Work through the 7 stages in order. For each stage:
1. Print the **progress tracker** (updated format below), including the mode
2. Read the **stage's reference file** from the Stage Manifest (mode-specific for stages 3–6)
3. Follow the instructions in that file

Complete each stage fully before advancing.

---

## Progress Tracker

Print this at the start of every stage. Replace `[ ]` with `[✓]` for completed stages.
Show `[API]` or `[decK]` on the header line based on `JUNGLE_MODE`.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Welcome to the Jungle — Konnect Tutorial [MODE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  [ ] 0  Account Setup
  [ ] 1  Control Plane
  [ ] 2  Data Plane (Docker)
  [ ] 3  Service + Route
  [ ] 4  Consumer + Key Auth
  [ ] 5  Rate Limiting
  [ ] 6  Verify End-to-End
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Stage Manifest

Stages 0–2 are identical in both modes. Stages 3–6 depend on `JUNGLE_MODE`.

| Stage | API mode file | decK mode file |
|-------|--------------|----------------|
| 0 | `references/stage-0-account-setup.md` | same (+ deck section at end) |
| 1 | `references/stage-1-control-plane.md` | same (MCP always used for CP) |
| 2 | `references/stage-2-data-plane.md` | same (Docker always manual) |
| 3 | `references/stage-3-service-route.md` | `references/stage-3-deck-service-route.md` |
| 4 | `references/stage-4-consumer-keyauth.md` | `references/stage-4-deck-consumer-keyauth.md` |
| 5 | `references/stage-5-rate-limiting.md` | `references/stage-5-deck-rate-limiting.md` |
| 6 | `references/stage-6-verification.md` | `references/stage-6-deck-verification.md` |

---

## Rules

**State tracking.** Save these values in your working context:

| Variable | Set in | Value |
|----------|--------|-------|
| `JUNGLE_MODE` | Mode selection | `api` or `deck` |
| `JUNGLE_CP_NAME` | Stage 1 | Control Plane name (`jungle-cp`) |
| `JUNGLE_CP_ID` | Stage 1 | Control Plane ID |
| `JUNGLE_CLUSTER_ENDPOINT` | Stage 1 | e.g. `abc123.us.cp0.konghq.com` |
| `JUNGLE_TELEMETRY_ENDPOINT` | Stage 1 | e.g. `abc123.us.tp0.konghq.com` |
| `JUNGLE_CLUSTER_SERVER_NAME` | Stage 1 | same as cluster endpoint if absent |
| `JUNGLE_TELEMETRY_SERVER_NAME` | Stage 1 | same as telemetry endpoint if absent |
| `JUNGLE_SERVICE_ID` | Stage 3 (API) | Service ID |
| `JUNGLE_ROUTE_ID` | Stage 3 (API) | Route ID |
| `JUNGLE_CONSUMER_ID` | Stage 4 (API) | Consumer ID |
| `JUNGLE_API_KEY` | Stage 4 | API key value (API: from MCP; decK: user-chosen) |
| `JUNGLE_RATE_LIMIT` | Stage 5 | Requests per minute |
| `JUNGLE_KONNECT_REGION` | Stage 0 | e.g. `us`, `eu`, `au` |

**Control plane scoping (API mode).** Always pass `control_plane_id: JUNGLE_CP_ID`
on every configuration API call.

**Schema lookup (API mode).** If uncertain about parameter names, call
`mcp__Konnect__get_schema` with the operation ID first.

**Confirm after create (API mode).** After each MCP create call, call the corresponding
`list_*` or `get_*` to confirm, then show the ID to the user.

**Docker command assembly.** In stage 2, build the complete `docker run` command from
live MCP response values. Never leave placeholder text.

**Cert file handling.** Write cert/key to temp files via Bash before Docker run.

**decK file location.** In deck mode, always write and update `./jungle.yaml` in the
current working directory. Show the full file content after each update.

**decK sync pattern.** In deck mode: write the YAML → validate → show diff command
→ show sync command → wait for user to confirm they ran it before continuing.

**Tone.** Brief concept explanations before each action. Keep energy up — lab, not manual.

---

## MCP Quick Reference (API mode stages 1–4)

| Operation | Stage | Notes |
|-----------|-------|-------|
| `create_control_plane` | 1 | `cluster_type: "CLUSTER_TYPE_HYBRID"` |
| `create_dataplane_certificate` | 2 | Returns `cert` + `key` |
| `list_dataplane_nodes` | 2 | Poll until node appears after Docker run |
| `create_service` | 3 | `url: "https://httpbin.org"` |
| `create_route_with_service` | 3 | `paths: ["/demo"]`, `strip_path: true` |
| `create_consumer` | 4 | `username: "jungle-user"` |
| `create_plugin_with_service` | 4 | `name: "key-auth"` |
| `create_key_auth_with_consumer` | 4 | Capture the `key` field |
| `create_plugin_with_route` | 5 | `name: "rate-limiting"` |

---

Begin now. Ask the mode selection question, then load `references/stage-0-account-setup.md`.
