# Travel Buddy

A starter workspace that drives an agent through the **Agent Factory App**. The included `setupAgent` automation provisions a friendly travel-companion agent with a ready-made persona; the chat page sends questions to it and renders the reply.

## Setup

1. **Get an Agent Factory API key.** In **Agent Creator**, open any existing agent → **Settings → API Keys → Create**. Copy the value.
2. **Configure the Agent Factory App.** In the imported workspace, open **Imports → Agents → Configure** and paste the key into `apiKey`.
3. **Provision the agent.** Open **Builder → Automations → `setupAgent`** and click *Play*. The output contains an `agentId` (something like `agent_8cda…`). Copy it.
4. **Wire the agent into the workspace.** Open **Settings → Workspace config** and paste the `agentId` into the `agentId` field.
5. **Try it.** Open the **Travel Buddy** page and ask it something — a destination, a vibe, a vague dream.

## How it works

| File | Role |
|---|---|
| `index.yml` | Workspace metadata; exposes a single config field `agentId` |
| `imports/Agents.yml` | Agent Factory App instance (apiKey blank — fill it in after import) |
| `automations/setupAgent.yml` | One-shot endpoint that calls `Agents.createAgent` with the Travel Buddy persona; returns the new `agentId` |
| `automations/chat.yml` | Listens to the `ask` event, calls `Agents.sendMessage` with `config.agentId`, stores the reply in `session.reply`, emits `replyReady` |
| `automations/loadPage.yml` | Page init — exposes `session.reply` to the page |
| `pages/index.yml` | Intro + form + reply panel; `updateOn: replyReady` re-runs `loadPage` |
| `security.yml` | Default RBAC |

The agent persona lives in `setupAgent.yml` — edit the `instructions` block before running it (or update the agent later in Agent Creator) to repurpose this template into any other single-agent assistant.
