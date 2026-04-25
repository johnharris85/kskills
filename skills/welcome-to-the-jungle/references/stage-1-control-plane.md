# Stage 1: Control Plane

## Concept

Explain this briefly before creating anything:

> A **Control Plane (CP)** is the brain of your Kong setup. It stores all your gateway
> configuration — services, routes, plugins, consumers — and distributes it to data
> plane nodes. No traffic flows through the CP itself; it's purely configuration.
> We're creating a **hybrid** CP, which means data planes connect out to Konnect
> rather than requiring inbound network access.

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
