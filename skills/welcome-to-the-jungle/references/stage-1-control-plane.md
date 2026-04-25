# Stage 1: Control Plane

## Concept

Explain this before creating anything:

> A **Control Plane (CP)** is the brain of your Kong setup. It stores all your gateway
> configuration — services, routes, plugins, consumers — and distributes it to data
> plane nodes. No traffic flows through the CP itself; it's purely configuration.
>
> We're creating a **hybrid** CP (`CLUSTER_TYPE_HYBRID`). In hybrid mode, data planes
> reach out to Konnect to pull their config — they initiate the connection, so they work
> even if they're behind a NAT or firewall. The alternative types are:
> - `CLUSTER_TYPE_K8S_INGRESS_CONTROLLER` — for Kong Ingress Controller on Kubernetes
> - `CLUSTER_TYPE_SERVERLESS` — fully managed DPs, no Docker required (Konnect-hosted)
>
> Hybrid gives you the most control: your DP runs on your infrastructure, but Konnect
> manages the config. If Konnect ever has an outage, your DP keeps running on its last
> known good config — it doesn't need a live connection to serve traffic.
>
> The CP response includes two endpoints you'll need in Stage 2:
> - **cluster_endpoint** — where DPs connect to receive config updates (port 443)
> - **cluster_telemetry_endpoint** — where DPs send analytics/vitals data
>
> These are unique to your CP. If you create multiple CPs (e.g. prod vs staging),
> each gets its own set of endpoints, so DPs can't accidentally join the wrong cluster.

Render this diagram when explaining the concept:

```
  ┌──────────────────────────────────────────┐
  │            Konnect (Cloud)               │
  │  ┌────────────────────────────────────┐  │
  │  │          Control Plane             │  │
  │  │  services · routes · plugins       │  │
  │  │  consumers · certificates          │  │
  │  └─────────────────┬──────────────────┘  │
  └────────────────────│────────────────────-┘
                       │ config sync (mTLS)
                       ▼
  ┌──────────────────────────────────────────┐
  │          Your machine (Docker)           │
  │  ┌────────────────────────────────────┐  │
  │  │           Data Plane               │  │
  │  │       Kong Gateway process         │  │
  │  │   enforces rules · routes traffic  │  │
  │  └────────────────────────────────────┘  │
  │        ↑ requests       ↓ responses      │
  └──────────────────────────────────────────┘
```

---

## Create the Control Plane

Call the MCP to create the CP. Check the schema first if needed:

```
mcp__Konnect__get_schema  →  operation_id: "create_control_plane"
```

Then create it:

```json
{
  "name": "jungle-cp",
  "cluster_type": "CLUSTER_TYPE_HYBRID",
  "description": "Jungle tutorial control plane"
}
```

**From the response, capture and save:**
- `id` → `JUNGLE_CP_ID`
- `config.cluster_endpoint` → `JUNGLE_CLUSTER_ENDPOINT`
  (strip any `https://` prefix if present — just the hostname)
- `config.cluster_telemetry_endpoint` → `JUNGLE_TELEMETRY_ENDPOINT`
  (strip prefix, just hostname)
- `config.cluster_server_name` → `JUNGLE_CLUSTER_SERVER_NAME`
  (if absent, use same value as `JUNGLE_CLUSTER_ENDPOINT`)
- `config.cluster_telemetry_server_name` → `JUNGLE_TELEMETRY_SERVER_NAME`
  (if absent, use same value as `JUNGLE_TELEMETRY_ENDPOINT`)

---

## Confirm

Show the user:

```
✓ Control Plane created
  ID:               <JUNGLE_CP_ID>
  Cluster endpoint: <JUNGLE_CLUSTER_ENDPOINT>
```

Then tell them what comes next:

> The CP is live in Konnect, but it won't do anything until a data plane connects to
> it. That's Stage 2 — we'll spin up a Kong Gateway in Docker and point it here.
