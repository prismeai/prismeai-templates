# Travel Buddy

A starter workspace that drives an agent through the **Agent Factory App**. The included `setupAgent` automation provisions a friendly travel-companion agent with a ready-made persona; the `chat` automation sends questions to it and returns the reply. Two automations, one config field, no UI.

## Setup

### Authentication — do you need an API key?

The Agent Factory App authenticates each call to `agent-factory` with whatever identity the runtime can resolve:

- **User-triggered runs** (Builder Play button, an authenticated page, `execute_automation` with a session). The runtime auto-injects the calling user's JWT on outgoing Prisme.ai fetches, so `agent-factory` resolves *their* permissions. **If the user already has `agent-factory:agents:read`/`write` via their role, no API key is needed** — leave `imports/Agents.yml` `apiKey` empty.
- **Programmatic / external runs** (webhook calls from outside, cron jobs, agent-to-agent, anonymous public pages). There's no user context, so the Agent Factory App needs its own credentials. Provision an `iak_…` key (Flow A or Flow B below) and paste it into `imports/Agents.yml`.

### Issuing an API key (when needed)

Two valid flows. Both produce an `iak_…` value to paste into `imports/Agents.yml`.

**Flow A — Governance → API Keys (org-level, required for the very first `setupAgent` call)**

You need this flow at least once, because `setupAgent` calls `agent-factory:agents:write` to create an agent that doesn't exist yet (so you can't scope to it from Flow B).

1. Open **Governance → API Keys**, click **Create API Key**.
2. Name it (e.g. `travel-buddy-bootstrap`).
3. In the permission tree, expand **Agent Factory** and check:
   - `agent-factory:agents:read` (talk to agents)
   - `agent-factory:agents:write` (create / edit agents — required to run `setupAgent`)
4. **Scopes** — tick **No restriction** (`*`) for the initial run; you don't have an `agentId` yet to scope to.
5. Set an expiration, **Create**, copy the `iak_…` value.

**Flow B — Agent → Settings → API Keys (per-agent, lower privilege, available after `setupAgent`)**

Once Travel Buddy exists, you can issue a key that's auto-scoped to that single agent — useful for production where you don't want a wildcard `iak_…` floating around.

1. Open **Agent Creator → Travel Buddy → Settings → API Keys**.
2. Click **Generate API Key**.
3. Pick permissions:
   - **Read** — talk with the agent (enough for the `chat` automation)
   - **Write** — modify the agent (only if you want to update it programmatically)
4. Pick an expiration, generate, copy the `iak_…` value. The runtime auto-applies the scope `agent-factory:agents:<thisAgentId>`.

> **Recommended path for production:** use Flow A to bootstrap, run `setupAgent`, then switch to a Flow B read-only key for `chat`. Revoke the bootstrap key when done.
>
> **For interactive use only:** skip the API key entirely. If the user running the page/automation has `agent-factory:agents:read` (and `write` to provision), the runtime will use their identity automatically.

### Provision and call the agent

1. **Configure the Agent Factory App** (only if you went the API key route). In the imported workspace, open **Imports → Agents → Configure** and paste the `iak_…` key into `apiKey`.
2. **Provision the agent.** Open **Builder → Automations → `setupAgent`** and click *Play*. The output contains an `agentId` (something like `agent_8cda…`). Copy it.
3. **Wire the agent into the workspace.** Open **Settings → Workspace config** and paste the `agentId` into the `agentId` field.
4. **(Optional)** Generate a Flow B key from the new agent's Settings and swap it into `imports/Agents.yml`.
5. **Try it.**
   - **From Builder:** open **Automations → `chat`**, hit *Play*, and pass `{ "question": "..." }`.
   - **From an HTTP client (requires an `iak_…` key on the Agent Factory App):**
     ```bash
     curl -X POST 'https://api.<instance>.prisme.ai/v2/workspaces/<workspaceId>/webhooks/chat' \
       -H 'x-prismeai-api-key: iak_<orgSlug>_<...>' \
       -H 'Content-Type: application/json' \
       -d '{"question": "Three days in Lisbon, what do I skip?"}'
     ```

The response is the automation's `output` block:

```json
{ "reply": "..." }
```

If `agentId` is not yet set in workspace config, the response is `{ "reply": "", "error": "Set the agent ID in workspace config first. Run setupAgent if needed." }`.

## How it works

| File | Role |
|---|---|
| `index.yml` | Workspace metadata; exposes a single config field `agentId` |
| `imports/Agents.yml` | Agent Factory App instance (apiKey blank — fill it in only for programmatic use) |
| `automations/setupAgent.yml` | One-shot endpoint that calls `Agents.createAgent` with the Travel Buddy persona; returns the new `agentId` |
| `automations/chat.yml` | Callable endpoint. Takes a `question`, calls `Agents.sendMessage` with `config.agentId`, returns `{ reply }` |
| `security.yml` | Default RBAC |

The agent persona lives in `setupAgent.yml` — edit the `instructions` block before running it (or update the agent later in Agent Creator) to repurpose this template into any other single-agent assistant.
