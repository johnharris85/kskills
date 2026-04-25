# Stage 0: Account Setup

Welcome the user to the tutorial with a short punchy intro. Something like:

> **Welcome to the Jungle.** We're going to build a real Kong Konnect setup together —
> from a blank account to a secured, rate-limited API proxy. By the end, you'll have
> a working gateway running on your machine and understand the core pieces of Kong's
> architecture. Let's go.

Then explain what Konnect is in 3-4 sentences:

> **Kong Konnect** is a managed control plane for Kong Gateway. You configure everything
> (services, routes, plugins) through Konnect's API or UI, and it pushes that config to
> one or more **data plane** nodes — the Kong Gateway processes that actually handle
> traffic. This split means you get a single control point for your whole fleet of
> gateways.

---

## Step 0a: Create a Konnect Account

If the user doesn't have one yet, direct them to sign up for free:
**https://konghq.com/products/kong-konnect/register**

They'll choose a **region** (US, EU, AU, etc.) during sign-up. Tell them any region works
for this tutorial — pick the one closest to them.

---

## Step 0b: Generate a Personal Access Token (PAT)

A PAT is how the Konnect MCP authenticates with the API on the user's behalf.

Instructions:
1. Log in to **https://cloud.konghq.com**
2. Click their avatar/initials in the top-right → **Personal Access Tokens**
3. Click **Generate Token**
4. Give it a name like `jungle-tutorial`
5. Set expiration to 30 days (or whatever works for them)
6. **Copy the token now** — it won't be shown again

---

## Step 0c: Configure the Konnect MCP

The Konnect MCP server needs the PAT to call the API. Show them the configuration:

```json
{
  "mcpServers": {
    "Konnect": {
      "env": {
        "KONNECT_TOKEN": "kpat_YOUR_TOKEN_HERE",
        "KONNECT_REGION": "us"
      }
    }
  }
}
```

Tell them to replace `kpat_YOUR_TOKEN_HERE` with their actual token and set
`KONNECT_REGION` to match the region they chose (e.g., `us`, `eu`, `au`).

This config goes in their Claude Code settings — either `~/.claude/settings.json`
(global) or `.claude/settings.json` in their project.

---

## Step 0d: Verify MCP Connection

Once the user has configured the MCP, ask them to restart Claude Code and confirm
the Konnect MCP appears as connected (they'll see it in their MCP server list).

**Wait for the user to confirm the MCP is connected before continuing.**

Ask: *"Let me know when the Konnect MCP is showing as connected and I'll kick off Stage 1."*

When they confirm, advance to the next step.

---

## Step 0e: Install and Configure decK (decK mode only)

**Skip this step entirely if `JUNGLE_MODE` is `api`.**

### What is decK?

> **decK** is Kong's declarative configuration tool. Instead of making API calls one by
> one, you describe your entire gateway config in a YAML file and decK reconciles it with
> Konnect. This mirrors a real GitOps workflow — your gateway config lives in a file you
> can commit, review, and roll back. `deck gateway diff` shows what will change;
> `deck gateway sync` applies it.

### Install decK

**macOS (Homebrew):**
```bash
brew install kong/deck/deck
```

**Linux:**
```bash
curl -sL https://github.com/kong/deck/releases/latest/download/deck_linux_amd64.tar.gz \
  | tar -xz -C /tmp && sudo mv /tmp/deck /usr/local/bin/
```

**Windows:** Download the latest release from
https://github.com/kong/deck/releases and add it to PATH.

Verify:
```bash
deck version
```

### Configure decK for Konnect

Create `~/.deck.yaml` so every `deck` command knows where to connect without
repeating flags. Substitute real values for the placeholders:

```yaml
konnect-token: kpat_YOUR_TOKEN_HERE
konnect-addr: https://REGION.api.konghq.com
konnect-control-plane-name: jungle-cp
```

**File location by OS:**
- macOS / Linux: `~/.deck.yaml`
- Windows (PowerShell): `$HOME\.deck.yaml` (or `%USERPROFILE%\.deck.yaml` in cmd.exe)
- Windows (WSL2): `~/.deck.yaml` (same as Linux)

Replace:
- `kpat_YOUR_TOKEN_HERE` → the PAT from Step 0b
- `REGION` → the region chosen at sign-up (e.g. `us`, `eu`, `au`)

The `konnect-addr` for common regions:
| Region | Address |
|--------|---------|
| US | `https://us.api.konghq.com` |
| EU | `https://eu.api.konghq.com` |
| AU | `https://au.api.konghq.com` |

Save `JUNGLE_KONNECT_REGION` to remember the region for later.

### Verify decK can reach Konnect

After Stage 1 creates the control plane, you'll be able to run:

```bash
deck gateway ping
```

For now, tell the user: "We'll verify the decK connection right after the control
plane exists in Stage 1."

---

Advance to Stage 1.
