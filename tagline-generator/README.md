# Tagline Generator

A starter workspace that calls the Prisme.ai LLM Gateway through the **LLM App**. Pass a one-line product description, get three punchy taglines back. One automation, one config field, no UI.

## Setup

### Authentication — do you need an API key?

The LLM App authenticates each call to `llm-gateway` with whatever identity the runtime can resolve:

- **User-triggered runs** (Builder Play button, an authenticated page, `execute_automation` with a session). The runtime auto-injects the calling user's JWT on outgoing Prisme.ai fetches, so `llm-gateway` resolves *their* permissions. **If the user already has `llm-gateway:models:chat-completions` via their role, no API key is needed** — leave `imports/LLM.yml` `apiKey` empty.
- **Programmatic / external runs** (webhook calls from outside, cron jobs, agent-to-agent, anonymous public pages). There's no user context, so the LLM App needs its own credentials. Provision an `iak_…` org API key (steps below) and paste it into `imports/LLM.yml`.

### Issuing an API key (when needed)

Issue an organization API key from **Governance**:

1. Open **Governance → API Keys**, click **Create API Key**.
2. Name it (e.g. `tagline-generator`).
3. In the permission tree, expand **LLM Gateway** and check:
   - `llm-gateway:models:chat-completions` (required — lets the key call chat completions)
   - `llm-gateway:models:read` (optional — useful if you want to introspect models)
4. In **Scopes**, either tick **No restriction** (`*`) or restrict to a specific model with `llm-gateway:models:<modelId>` (e.g. `llm-gateway:models:gpt-4o-mini`). Wildcards like `llm-gateway:models:*` are supported.
5. Set an expiration if desired, then **Create**. Copy the `iak_…` value — it's only shown once.
6. In the imported workspace, open **Imports → LLM → Configure** and paste the key into `apiKey`.

### Trying it

- **From Builder:** open **Automations → `generateTaglines`**, hit *Play*, and pass `{ "description": "..." }`.
- **From an HTTP client (requires an `iak_…` key on the LLM App):**
  ```bash
  curl -X POST 'https://api.<instance>.prisme.ai/v2/workspaces/<workspaceId>/webhooks/generateTaglines' \
    -H 'x-prismeai-api-key: iak_<orgSlug>_<...>' \
    -H 'Content-Type: application/json' \
    -d '{"description": "A pocket-sized espresso maker for hikers."}'
  ```

The response is the automation's `output` block:

```json
{ "taglines": "1. ...\n2. ...\n3. ..." }
```

## How it works

| File | Role |
|---|---|
| `index.yml` | Workspace metadata and config schema |
| `imports/LLM.yml` | LLM App instance (apiKey blank — fill it in only for programmatic use) |
| `automations/generateTaglines.yml` | Callable endpoint. Takes a `description`, calls `LLM.chatCompletion`, returns `{ taglines }` |
| `security.yml` | Default RBAC |

The pattern is intentionally simple so you can copy it into any workspace that needs a single, stateless LLM call. To wire it into a page later, add an `onSubmit` event listener that calls `generateTaglines` and renders the returned `taglines`.
