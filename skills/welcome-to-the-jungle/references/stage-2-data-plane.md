# Stage 2: Data Plane (Docker)

## Concept

Explain briefly:

> A **Data Plane (DP)** is where traffic actually flows. It runs Kong Gateway, receives
> config from the control plane, and enforces your rules — rate limits, auth, routing —
> on every request. We'll run one locally in Docker so you can test the full pipeline
> on your machine.

Check Docker is available:

```bash
docker --version
```

If Docker isn't installed, direct them to **https://docs.docker.com/get-docker/** and
wait for them to install it before continuing.

---

## Step 2a: Generate a DP Certificate

The DP authenticates to the CP using mutual TLS. We'll generate a certificate pair
and pin it to the control plane.

Check the schema if needed:
```
mcp__Konnect__get_schema  →  operation_id: "create_dataplane_certificate"
```

Call with `control_plane_id: JUNGLE_CP_ID`. The response will contain:
- `cert` — PEM certificate (multi-line string)
- `key` — PEM private key (multi-line string)
- `id` — certificate ID (save this as `JUNGLE_CERT_ID` for reference)

---

## Step 2b: Write Cert Files

**Use your Write tool** (not a Bash heredoc) to write the cert and key to disk.
This works on all operating systems regardless of the user's shell.

Write the cert PEM content to:  `./certs/jungle-cluster.crt`
Write the key PEM content to:   `./certs/jungle-cluster.key`

Use the actual values from the MCP response — not placeholders.

Then get the absolute path of the current directory so you can use it in the
Docker volume mount:

```bash
pwd
```

Save the result as `JUNGLE_CERTS_DIR` (e.g. `/home/user/project/certs` or
`C:\Users\user\project\certs`). You'll substitute this into the `docker run`
command below.

---

## Step 2c: Build and Show the Docker Command

Construct the complete `docker run` command using live values from stages 1 and 2.
Replace every placeholder with the real value — the user must be able to copy-paste
this directly.

Use the absolute path from `JUNGLE_CERTS_DIR` in the volume mounts:

```bash
docker run -d \
  --name kong-jungle-dp \
  --restart unless-stopped \
  -e "KONG_ROLE=data_plane" \
  -e "KONG_DATABASE=off" \
  -e "KONG_CLUSTER_MTLS=pki" \
  -e "KONG_CLUSTER_CONTROL_PLANE=<JUNGLE_CLUSTER_ENDPOINT>:443" \
  -e "KONG_CLUSTER_SERVER_NAME=<JUNGLE_CLUSTER_SERVER_NAME>" \
  -e "KONG_CLUSTER_TELEMETRY_ENDPOINT=<JUNGLE_TELEMETRY_ENDPOINT>:443" \
  -e "KONG_CLUSTER_TELEMETRY_SERVER_NAME=<JUNGLE_TELEMETRY_SERVER_NAME>" \
  -e "KONG_CLUSTER_CERT=/certs/cluster.crt" \
  -e "KONG_CLUSTER_CERT_KEY=/certs/cluster.key" \
  -e "KONG_LUA_SSL_TRUSTED_CERTIFICATE=system" \
  -e "KONG_KONNECT_MODE=on" \
  -e "KONG_VITALS=off" \
  -v <JUNGLE_CERTS_DIR>/jungle-cluster.crt:/certs/cluster.crt:ro \
  -v <JUNGLE_CERTS_DIR>/jungle-cluster.key:/certs/cluster.key:ro \
  -p 8000:8000 \
  -p 8001:8001 \
  kong/kong-gateway:latest
```

**Windows note:** Docker Desktop on Windows accepts both forward-slash and
backslash paths in volume mounts. If the user is on Windows without WSL2,
the cert files will be at a path like `C:\Users\...\certs\jungle-cluster.crt`
and Docker Desktop will handle mounting it correctly.

Show this command to the user with all real values filled in. Tell them:

> Copy and run this command in your terminal. The DP will start and connect to your
> control plane within a few seconds.

**Wait for the user to confirm they've run it.**

---

## Step 2d: Verify DP Connected

Poll the Konnect API until the node appears (check up to 5 times, 5 seconds apart):

```
mcp__Konnect__execute  →  list_dataplane_nodes  →  control_plane_id: JUNGLE_CP_ID
```

When a node appears in the response, show:

```
✓ Data Plane connected
  Node ID:  <node_id>
  Version:  <version>
  Status:   connected
```

If after 5 attempts no node appears, suggest the user check `docker logs kong-jungle-dp`
for error messages, and confirm the cert files exist in `./certs/`.

---

## Bridge to Stage 3

Once the DP is connected:

> Perfect — your gateway is running and talking to Konnect. Right now it has no
> configuration, so all requests will get a 404. Let's fix that by adding a Service
> and Route.
