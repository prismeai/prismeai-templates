# Prisme.ai workspace templates

Two minimal, working starter workspaces that show how to call Prisme.ai's
client apps from your own workspace. Import either one, paste an API key,
and you have a usable automation in under five minutes.

| Template | Client app used | What it shows |
|---|---|---|
| [`tagline-generator/`](./tagline-generator) | **LLM App** (wraps `llm-gateway`) | Stateless `chatCompletion` call from a callable automation |
| [`travel-buddy/`](./travel-buddy) | **Agent Factory App** (wraps `agent-factory`) | Creating an agent and sending messages to it |

Both templates are **automations-only** — no pages, no UI blocks. You drive them from the Builder Play button, an HTTP webhook, or any other workspace via `execute_automation`. Add a page later if you need a UI; the contract is just "call this automation with these arguments."

## When to pick which

- **LLM App** — one-shot text in, text out. Summaries, classifications, taglines, transformations. No memory, no tools, no canvas.
- **Agent Factory App** — stateful agents with instructions, tools, and conversation history. Anything where you want a *named persona* doing recurring work.

If you're unsure, start with the LLM App. If you find yourself rebuilding memory or tool-calling on top of `chatCompletion`, switch to the Agent Factory App.

## How to import a template

1. **Zip the template folder.** Pick exactly one template directory (e.g. `tagline-generator/`) and create a `.zip` of its contents. Don't zip the whole repo, and don't include any parent folder — the archive should contain `index.yml`, `automations/`, `imports/`, etc. at its root.
2. **Import the zip.** You have two options:
   - **From the UI:** open https://studio.prisme.ai (or your own instance), go to **Workspaces → Import**, and drop the zip in.
   - **From the API:** `POST` the zip to `/v2/workspaces/import` (see the [Workspaces API reference](https://docs.prisme.ai/api-reference/overview)).
3. Open the freshly imported workspace in Builder.
4. Follow the setup steps in that template's own README ([`tagline-generator/README.md`](./tagline-generator/README.md), [`travel-buddy/README.md`](./travel-buddy/README.md)). Each one walks through getting the right API key, configuring the client app, and any per-template provisioning.

## Anatomy of each template

Both templates follow the same shape so you can copy patterns between them:

```
<template>/
├── index.yml                    # workspace metadata + config schema
├── security.yml                 # default RBAC (editor + viewer + workspace API key)
├── imports/
│   └── <App>.yml                # the client app instance (apiKey left empty)
└── automations/
    ├── <action>.yml             # the callable automation (endpoint: true)
    └── setupAgent.yml           # (Travel Buddy only) one-time agent provisioning
```

Each automation is callable as an endpoint. From the Builder, hit **Play** and pass arguments inline. From outside the workspace, call:

```bash
curl -X POST 'https://api.<instance>.prisme.ai/v2/workspaces/<workspaceId>/webhooks/<automationSlug>' \
  -H 'x-prismeai-api-key: iak_<orgSlug>_<...>' \
  -H 'Content-Type: application/json' \
  -d '{"description": "..."}'   # or {"question": "..."} for travel-buddy
```

The automation returns its `output` block as JSON.

> **About the `iak_…` key:** the runtime auto-injects the calling user's identity on outgoing Prisme.ai fetches, so if you're running an automation from an authenticated session (Builder Play button, an internal page, `execute_automation`) and your user already has the right org-level permissions, **you can leave the imported app's `apiKey` blank**. An explicit `iak_…` key is only needed for programmatic / external runs where there is no user context (webhook callers, cron jobs, agent-to-agent calls, anonymous pages). When you do need one, issue it from **Governance → API Keys**; each template's README lists the exact permissions and scopes (e.g. `llm-gateway:models:chat-completions` for tagline-generator, `agent-factory:agents:read/write` for travel-buddy — travel-buddy also supports a per-agent flow under Agent → Settings → API Keys). See the [Identity & Access](https://docs.prisme.ai/products/ai-governance/identity-access#api-keys) docs for the full permission model.

## Official documentation

- Builder & DSUL — https://docs.prisme.ai/products/ai-builder/overview
- Agent Factory — https://docs.prisme.ai/products/agent-factory/overview
- Webhooks & API keys — https://docs.prisme.ai/api-reference/overview
- Tutorials — https://docs.prisme.ai/resources/tutorials/no-code-rag-agent

## License

Released under the MIT License. See [LICENSE](./LICENSE) for the full text.
