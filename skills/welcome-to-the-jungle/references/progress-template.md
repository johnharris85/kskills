# Progress HTML Companion — Template & Update Guide

## When to use this file

Read this file when SKILL.md tells you to generate or update `jungle-progress.html`.
Generate the file immediately after the user selects a mode; update it (full rewrite with
Write tool) after each step completes.

---

## HTML Template

Use the template below verbatim. Substitute real values where indicated.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Welcome to the Jungle — Kong Konnect Tutorial</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      background: #0f0f1a;
      color: #c8c8d4;
      font-family: 'SF Mono', 'Fira Code', 'Consolas', monospace;
      font-size: 14px;
      line-height: 1.6;
      padding: 32px;
      max-width: 860px;
      margin: 0 auto;
    }
    header { margin-bottom: 32px; border-bottom: 1px solid #1e1e32; padding-bottom: 16px; }
    header h1 { color: #00b388; font-size: 1.4rem; letter-spacing: 0.04em; }
    header .subtitle { color: #555; font-size: 0.82rem; margin-top: 4px; }
    .mode-badge {
      display: inline-block;
      background: #0d1f18;
      color: #00b388;
      border: 1px solid #00b388;
      border-radius: 3px;
      padding: 2px 10px;
      font-size: 0.75rem;
      margin-top: 10px;
    }
    .progress { margin-bottom: 32px; }
    .section-title {
      color: #444;
      font-size: 0.72rem;
      letter-spacing: 0.12em;
      text-transform: uppercase;
      margin-bottom: 12px;
    }
    .step {
      display: flex;
      align-items: center;
      padding: 7px 12px;
      margin: 2px 0;
      border-radius: 4px;
      gap: 10px;
      border-left: 2px solid transparent;
    }
    .step.done { color: #00b388; }
    .step.done::before { content: "✓"; width: 14px; flex-shrink: 0; }
    .step.current {
      background: #13132a;
      color: #ffffff;
      border-left-color: #00b388;
    }
    .step.current::before { content: "→"; width: 14px; flex-shrink: 0; color: #00b388; }
    .step.pending { color: #333; }
    .step.pending::before { content: "○"; width: 14px; flex-shrink: 0; }
    .credentials { margin-bottom: 32px; }
    table { width: 100%; border-collapse: collapse; }
    td {
      padding: 6px 12px;
      border-bottom: 1px solid #1a1a2e;
      vertical-align: top;
      font-size: 0.88rem;
    }
    td:first-child { color: #555; width: 36%; white-space: nowrap; }
    td.value { color: #e0e0e0; word-break: break-all; }
    td.empty { color: #2a2a3e; font-style: italic; }
    .diagram-section { margin-bottom: 32px; }
    pre.diagram {
      background: #090910;
      border: 1px solid #1a1a2e;
      border-radius: 6px;
      padding: 20px 24px;
      font-size: 12px;
      line-height: 1.5;
      overflow-x: auto;
      color: #8080a0;
    }
    footer {
      color: #2a2a3e;
      font-size: 0.72rem;
      border-top: 1px solid #1a1a2e;
      padding-top: 12px;
      margin-top: 32px;
    }
  </style>
</head>
<body>

<header>
  <h1>🌴 Welcome to the Jungle</h1>
  <div class="subtitle">Kong Konnect Guided Tutorial</div>
  <span class="mode-badge">MODE: [API or decK]</span>
</header>

<section class="progress">
  <div class="section-title">Progress</div>
  <!-- Each step: class="step done", "step current", or "step pending" -->
  <div class="step [CLASS]">Account Setup</div>
  <div class="step [CLASS]">Control Plane</div>
  <div class="step [CLASS]">Data Plane</div>
  <div class="step [CLASS]">Service + Route</div>
  <div class="step [CLASS]">Consumer + Key Auth</div>
  <div class="step [CLASS]">Rate Limiting</div>
  <div class="step [CLASS]">Verify End-to-End</div>
</section>

<section class="credentials">
  <div class="section-title">Credentials &amp; IDs</div>
  <table>
    <tr><td>Konnect Region</td><td class="[value or empty]">[JUNGLE_KONNECT_REGION or —]</td></tr>
    <tr><td>Control Plane ID</td><td class="[value or empty]">[JUNGLE_CP_ID or —]</td></tr>
    <tr><td>Cluster Endpoint</td><td class="[value or empty]">[JUNGLE_CLUSTER_ENDPOINT or —]</td></tr>
    <tr><td>Telemetry Endpoint</td><td class="[value or empty]">[JUNGLE_TELEMETRY_ENDPOINT or —]</td></tr>
    <!-- API mode only: -->
    <tr><td>Service ID</td><td class="[value or empty]">[JUNGLE_SERVICE_ID or —]</td></tr>
    <tr><td>Route ID</td><td class="[value or empty]">[JUNGLE_ROUTE_ID or —]</td></tr>
    <tr><td>Consumer ID</td><td class="[value or empty]">[JUNGLE_CONSUMER_ID or —]</td></tr>
    <!-- decK mode only: replace the three IDs above with: -->
    <!-- <tr><td>Config file</td><td class="value">./jungle.yaml</td></tr> -->
    <tr><td>API Key</td><td class="[value or empty]">[JUNGLE_API_KEY or —]</td></tr>
    <tr><td>Rate Limit</td><td class="[value or empty]">[JUNGLE_RATE_LIMIT req/min or —]</td></tr>
  </table>
</section>

<section class="diagram-section">
  <div class="section-title">Architecture</div>
  <pre class="diagram">
[DIAGRAM — see per-step guide below]
  </pre>
</section>

<footer>Last updated: [ISO timestamp, e.g. 2026-04-24 14:32]</footer>

</body>
</html>
```

---

## Per-Step Update Guide

After each step completes, update the step classes and credentials table, and swap in
the correct diagram from the list below.

### Initial generation (after mode selection, before Account Setup)

- All steps: `pending`
- All credential cells: `empty` with `—`
- No diagram block needed — omit the diagram section entirely or show a placeholder:
  ```
  Diagram will appear after the Control Plane is created.
  ```

### After Account Setup

- Account Setup: `done` | Control Plane: `current` | rest: `pending`
- Credentials: populate `Konnect Region` if known; rest `empty`
- Diagram: omit or keep placeholder

### After Control Plane

- Account Setup + Control Plane: `done` | Data Plane: `current` | rest: `pending`
- Credentials: populate CP ID, Cluster Endpoint, Telemetry Endpoint
- Diagram:
```
  ┌──────────────────────────────────────────┐
  │            Konnect (Cloud)               │
  │  ┌────────────────────────────────────┐  │
  │  │        Control Plane ✓             │  │
  │  │  services · routes · plugins       │  │
  │  └─────────────────┬──────────────────┘  │
  └────────────────────│─────────────────────┘
                       │ config sync (mTLS)
                       ▼
  ┌──────────────────────────────────────────┐
  │          Your machine (Docker)           │
  │  ┌────────────────────────────────────┐  │
  │  │         Data Plane  (starting...)  │  │
  │  └────────────────────────────────────┘  │
  └──────────────────────────────────────────┘
```

### After Data Plane

- Account Setup + Control Plane + Data Plane: `done` | Service + Route: `current`
- Credentials: unchanged
- Diagram: same as above but DP shows `connected ✓`

### After Service + Route

- ...through Data Plane: `done` | Consumer + Key Auth: `current`
- Credentials (API mode): populate Service ID, Route ID; (decK mode): show `./jungle.yaml`
- Diagram:
```
  ┌───────────┐  /demo   ┌────────────┐   ┌──────────────┐
  │  Client   │ ───────→ │   Route ✓  │ → │  Service ✓   │
  └───────────┘          │ strip path │   │ httpbin.org  │
                         └────────────┘   └──────┬───────┘
                                                 │
                                          GET /get (upstream)
```

### After Consumer + Key Auth

- ...through Service + Route: `done` | Rate Limiting: `current`
- Credentials (API mode): populate Consumer ID, API Key; (decK mode): populate API Key
- Diagram:
```
  Incoming request
         │
         ▼
  ┌──────────────────┐
  │    key-auth ✓    │  ← service-level
  │  no/bad key → 401│
  └────────┬─────────┘
           │ consumer identified
           ▼
       upstream (httpbin.org)
```

### After Rate Limiting

- ...through Consumer + Key Auth: `done` | Verify End-to-End: `current`
- Credentials: populate Rate Limit
- Diagram:
```
  Incoming request
         │
         ▼
  ┌──────────────────┐
  │    key-auth ✓    │  ← service-level
  │  no/bad key → 401│
  └────────┬─────────┘
           │ consumer identified
           ▼
  ┌──────────────────┐
  │  rate-limiting ✓ │  ← route-level
  │  over limit → 429│
  └────────┬─────────┘
           │ within limit
           ▼
       upstream (httpbin.org)
```

### After Verify End-to-End (Complete)

- All steps: `done` | no `current` step
- All credentials populated
- Diagram: same as Rate Limiting step but add a "✓ 401 / ✓ 200 / ✓ 429 verified" note below:
```
  Incoming request
         │
         ▼
  ┌──────────────────┐
  │    key-auth ✓    │  → 401 (no key) ✓
  └────────┬─────────┘
           │
           ▼
  ┌──────────────────┐
  │  rate-limiting ✓ │  → 429 (over limit) ✓
  └────────┬─────────┘
           │
           ▼
     upstream ✓  (200 with valid key) ✓
```
