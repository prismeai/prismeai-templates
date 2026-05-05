# Tagline Generator

A starter workspace that calls the Prisme.ai LLM Gateway through the **LLM App**. Type a one-line product description in the page form, get three punchy taglines back. One automation, one page, one config field.

## Setup

1. **Get an LLM Gateway API key.** Issue one against the `llm-gateway` workspace:
   ```bash
   curl -X POST 'https://api.<instance>.prisme.ai/v2/workspaces/<llmGatewayWorkspaceId>/security/apikeys' \
     -H "Authorization: Bearer $TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"name":"tagline-generator","rules":[{"action":"execute","subject":"automations"}]}'
   ```
   Copy the returned `apiKey`.
2. **Configure the LLM App.** In the imported workspace, open **Imports → LLM → Configure** and paste the key into `apiKey`.
3. **Try it.** Open the **Tagline Generator** page, type a product description, hit *Generate*.

## How it works

| File | Role |
|---|---|
| `index.yml` | Workspace metadata and config schema |
| `imports/LLM.yml` | LLM App instance (apiKey blank — fill it in after import) |
| `automations/generateTaglines.yml` | Listens to the `generate` event, calls `LLM.chatCompletion`, stores the result in `session.taglines`, emits `taglinesReady` |
| `automations/loadPage.yml` | Page init — exposes `session.taglines` to the page |
| `pages/index.yml` | Intro + form + result panel; `updateOn: taglinesReady` re-runs `loadPage` |
| `security.yml` | Default RBAC |

The pattern is intentionally simple so you can copy it into any workspace that needs a single, stateless LLM call.
