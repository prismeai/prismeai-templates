# Prisme.ai workspace templates

Two minimal, working starter workspaces that show how to call Prisme.ai's
client apps from your own workspace. Import either one, paste an API key,
and you have a usable page in under five minutes.

| Template | Client app used | What it shows |
|---|---|---|
| [`tagline-generator/`](./tagline-generator) | **LLM App** (wraps `llm-gateway`) | Stateless `chatCompletion` call from a form |
| [`travel-buddy/`](./travel-buddy) | **Agent Factory App** (wraps `agent-factory`) | Creating an agent and sending messages to it |

## When to pick which

- **LLM App** — one-shot text in, text out. Summaries, classifications, taglines, transformations. No memory, no tools, no canvas.
- **Agent Factory App** — stateful agents with instructions, tools, and conversation history. Anything where you want a *named persona* doing recurring work.

If you're unsure, start with the LLM App. If you find yourself rebuilding memory or tool-calling on top of `chatCompletion`, switch to the Agent Factory App.

## How to import a template

1. Open https://studio.prisme.ai (or your own instance).
2. **Workspaces → Import** → pick the template's folder zipped, or drop it in via the CLI / API.
3. Open the freshly imported workspace in Builder.
4. Follow the per-template setup note below.

## Per-template setup

### Tagline Generator

1. Get an LLM Gateway API key. On the LLM Gateway workspace (`llm-gateway`): `POST /v2/workspaces/{llmGatewayWorkspaceId}/security/apikeys` with `{"name":"...","rules":[{"action":"execute","subject":"automations"}]}`. Copy the returned `apiKey`.
2. In your imported workspace: **Imports → LLM → Configure**, paste the key into `apiKey`.
3. Open the **Tagline Generator** page. Type a one-line product description, hit Generate.

### Travel Buddy

1. Get an Agent Factory API key from any agent in **Agent Creator**: open the agent → **Settings → API Keys → Create**.
2. In your imported workspace: **Imports → Agents → Configure**, paste the key into `apiKey`.
3. Run the **`setupAgent`** automation once (Builder → Automations → setupAgent → Play). Copy the returned `agentId`.
4. **Settings → Workspace config**, paste the `agentId` into the `agentId` field.
5. Open the **Travel Buddy** page and ask it something.

## Anatomy of each template

Both templates follow the same shape so you can copy patterns between them:

```
<template>/
├── index.yml                    # workspace metadata + config schema
├── security.yml                 # default RBAC (editor + viewer + workspace API key)
├── imports/
│   └── <App>.yml                # the client app instance (apiKey left empty)
├── automations/
│   ├── loadPage.yml             # page init — exposes session state
│   ├── <action>.yml             # event listener that calls the client app
│   └── setupAgent.yml           # (Travel Buddy only) one-time agent provisioning
└── pages/
    └── index.yml                # the page itself: intro + form + result panel
```

The page → automation contract is the same in both:

- The form fires a custom event (`generate` / `ask`) on submit.
- An event-listener automation calls the client app, writes the result to `session.<key>`, and emits a "ready" event (`taglinesReady` / `replyReady`).
- The page's `updateOn:` re-runs `loadPage`, which re-reads `session.<key>` and pushes it back into the result block.

## Official documentation

- Builder & DSUL — https://docs.prisme.ai/products/ai-builder/overview
- Agent Factory — https://docs.prisme.ai/products/agent-factory/overview
- Webhooks & API keys — https://docs.prisme.ai/api-reference/overview
- Tutorials — https://docs.prisme.ai/resources/tutorials/no-code-rag-agent

## License

Released under the MIT License. See [LICENSE](./LICENSE) for the full text.
