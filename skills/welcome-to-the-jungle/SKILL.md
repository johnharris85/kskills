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

**Before doing anything else, check for a saved session, then ask the user which path they want to take:**

---

## Mode Selection — Ask This First

**If your harness provides an interactive question tool** (e.g. `AskUserQuestion` in
Claude Code, or a similar picker), use it here — it renders a much cleaner experience
than markdown text. Otherwise, present the options as formatted text and wait for a reply.

Ask the user to choose:

> **How would you like to build your Kong setup?**
>
> **API mode** — I'll call the Konnect API on your behalf at each step using the
> Konnect MCP. You see what gets created; I handle the calls.
>
> **decK mode** — Each step builds an incremental `jungle.yaml` file in Kong's
> declarative format, and I'll show you the `deck` commands to run. This mirrors a
> real GitOps workflow where your gateway config lives in version-controlled YAML.

Save their answer as `JUNGLE_MODE`:
- API answer → `JUNGLE_MODE = api`
- decK answer → `JUNGLE_MODE = deck`

---

## How This Skill Works

**At session start, before anything else:**
1. Attempt to Read `./jungle-state.json` (see State Persistence rule below)
2. If found, offer to resume — skip the mode question if resuming
3. If not found, ask the mode selection question, then proceed

**For each step:**
1. Print the **progress tracker** (updated format below), including the mode
2. Read the **step's reference file** from the Stage Manifest (mode-specific for steps 3–6)
3. Follow the instructions in that file
4. After completing the step: update `jungle-state.json` and `jungle-progress.html`

Complete each step fully before advancing.

---

## Progress Tracker

Print this at the start of every step. Replace `[ ]` with `[✓]` for completed steps.
Show `[API]` or `[decK]` on the header line based on `JUNGLE_MODE`.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Welcome to the Jungle — Konnect Tutorial [MODE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  [ ] Account Setup
  [ ] Control Plane
  [ ] Data Plane
  [ ] Service + Route
  [ ] Consumer + Key Auth
  [ ] Rate Limiting
  [ ] Verify End-to-End
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
| `JUNGLE_NUDGE_COUNT` | Any stage | Times user has declined returning to tutorial; start at 0 |

**Control plane scoping (API mode).** Always pass `control_plane_id: JUNGLE_CP_ID`
on every configuration API call.

**Schema lookup (API mode).** If uncertain about parameter names, call
`mcp__Konnect__get_schema` with the operation ID first.

**Confirm after create (API mode).** After each MCP create call, call the corresponding
`list_*` or `get_*` to confirm, then show the ID to the user.

**Docker command assembly.** In stage 2, build the complete `docker run` command from
live MCP response values. Never leave placeholder text.

**Cert file handling.** Use the Write tool to write cert and key to `./certs/` — not a Bash heredoc. This is cross-platform safe.

**decK file location.** In deck mode, always write and update `./jungle.yaml` in the
current working directory. Show the full file content after each update.

**decK execution pattern.** In deck mode, after writing or updating `jungle.yaml`:
1. Run `deck file validate jungle.yaml --konnect-compatibility` via Bash — show the output
2. If valid, run `deck gateway diff jungle.yaml` via Bash — show the diff output
3. Ask: *"Ready to apply? I'll run `deck gateway sync` now."* (use `AskUserQuestion` if available)
4. Only run `deck gateway sync jungle.yaml` via Bash after explicit confirmation — show output

Do NOT show commands and ask the user to run them. Claude runs validate and diff directly;
sync is the only step that pauses for confirmation (it's a write operation).

**State persistence.** At skill start, before anything else, attempt to Read
`./jungle-state.json`.
- If found: load it, restore all JUNGLE_* variables, and offer to resume:
  > "I found a saved session — you completed: [list done steps]. Want to resume
  > from [current_step], or start fresh?"
  - Resume: skip completed steps, continue from `current_step`
  - Fresh: overwrite the file and begin from the top
- After setting any JUNGLE_* variable: rewrite `./jungle-state.json` with the Write tool.
  Use this structure:
  ```json
  {
    "mode": "api",
    "completed_steps": ["Account Setup", "Control Plane"],
    "current_step": "Data Plane",
    "variables": {
      "JUNGLE_MODE": "api",
      "JUNGLE_KONNECT_REGION": "us",
      "JUNGLE_CP_ID": "...",
      "JUNGLE_CLUSTER_ENDPOINT": "...",
      "JUNGLE_TELEMETRY_ENDPOINT": "...",
      "JUNGLE_CLUSTER_SERVER_NAME": "...",
      "JUNGLE_TELEMETRY_SERVER_NAME": "...",
      "JUNGLE_SERVICE_ID": "...",
      "JUNGLE_ROUTE_ID": "...",
      "JUNGLE_CONSUMER_ID": "...",
      "JUNGLE_API_KEY": "...",
      "JUNGLE_RATE_LIMIT": 5,
      "JUNGLE_CERTS_DIR": "..."
    }
  }
  ```
  Omit variables not yet set rather than leaving them blank.

**Visual companion.** Maintain `jungle-progress.html` in the working directory.
- Read `references/progress-template.md` for the HTML template and per-step update guide.
- Generate the file (Write tool) immediately after the user selects a mode.
- Update it (Write tool, full rewrite) after each step completes.
- Tell the user once, after generating: *"I've created `jungle-progress.html` — open it
  in your browser to track progress and see your credentials as we go."*
  Don't mention it again unless they ask.

**Interactive inputs.** Whenever you need a choice from the user (mode selection, rate
limit value, etc.), prefer an interactive question tool if your harness provides one
(e.g. `AskUserQuestion` in Claude Code). This renders a cleaner picker than inline text.
If no such tool is available, fall back to a formatted lettered list.

**Terminal visuals.** Use ASCII box-drawing diagrams to illustrate architecture and
request flows — they're far more memorable than prose. Prefer Unicode box characters
(`┌ ─ ┐ │ └ ┘ ├ ┤ ┬ ┴ → ↓`) for clean lines. Each stage reference file includes a
diagram template; render it as a fenced code block so it's monospace-aligned.

**Tone.** Brief concept explanations before each action. Keep energy up — lab, not manual.

---

## Handling Questions Beyond the Tutorial

Users will sometimes ask Kong/Konnect questions that go beyond the current tutorial step.
That's great — curiosity is good. Answer them, but keep the tutorial moving.

**When a user asks something off-topic:**

1. Answer the question genuinely. Use `WebFetch` on `https://developer.konghq.com` to find
   accurate documentation rather than guessing. Summarize the relevant part for them.

2. After answering, if `JUNGLE_NUDGE_COUNT < 2`, add a gentle nudge like:
   > "Happy to keep exploring this — or we can pick up where we left off whenever you're ready."
   Keep it low-pressure. One sentence, at the end of your reply.

3. If the user declines the nudge or says they want to keep exploring, increment
   `JUNGLE_NUDGE_COUNT`. Once `JUNGLE_NUDGE_COUNT >= 2`, **stop nudging entirely** and
   just answer questions. Never mention getting back to the tutorial again unless they
   bring it up first.

4. When they are ready to return, resume from whatever stage they left off at.

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
